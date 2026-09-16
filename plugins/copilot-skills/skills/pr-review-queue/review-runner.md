# Queue Review Runner

Load this when a PR is selected or delivery needs recovery. It adapts existing
[Review Lens](../review-lens/SKILL.md),
[worker isolation](../review-lens/worker-isolation.md) and
[finding](../review-delivery/findings-contract.md) contracts; no callbacks or
queue-specific changes inside shared skills are needed.

## Review, then deliver

Pass each fresh stage only its needed facts: `operationId`, `prKey`, scoped
target/poster IDs, pinned target/base/head and ADO iteration, trusted rules and
execution permission, admission/debt evidence, matching artifacts, validated
`operation_bindings`, owned paths, and `direct_http: false`. Propagate applicable
requirements to child workers; use validated equivalent MCP/CLI routes without
weakening the shared rules.

1. Run a fresh **report-only** Review Lens coordinator. Every required specialist
   runs; no posting or acknowledgment occurs, including in formatting stages.
   Return the snapshot, anchored findings, coverage/verdict, full
   `coverageManifest` and coordinator-confirmed `reviewComplete`.
2. Validate each actual worker record, status, evidence and snapshot against
   Review Lens's completion gate. Save the versioned `reviewArtifact` in this
   PR's operation folder before setting `reviewComplete=true` and `delivering`.
   Partial/blocked work cannot authorize publication of a completed review.
3. Start one fresh **posting** `review-delivery` worker from that saved result,
   with the journal/receipt contract below. Wait for its verified receipt;
   only then may the queue acknowledge the request. No other PR runs between
   stages, and workers never select work or acknowledge requests themselves.

Each due head/request gets a full fresh review, not a delta skim. Reuse matching
facts, not stale conclusions or another review's reasoning. Still-applicable
unresolved findings affect the outcome even when duplicate inline posts are
omitted. Follow the shared finding format throughout artifacts and delivery.
Apply delivery's automatic approval policy to complete current-head clean or
nit-only reviews; report-only preparation does not suppress the required approval
in the authorized posting stage. Do not ask for per-PR approval confirmation.
Target/requester-owned or poster-owned PRs use GitHub `COMMENT`, no ADO vote.
No replies, thread resolution, fixes, pushes or feedback-autonomy cascades.

## Delivery journal

The queue owns cleanup; the active delivery worker exclusively writes its
per-operation journal. Before any remote write, atomically persist:

```text
version: 1
operationId, prKey, reviewedBase, reviewedHead, postingIdentity, reviewComplete
plannedWrites[]: stable key, kind, intent, anchor/iteration, exact body or
                immutable payload path, payload hash
receipts[]: attempt interval, provider ID, outcome, read-back evidence
```

`reviewComplete` comes from the saved coordinator result, never from comments.
Before first publication revalidate fallback-only age/history; retire if it
lapsed and no other eligibility applies. After known publication, retain
completion debt even if age or the operation's own review changes eligibility.

Persist `attempting` before each bound call, then its outcome/IDs before any
later write. Unknown outcomes, including server errors without non-delivery
proof, are ambiguous. Follow [recovery](state-machine.md), not blind retries
or persisted shell commands. Never store credentials or erase unsettled evidence.

GitHub requires a submitted review at the pinned head with the intended event
(including `APPROVED` for an `APPROVE` event), not a draft, issue comment or
COMMENT substituted for approval. ADO requires properly anchored iteration-bound
threads, summary and any required verdict vote. Read back every intended finding,
summary and vote. Acknowledgment remains the queue's separate responsibility.

## Internal receipt

Return this to the queue, not in the public body:

```text
status: verified | partial | ambiguous | blocked
operationId, prKey, reviewedBase, reviewedHead, postingIdentity
reviewIds[], threadIds[], reviewUrl
postedAt, verifiedAt
voteStatus: not-requested | verified | missing
```

`reviewedBase` retains the target repository ID, ref and exact base SHA;
`reviewedHead` is an exact SHA, with ADO iteration evidence alongside it.
Accept `verified` only when operation, snapshot and actor match both complete
coordinator-confirmed coverage and read-back of all required writes.
Diagnostics, drafts, local reports or prose success never qualify. Use actual
IDs/times; missing writes/coverage are `partial`, unknown outcomes `ambiguous`,
and unavailable operations `blocked`. Own-PR votes are `not-requested`.

Return the receipt unchanged and retain all artifacts until the queue durably
records acknowledgment or lifecycle retirement.
