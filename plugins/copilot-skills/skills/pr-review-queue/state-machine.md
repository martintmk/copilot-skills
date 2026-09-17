# Durable review queue

This file owns local cache identity, persistence, recovery and acknowledgment.
Use [preflight](provider-preflight.md) for provider facts and the
[runner](review-runner.md) for review/delivery contracts.

## One folder per PR

Store state outside the checkout at
`%USERPROFILE%\.copilot\pr-review-queue\` (equivalent home path on other hosts):

```text
state.json                         version 2: configuration and schedule only
monitor.lock                       owner/session/worker liveness
prs\<prKey-hash>\state.json         this PR's cache and active operation
prs\<prKey-hash>\operations\<id>\    review, delivery, receipt and evidence files
```

`prKey` combines provider, host, organization when applicable, immutable
repository ID and stable PR ID. Normalize provider ID representations before
keying; URLs, names and branches are not identities. Folder names are lowercase
SHA-256 hex of UTF-8 `prKey`; verify the stored key matches before reuse.
Keep owned artifact paths inside the state directory and never store credentials.

| Record | Retained data |
| --- | --- |
| Monitor | Confirmed active/inactive repositories, scoped target IDs, cadence, immutable configuration history, schedule key/ID/status and first-scan policy. |
| PR cache | `version: 2`, `prKey`, URL, scoped actors, lifecycle, eligibility/request provenance, target/head/iteration observations, `observedRevision`, `pendingWork`, `watch`, `lastVerifiedOperationId`, `completed`, `incompleteCoverage`, `unfinishedOperation`, `quarantinedWork`, `activeOperation`. |
| Active operation | ID, complete work key, trigger/cycle/time/evidence, exact target repository/ref/base SHA, head/ADO iteration, posting identity, phase/`resumePhase`, `reviewComplete`, `reviewPublishable`, artifact/journal paths, receipt and acknowledgment status/attempt/evidence. |
| Operation folder | Pinned metadata/diff/CI/rules, matching factual artifacts, versioned review/coverage result, exact payloads/hashes, delivery journal, whole receipt, request/acknowledgment proofs and actual outcome. Write artifacts before referencing them. |

Create/update the candidate's folder before comparing it with completed or
pending work. There is no separate durable candidate queue or monitor-wide
`tracked` map. Scan per-PR active records before collection; at most one may be
in flight. Multiple/unknown active operations block rather than choosing one.
Parse the cache locally; give selection only current observations, pending work,
quarantine and completion summaries. Load historical artifacts only for the
selected review or recovery, not into every tick's conversation.

## Lock and persistence

All state/configuration/scheduler changes hold one monitor lock through workers
and final persistence. Busy ticks exit without writes. A short-lived PowerShell
process cannot hold a lock for later calls: retain a live exclusive handle or
create-new lock file with PID/start time, session and worker ownership. Prove
the old owner and workers ended before recovery; never steal an uncertain lock.
Confirm acquisition and release; do not abandon a detached lock-holder.

Write complete JSON to a sibling staging file, flush, then atomically replace.
On Windows use `FileStream.Flush(true)` and `File.Replace` with a concrete
sibling backup path, not `$null`. First creation moves the flushed file without
overwrite. Never truncate/delete the destination first. Failed persistence
stops effects; preserve uncertain staging/backup files for reconciliation.
Atomic local replacement does not make provider writes atomic.

Missing state is fresh setup only if no history/journals exist. Malformed or
unsupported records block, never reset. Retain history across lifecycle/scope
changes. Do not replace identities or deactivate a repository with unresolved
work. Explicit scope narrowing preserves inactive history and confirmed cadence;
inactive repositories are not polled or treated as active preflight blockers.
An empty active set disables monitoring.

For scheduling, save a stable key and `registering` before creation. The fixed
tick prompt names that key/state path and reads saved active scope only.
Reconcile uncertain registration by exact key before retrying; ambiguous matches
block. Read back the single schedule/cadence and save its ID before claiming
it active. One-shot batches leave schedules untouched; null cadence is valid.

### Existing version-1 state

Migrate under the same lock, with all old workers stopped, before new work:

1. Validate and preserve an immutable copy of the old monitor, root
   `in-flight.json` and referenced artifacts; write a migration manifest with
   source hashes and old-to-new path mappings.
2. Copy each `tracked[prKey]` into its PR folder, including all fields, completed
   receipts/cycles, observed/pending/unfinished work, quarantines and identities.
   Attach the sole old in-flight operation to its PR. Copy referenced artifacts
   into that folder, remapping local paths without changing payloads/evidence.
3. Verify all records, hashes and references, then atomically commit the version-2
   monitor **last**, preserving configuration and the existing schedule identity.
   Only after that commit are the old root journal/map retired.

After interruption, resume from the manifest and verify existing destinations
before reuse. Never overwrite a conflicting record, replay provider writes,
register a new schedule or discard the legacy copy to complete migration.
Editing/installing this skill alone performs no migration or monitoring.

## Compare fetched facts with cache

Apply [eligibility and ordering](SKILL.md#eligibility) to a complete finite scan.
Unknown request presence/generation/time that could affect priority blocks the
whole selection. Unknown creation time blocks every non-target-authored reason.
Unknown review history blocks fallback only, not an otherwise established
in-window request, target-authored PR or in-window watched change.

- Derive request cycles/times from authoritative events with source IDs, not
  membership, votes, polling or PR creation. ADO evidence must distinguish
  assignment, re-request, reset and iteration changes.
- Request work keys use `(prKey, targetId, cycle, targetRepoId, targetRef, head)`,
  retaining ADO iteration evidence. A new same-head cycle is new work;
  completed cycles suppress unchanged assignments, not later requests.
- Other work uses `initial` or durable `observedRevision` plus the scoped
  snapshot. Advance that counter only on observed head/review-target changes.
  Compare the latest compatible verified baseline, not any historical matching
  SHA or the counter alone. Pin the actual base SHA separately; base-branch
  advancement alone is not a trigger, and retargeting requires a fresh baseline.
- Before sorting, save `pendingWork.key` and first-eligible UTC time: `scanAt`
  for collection, actual observation time for later changes. Preserve it
  through restarts/pauses; a changed work key gets a new time. It never replaces
  authoritative request time or derives from discussion activity.
- Exclude only exact active quarantine-key matches. A changed head, target,
  observed revision, applicable iteration or request cycle is not suppressed.
  Completion clears only work covered by its verified snapshot, never newer debt.
- Suppress duplicate delivery for an exact `incompleteCoverage` work key. Make it
  due again only when its head/request cycle changes, normal preflight observes
  a changed blocker fingerprint, or an explicit retry clears that record. This
  record is delivered coverage debt, not completion or a verified baseline.

Consider each PR once per tick. Revalidate age at admission and before every
non-target-authored publication; revalidate fallback history as well, including
later zero-effect retries. If eligibility lapses, retire without posting unless
the target-authored reason applies. Attempted or unknown provider effects still
require recovery, but elapsed age prevents any new write or retry after their
actual outcome is settled.

## Active operation

The queue atomically saves the active operation in its PR cache before workers
start and exclusively owns phase transitions. The delivery worker alone writes
its journal while active; the queue waits rather than editing that ledger.
Match operation, snapshot, actor and completion/publication status across
artifacts; mismatch blocks.
Before each mutation retain lifecycle/head/request provenance, including proven
absence and history anchors, and refresh those facts with validated bindings.

| Phase | Required durable gate |
| --- | --- |
| `reviewing` | Save the full report-only result and coordinator-confirmed `reviewComplete` or `reviewPublishable` before `delivering`; revalidate snapshot currency separately. |
| `delivering` | Journal every planned/attempted write; verify all findings, summary and required votes before `acknowledging`. |
| `acknowledging` | Reconcile only the processed request; never repost the review. |
| `commit-ready` | Archive verified receipt/acknowledgment; merge either completion or incomplete-coverage debt by operation ID, enroll watching only for the applicable lifecycle/age window, clear active operation last. |
| `quarantined` | Apply [PR-local quarantine](#pr-local-quarantine); persist its record/evidence before clearing active state, then continue later candidates. |
| `blocked` / `paused` | Retain reason, `resumePhase` and evidence; no next PR while provider effects or persistence are unsettled. |

A posted review, verified whole delivery and acknowledged request are separate
facts. On changed inputs with proven zero effects, save newer work, archive the
canceled attempt and end this tick. After attempted effects, pause delivery and
reconcile first; never relabel the old operation ID, snapshot or request cycle.

## Recover delivery

The [runner](review-runner.md#internal-receipt) owns receipt fields and verification.
Recover by provider IDs first; a lost response may be matched only by unique,
exact actor/head-or-iteration/body/authoritative-time evidence associated with
this operation. An older same-head review, incomplete/eventually consistent
listing or multiple matches cannot prove delivery or non-delivery.

Read/reconcile ambiguous attempts, never retry them blindly. Resume partial ADO
delivery only for missing writes proven unapplied, retaining verified threads.
Do not claim exactly-once delivery. Use provider receipts; visible or hidden
correlation markers require a separately agreed delivery contract.

Save newly observed heads/targets/cycles as debt before settling the old operation;
an old receipt cannot overwrite newer observations or complete newer work.
Stronger coverage rules do not invalidate completed history or force same-head
replay. In-flight work lacking a full manifest cannot authorize new publication:
reconcile prior attempts before obtaining missing review work, preserving evidence.

## Acknowledge and commit

Only a verified whole receipt permits acknowledgment: it must carry either
coordinator-confirmed complete coverage or a coordinator-confirmed publishable
incomplete review whose COMMENT/no-vote action and blocked-area warning were
confirmed by read-back.
For non-request work, record `not-applicable`; never clear a newly arrived request.

**GitHub:** reread current individual requests and authoritative generation
history. If submission auto-cleared the processed cycle, verify that outcome.
If it remains current, journal removal before the call, remove only the target
individual, then verify that exact generation ended. Never remove other users/teams.
Read/write/read-back is not atomic; do not attempt removal without evidence able
to detect and attribute re-request races, including submission's automatic removal.
Transport errors do not authorize retries.

A newer cycle stays pending. Proven supersession can settle the old cycle only;
head-only changes remain watched debt. If this operation might have cleared a
newer cycle, or history cannot establish which disappeared, block and retain
unconsumed work. Do not silently acknowledge, restore assignments or repost.

**ADO:** preserve assignments and votes. After verified delivery, durably complete
only the exact request cycle/iteration/head locally with provenance. Delivery's
permitted verdict vote is separate; acknowledgment never deletes or resets it.
New cycles/heads stay due, and missing generation evidence blocks.

Retain payloads/journals until acknowledgment is durable, then enter `commit-ready`.
Merge idempotently by operation ID and clear active state last. Complete reviews
advance `completed` and the compatible verified baseline. Publishable incomplete
reviews instead save `incompleteCoverage` with the exact work key, snapshot,
blocked areas, blocker fingerprint and receipt; clear the processed
`pendingWork`, but do not mark it completed or advance the verified baseline.
If acknowledgment succeeded but persistence failed, finish that commit without
redelivery.

## PR-local quarantine

A review/evidence failure may release the queue only when it is confined to this
exact PR work item, its snapshot and complete deduplication key are known, every
owned worker has stopped, and the journal proves **no provider mutation was
attempted** with `remoteWritesPerformed: 0`. Unknown snapshot/key blocks quarantine.
Auth, scanning, request-history semantics, global capability and persistence
failures do not qualify.

Archive its complete work key, snapshot/cycle, reason, evidence and UTC time in
`quarantinedWork` before clearing active state. Do not mark it completed, advance
the verified watch baseline, clear pending work or acknowledge the request.
Continue later candidates and keep recurrence active. Suppress only that exact
unchanged key until fresh work arrives or the user clears the quarantine.

Diagnostic comments are disabled unless confirmed configuration enables them.
If enabled, use a separate journal and read back the comment before recording
`quarantined-with-diagnostic`. It is never review completion or acknowledgment;
an unknown comment outcome blocks and cannot qualify as zero-write quarantine.

## Lifecycle

Non-target-authored drafts stop new writes, votes and acknowledgment;
target-authored drafts remain eligible. Closed, abandoned or merged PRs stop all
new effects. Stop/wait for workers and reconcile attempted effects first. Once
effects are settled, cancel unattempted steps, archive the actual
outcome/lifecycle reason, retire watching and debt, and clear active state last.
A publishable incomplete review never updates the verified baseline. Its exact
work key remains suppressed by `incompleteCoverage` until the head/request cycle
or blocker fingerprint changes, or an explicit retry clears that debt.

Retain `unfinishedOperation` only while a paused PR remains eligible. Publication
of a non-target-authored draft resumes its in-window work; changed snapshots
need fresh review, not stale writes. Closure, abandonment and merge permanently
retire watching/debt while preserving audit history. Reopening does not
reenroll the PR or resume completed, pending or unfinished work.
