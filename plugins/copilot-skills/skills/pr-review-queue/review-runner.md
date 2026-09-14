# Queue Review Runner

These are invocation-specific requirements for `pr-review-queue`, not changes
to the shared review skills. Use Review Lens's existing review, evidence,
isolation and presentation rules; keep queue bookkeeping here.

Preserve the [findings contract](../review-delivery/findings-contract.md)
through report-only output, saved artifacts and posting. Every finding,
including a design note, retains its attribution, bold title, **Problem** and
**Why this matters**. Actionable findings also require **Suggested fix**. Clean
summaries and internal receipts are not finding bodies.

## Two stages, one PR at a time

The queue owns the phase boundary; it does not need callbacks or modifications
inside shared skills. Supply each stage with the relevant subset of:

- operation ID, `prKey`, target/requester and posting identities;
- exact target/base/head or iteration, trusted rules and execution permission;
- admission reasons/evidence and any unfinished-operation references;
- matching factual artifacts, never another review's reasoning;
- the validated MCP/provider-CLI operation bindings from
  [preflight](provider-preflight.md), with direct HTTP disabled; and
- an exclusively owned delivery-journal path and the receipt contract below.

1. Run a fresh `review-lens` coordinator with **explicit report-only mode**.
   Return the pinned operation/snapshot, complete findings with anchors,
   coverage/verdict, the full `coverageManifest` required by Review Lens, and
   coordinator-confirmed `reviewComplete`. Every sub-review must run; the queue
   does not authorize a risk-selected subset. Any local formatting stage must
   remain report-only. No posting or request clearing
   occurs in this stage.
2. Validate and atomically save that result in the operation's versioned
   `reviewArtifact`. Validate every required skill's actual worker record and
   snapshot against Review Lens's completion gate. Only complete,
   snapshot-matching work allows the queue to persist `reviewComplete=true` and
   transition to `delivering`.
3. Start one fresh `review-delivery` worker with the saved result and the
   intended PR-posting mode. Supply the journal/receipt instructions below.
   The queue waits for this worker, then validates its receipt before entering
   acknowledgment. No other PR begins between these stages.

Tell the coordinator to carry these queue-specific requirements into its
workers' prompts. Use the selected equivalent operation when a shared skill's
default tool is unavailable; do not weaken its evidence, anchoring or verdict
rules. Missing MCP support is not a blocker when a validated CLI route covers
the operation. Workers cannot improvise unvalidated HTTP calls.

Perform a full review of the selected current head. Suppress duplicate inline
findings, but keep still-applicable unresolved issues in the outcome: no new
finding is not approval when an existing blocker remains. Do not reply to
discussion, resolve threads, edit code, or invoke feedback-autonomy.
On the target's or posting identity's own PR, use COMMENT/no ADO vote.

## Delivery and journaling

Pass these instructions to the fresh delivery worker alongside the normal
`review-delivery` input. The queue owns the journal and recovery payloads; the
delivery worker has exclusive write access while active, not cleanup ownership.

Before the first remote write, atomically save a version-1 JSON journal with
`version`, `operationId`, `prKey`, `reviewedBase`, `reviewedHead`,
`postingIdentity`, `reviewComplete`, `plannedWrites` and `receipts`.
`reviewComplete` comes from the coordinator, not the existence of comments.
Keep exact bodies or immutable payload artifacts plus hashes, stable write
keys, kinds, anchors and intent in `plannedWrites`; never store credentials.

Before first publication, revalidate fallback-only age/history eligibility.
If it has lapsed and no other admission reason applies, retire without posting.
Do not apply that fresh-admission test to recovery after known publication:
the operation's own feedback must not disqualify its completion.

Persist a write as `attempting` before calling its MCP or CLI operation, then
record its provider ID/outcome before another write. Unknown attempted outcomes
are ambiguous, not unsent; a server error alone is not proof of non-delivery.
Use the journal and provider read-back to recover only missing work. Revalidate
bindings on recovery instead of replaying stored shell commands.

For GitHub, require a submitted review, not a draft or issue comment. Bind the
head and verify all intended inline findings and summary. For ADO, pin threads
to the reviewed iteration using actual diff/schema fields and confirm that
association on read-back. Use the selected CLI route if it supplies a required
operation missing from MCP; a missing required vote is not successful delivery.
Request acknowledgment belongs to the queue, never to this delivery worker.

## Return an internal receipt

Return this metadata to the queue alongside the normal review result, never in
the public review body:

```text
status: verified | partial | ambiguous | blocked
operationId, prKey, reviewedBase, reviewedHead, postingIdentity
reviewIds[], threadIds[], reviewUrl
postedAt, verifiedAt
voteStatus: not-requested | verified | missing
```

`reviewedBase` retains the target repository ID, ref and exact base SHA;
`reviewedHead` is the exact head SHA. Preserve ADO iteration evidence alongside
the snapshot. Do not flatten a target change into a matching head alone.

`verified` requires the complete Review Lens coverage manifest and provider
read-back of every planned finding, summary and required vote for the exact snapshot.
Posted diagnostics, incomplete coverage, drafts and local reports do not
qualify. Return actual IDs/timestamps when known; do not invent them.
Missing writes/coverage are `partial`, unknown write outcomes `ambiguous`, and
unavailable operations `blocked`.

Return the receipt unchanged to the queue. Retain the saved review result,
delivery journals and referenced payloads until the queue records acknowledgment
or lifecycle retirement durably under the [state machine](state-machine.md).
Cleanup must not erase unresolved recovery evidence.
