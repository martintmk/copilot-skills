# Durable review queue

Use [provider-preflight.md](provider-preflight.md) for authoritative facts and
validated per-operation `operation_bindings` (MCP or official provider CLI).
Raw HTTP is disabled by default; MCP+CLI coverage gaps block work, never silently enable HTTP.
Review new commits until merge. Never perform discussion-only follow-ups,
replies, fixes, or thread resolution.
Revalidate prior issues on new heads: unresolved ones still affect verdict and
summary, even when duplicate inline findings are omitted.

## Setup, storage, and ownership

- Resolve scoped GitHub `martintmk` and independent, stable ADO human identities;
  never derive ADO identity from GitHub. Missing required unattended setup blocks.
- First actual monitor setup asks/saves repositories and confirmed cadence before starting.
  Explicit one-shot batches ask only for needed repos/identities; absent/null cadence is legal.
  Preserve saved cadence; never create/change a schedule for a one-shot batch.
- Store `state.json`, `in-flight.json`, and owned delivery artifacts outside
  the checkout, under `%USERPROFILE%\.copilot\pr-review-queue\` on Windows
  (the equivalent user-home directory on other hosts). Do not commit them.
- Take an exclusive state-directory lock covering worker lifetimes. Overlapping runs
  exit busy without writes. Never steal an old lock: prove its owner and workers
  ended. Uncertain ownership/liveness blocks.
- Write complete versioned JSON through a sibling staging file, flush it, then
  atomically replace the destination. Atomic local replacement is **not** an
  atomic provider transaction. Persistence failure stops further side effects.
- Missing state is fresh setup only when no journal/history exists. Malformed
  or unsupported state/journals block; preserve them, never silently reset.
  Explicit migrations preserve identities, receipts, cycles, and pending work.
- Configuration changes also take the lock. Never replace identities or remove
  an in-flight repository mid-transaction; retain removed repositories' history.

Illustrative version-1 state; IDs/SHAs are placeholders and `15m` is not a default:

```json
{
  "version": 1, "configuration": {
    "cadence": "15m",
    "repositories": [
      {"provider": "github", "host": "github.com", "repositoryId": "R1", "repository": "example/project"},
      {"provider": "ado", "host": "dev.azure.com", "organizationId": "O1", "projectId": "J1", "repositoryId": "R2"}
    ],
    "targets": {
      "github|github.com": {"id": "U1", "login": "martintmk"},
      "ado|dev.azure.com|O1": {"id": "human-identity-id", "kind": "human"}
    }
  },
  "schedule": {"key": "monitor-1", "id": null, "status": "not-started"},
  "tracked": {
    "github|github.com|R1|P1": {
      "url": "https://github.com/example/project/pull/1",
      "lifecycle": "open", "watch": true, "observedRevision": 2, "observedHead": "head-2",
      "lastVerifiedOperationId": "op-1",
      "unfinishedOperation": null,
      "pendingWork": {"key": "work-2", "eligibleAt": "2026-09-14T08:05:00Z"},
      "completed": [{
        "operationId": "op-1", "targetId": "U1",
        "trigger": {"kind": "request", "cycle": "event-10"},
        "reviewedBase": {"repositoryId": "R1", "ref": "main", "sha": "base-1"},
        "reviewedHead": "head-1", "receiptArtifact": "operations\\op-1.json",
        "acknowledgment": {"status": "verified", "evidenceId": "event-11"}
      }]
    }
  }
}
```

For monitor registration, persist a stable `schedule.key` and `registering`
status before creation; include the key in the fixed tick prompt. Record the
returned ID/status afterward. Reconcile that exact key against the scheduler
before retrying an uncertain registration; multiple/unknown matches block.
One-shot runs leave this record and existing schedules untouched.

Archive immutable, versioned review results, whole receipts, delivery journals, and request/acknowledgment
proof before state references them. Keep history across lifecycle/configuration changes and payloads until durable acknowledgment.

## Facts, selection, and deduplication

1. Resume existing work before new collection/selection. Then capture one UTC
   `scanAt` and a finite, fully paginated list of open/published non-draft PRs
   without an age cutoff; refresh tracked lifecycles/heads only in the current
   monitored scope. Retained history is not permission to poll a removed repo.
   Bound reads/retries; exhaustion blocks.
2. Derive individual target request cycles and request times from authoritative
   provider events/history, preserving source IDs. Do not use assignment
   membership, polling time, PR creation, or a reset-to-zero vote as substitutes.
   ADO cycle evidence must distinguish reassignment, re-request, reset, and
   iteration changes; unchanged membership alone is neither due nor completed.
3. Select due PRs in exactly two groups; each PR appears only once:
   - **Current explicit requests:** oldest authoritative current-generation
     request time, then `prKey`. Prior submitted reviews never exclude them.
   - **All other eligible work combined:** target-authored PRs initially or with
     changed heads/targets, and watched head/target changes, irrespective of age
     or prior reviews; plus other PRs with `24h < scanAt - createdAt <= 7 * 24h`
     and complete authoritative history proving no submitted review by anyone.
     Published partial work retained in `unfinishedOperation` is completion
     debt, not a new age-fallback admission.
     Sort this whole group by `pendingWork.eligibleAt` oldest first, then `prKey`;
     ownership/eligibility categories have no separate priority.
   Discussion/pending GitHub reviews do not count as submitted reviews;
   dismissed/historical reviews do. Reset ADO votes never prove no prior review.
4. Unknown request presence/generation/time that could affect request priority
   blocks selection, not merely that PR. Unknown creation/history blocks
   incidental eligibility; it does not exclude an otherwise fully established
   request, own PR, or watched head change. Never report unknown as absence.
5. `prKey` includes provider, host, organization where applicable, immutable
   repository ID, and stable PR ID. Scope actors by provider/host/organization.
   URLs, display names, and branch names alone are not durable identity.
6. Request deduplication uses `(prKey, targetId, cycle, targetRepoId, targetRef, head)`;
   pin the exact reviewed base separately. **Same-head new cycles are new work.**
   Completed ADO cycles/heads suppress unchanged assignments, not later resets/iterations/heads.
7. Non-request work uses `initial` or durable `observedRevision`, advanced only
   on observed head/target changes, plus the scoped snapshot. Never use it as
   request evidence. Compare the latest compatible baseline, not all historical SHAs.
   Before sorting, persist the scoped `pendingWork.key` and first-observed
   `eligibleAt`: `scanAt` for batch eligibility, actual UTC time for later changes.
   Reuse it across ticks/restarts/pauses; a changed key gets a new observation time.
   Never derive it from discussion/updated time or substitute it for request time.
   Completion clears only pending work covered by the verified snapshot, not newer work.
   An observation counter alone does not make the latest verified compatible
   snapshot due again without a new request or unfinished work.
8. Persist exact target repository/ref/base SHA, head SHA, and ADO iteration.
   Retargeting needs a fresh baseline. Base-branch advancement alone is not a
   new-head trigger; pin the actual base for work, never relabel old evidence.
9. Consider each PR at most once per tick; do not replenish the list. New heads/
   requests wait for later ticks. Settle the current transaction first; PRs are **sequential**.

Revalidate age/history for fallback-only work before admission and first
publication, including a zero-effect retry on a later day. If another review
arrived or the age window expired and no other reason applies, retire it safely.
After known publication, its own feedback or elapsed age must not invalidate
completion recovery; preserve that debt rather than treating it as a new PR.

## One in-flight operation

Before review work, atomically create the sole `in-flight.json`:

```json
{
  "version": 1, "operationId": "op-2",
  "prKey": "github|github.com|R1|P1",
  "targetId": "U1", "postingIdentity": "posting-actor-id", "phase": "reviewing",
  "trigger": {"kind": "request", "cycle": "event-12", "requestedAt": "2026-09-14T08:00:00Z", "evidenceId": "event-12"},
  "reviewedBase": {"repositoryId": "R1", "ref": "main", "sha": "base-2"},
  "reviewedHead": "head-2", "reviewComplete": false,
  "reviewArtifact": "operations\\op-2.review.json",
  "deliveryJournal": "operations\\op-2.delivery.json", "receipt": null,
  "acknowledgment": {"status": "not-started", "attempt": null}
}
```

The queue alone owns `in-flight.json` and transitions. The fresh Review Lens
coordinator runs report-only and returns its result; the queue saves it before
starting the separate posting worker. Pass identity/snapshot, run-local
`operation_bindings` and `direct_http: false` to both stages and their workers.
One delivery worker exclusively writes the
`deliveryJournal` while active; the queue never edits or duplicates its ledger.
The [queue-owned runner](review-runner.md) owns execution/journal/receipt contracts;
these are invocation requirements, not modifications to existing review skills.
Match operation/snapshot/actor/completion across journals; mismatch blocks.
Workers never select another PR or acknowledge requests. Before mutations, save
lifecycle/head/request provenance, including proven absence and history anchors.

| Phase | Allowed transition / durable gate |
| --- | --- |
| `reviewing` | Receive complete report-only Review Lens work on the pinned snapshot; durably save `reviewArtifact` and coordinator-confirmed `reviewComplete=true` before `delivering`. |
| `delivering` | Worker persists the complete plan before any remote write and journals each attempt/outcome. Read back all required posts/vote before `acknowledging`. |
| `acknowledging` | Delivery is verified; perform only request acknowledgment/reconciliation, never repost review feedback. |
| `commit-ready` | Verified whole receipt and acknowledgment proof are durable; idempotently merge into tracked history, set the watch baseline, then remove the journal last. |
| `blocked` / `paused` | Persist `resumePhase`, the reason and all evidence. No next PR while review, delivery, or acknowledgment remains unsettled. |

A posted review, verified whole operation, and acknowledged request are separate
facts. Refresh lifecycle, target/head, and request cycle before every new write.
With proven zero effects, changed inputs invalidate artifacts: archive the
canceled attempt and end this tick. After attempted effects, pause further
delivery on changes and reconcile first; never change its ID, snapshot, or cycle.

## Delivery receipt and crash recovery

- Each `plannedWrites` entry has a stable key, kind, anchor/iteration, intent, and
  **(exact body or immutable payload artifact) plus hash**. Persist `attempting`
  before the bound MCP/CLI call; save provider IDs/outcomes/read-back in `receipts` before
  later writes. Unknown attempted outcomes are ambiguous, never presumed unsent;
  server errors alone do not prove non-delivery. Retain attempt intervals.
- Require an **internal**, never posted, receipt: `status=verified|partial|ambiguous|blocked`,
  `operationId`, `prKey`, `reviewedBase`, `reviewedHead`, `reviewIds/threadIds`,
  `reviewUrl`, `postingIdentity`, `postedAt`, `verifiedAt`, and `voteStatus`.
- Accept `verified` only for the exact operation/snapshot after full provider read-back
  of intended feedback/required votes **and** coordinator-confirmed complete selected
  Review Lens work. Never infer `reviewComplete` from posted comments or journal existence.
  It is not an extra receipt field; diagnostics with blocked/incomplete coverage and drafts never qualify.
  No ADO vote on target/poster-owned PRs: use `voteStatus=not-requested`, not cast.
- Recover by provider IDs first. After a lost response, a unique exact
  author/head-or-iteration/body/authoritative-time match may reconcile a write.
  Multiple matches, missing history, or an unprovable operation association
  block. A matching old review on the same head is not enough.
- Prefer provider receipts, not visible or supposedly hidden correlation tags.
  Any marker requires a separately agreed delivery contract.
- Partial ADO thread delivery resumes only missing writes proven unapplied,
  preserving verified threads. Ambiguous writes are **read/reconciled**, not
  retried. Missing results in an incomplete/eventually consistent listing are
  not proof of no effect. No blind whole-operation retry or exactly-once claim.
- Changed heads/targets/cycles do not invalidate historical receipts or satisfy
  new work. Persist the new observation/debt before settling the old transaction;
  never replace a newer observed head with the old receipt's head during commit.

## Acknowledgment and finalization

**GitHub:** only after verified whole delivery, reread current individual
requests and generation history. Automatic removal by review submission counts
only when evidence proves the processed generation ended.

- If that exact cycle is still current, refresh head/cycle immediately before
  removing **only the configured target individual**, then read back requests
  and authoritative generation history. Never remove teams or other users.
- Do not assume a conditional-generation delete exists. Read/write/read-back
  is not atomic: re-requests may occur between reads and either review submission
  (which may auto-clear requests) or explicit removal. Do not attempt removal
  if the available evidence cannot detect/attribute that race.
- A newer current cycle stays untouched and unconsumed. Proven supersession of
  the old cycle can settle only the old acknowledgment; retain the newer cycle
  as due. Persist head-only changes as watched debt before settling the old
  cycle; acknowledgment never marks the newer head reviewed.
- If a newer cycle may have been cleared by this operation, or history cannot
  establish which cycle disappeared, preserve the unconsumed work and **block
  for reconciliation**. Do not silently acknowledge it, auto-restore assignments,
  repost feedback, or claim the safeguards prevented the race.
- Journal removal attempts in `in-flight.json` before writing. A transport error is not
  success and not permission to repeat removal. Resolve the exact cycle's
  disappearance from authoritative evidence; otherwise retain pending
  acknowledgment. Verify the processed cycle is no longer pending before
  proceeding. With no processed request, acknowledgment is not applicable;
  never clear a newly arrived request.

**ADO:** preserve reviewer assignments and votes during acknowledgment. Never
delete/reset them to emulate GitHub removal. After verified delivery, durably
complete only the exact processed request cycle/iteration/head locally, retaining
its provenance. A required non-own review verdict is part of delivery, not an
acknowledgment cleanup. New authoritative cycles/heads remain due; missing
generation evidence blocks rather than rerunning membership or suppressing it
forever. Non-request operations have an explicit not-applicable acknowledgment.

Retain the delivery journal/payloads until acknowledgment is durable. Archive proof,
enter `commit-ready`, merge by `operationId`, then delete the in-flight journal.
If clearing succeeded but saving failed, idempotently finish the commit, never redeliver.

## Lifecycle and edge cases

Draft/closed/abandoned/merged PRs stop new writes. Wait for workers to finish and
reconcile all attempted writes; unknown outcomes still block. Once effects are
settled, cancel unattempted steps, including acknowledgment that is no longer
applicable, and archive the operation with its actual result and lifecycle
reason. Do not record a partial review as complete or update its verified
baseline. Remove the in-flight record last so another eligible PR can proceed.

Merge ends watching. Other inactive states retain history and pending work;
on reopening/publication, reconcile archived partial artifacts before starting
due work, without replaying completed snapshots or blindly creating another
draft review. Unsafe or unprovable recovery still blocks. Never post, vote or
remove reviewers after a PR has become ineligible.
Keep a tracked `unfinishedOperation` artifact reference for published partial
work. On reopening it remains completion debt even if age/history no longer
qualify for initial fallback; changed snapshots require a fresh review, not
replay of stale writes. Clear only debt covered by successful completion, or
retire it permanently on merge while retaining its audit history.

| Case | Required outcome |
| --- | --- |
| Same SHA, new explicit request | New authoritative cycle creates a fresh review; old SHA receipt cannot consume it. |
| New commits more than seven days later | Watched open PR remains due; the initial age window no longer applies. |
| Target-authored PR already reviewed | Still initially eligible; thereafter review changed heads/targets or new requests. |
| Exactly 24h / exactly 7d / over 7d | Incidental eligibility: no / yes / no, using the single UTC scan instant. |
| Partial threads or lost posting response | Reconcile exact writes; no next PR and no blind duplicate posts. |
| Crash after posting, before clearing | Verify whole receipt, then acknowledgment only; never redeliver. |
| Merge/closure before all planned steps finish | Reconcile attempted writes, retire unattempted steps without claiming completion, then release the queue. |
| New request during completion | Preserve newer cycle; if a mutation may have consumed it, block and reconcile. |
| Unknown request time or review history | No fabricated ordering/absence; block the applicable selection/work. |
| Closed/draft, then reopened | Retain history, pause writes, resume due current work; merged stays terminal. |
| Concurrent scheduler ticks | One lock owner and one in-flight transaction; other ticks exit busy. |
