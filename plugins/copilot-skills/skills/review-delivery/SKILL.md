---
name: review-delivery
description: >
  Deliver merged review findings to GitHub, Azure DevOps or a local report.
  Keep findings inline on current code after new commits, reconcile prior
  threads, close owned non-actionable threads, and recover partial writes without
  duplicates. Owns review events,
  AI attribution and required human-review labels. Not for generating findings
  or formatting intermediate area output.
---

# Review Delivery

Publish supported findings once at their code locations. Reuse existing threads
and any caller-owned rolling report. Delivery cannot invent evidence or expand
write permissions.

## Inputs and preparation

Require merged [findings](findings-contract.md), coverage/outcome, full-review
revision, current revalidated revision, target ref, merge base, PR identity and
post/report-only mode. Accept prior report/review/thread IDs and pending writes.
For Review Lens, also require the three report blocks and the combined
`human-review-required` decision with reasons.
Return missing evidence to the caller, not another review pass.

1. Build the public summary from the current result using the
   [summary contract](findings-contract.md#area-result-and-final-summary).
   Do not paste internal area reports, delivery logs or historical proposals.
2. Invoke `simple-english` for prose. Preserve the contract's exact attribution,
   headings, code and technical facts. Check serialized bodies after rewriting.
3. Read current lifecycle, head, target ref, merge base and submitted reviews.
   If the PR is closed or merged, stop PR writes and return its terminal state.
   If scope changed, return to the caller's
   [head-movement procedure](../review-lens/coordinator-reference.md#when-the-head-moves).
   Never fix a stale review by moving line numbers alone.
4. Make sure that the label decision agrees with the current mandatory reports.
   API additions/breaks or integration-test behavioral breaks require the label.
   Otherwise both reports must be complete for `not required`.
   Missing evidence means `undetermined` unless a proved change already requires
   human review. Historical reports cannot supply a new current-head decision.
   Benchmarks do not decide this signal.

For a findings-only refresh with historical reports, retain their prior decision
as historical and make no new label write from it.

## Reconcile inline findings

Use the full current PR diff, not only the delta since the previous review.
The commit delta revalidates findings. The cumulative diff supplies anchors.

1. Read existing threads with replies, status and current locations.
   Match root cause and symbol, including findings already in a top-level report.
   Return disputed or explicitly deferred findings to the caller with their
   disposition. Do not repost them as new issues without new evidence.
2. Reuse an unresolved inline thread that still represents the finding on the
   current code. Provider line tracking alone is not proof that the finding still applies.
3. For each new or still-applicable unrepresented finding, locate the smallest
   relevant block in the current diff. Map renames and moved lines from current source.
   A report-only finding is not an existing inline comment.
4. If an old thread is outdated and no longer locates the affected block, create
   one current inline replacement with a link to it. Do not duplicate threads
   that the provider already mapped correctly.
5. If the root cause is fixed, omit it from current findings. If it is uncertain,
   retain it only as a coverage limit, not a proven defect.
   Neither decision authorizes resolving or editing another person's thread.
6. Put full findings in the summary only when no valid anchor exists or an
   authorized inline write is blocked. State the reason and pending delivery.
   Refresh, incomplete coverage and new commits are not anchor failures.

Preserve the finding's evidence and correction when its location changes.
Do not attach it to unrelated code merely to obtain an inline comment.
Public-API reviewers supply symbols, not source anchors. Delivery can map those
symbols to the diff without feeding source back into their output-only review.

## Close owned non-actionable threads

A non-actionable comment reports information without asking for a change, answer or decision.
Keep summaries and design notes visible, but do not leave their threads open as unfinished review work.
Non-blocking defects and nits still request changes. Keep their unresolved threads open.

1. Require thread-resolution permission within the caller's write scope.
   Permission to post or edit comments alone does not grant it.
   Use the [ownership rules](#ownership-and-recovery) to prove that this delivery owns the thread's root comment.
   AI attribution alone does not prove ownership.
2. Read the whole current thread, including replies and status.
   Close an owned summary or design-note thread only when it contains no unresolved finding, question or requested decision.
   A summary that only links separate finding threads can close. Leave those finding threads open.
   Keep summaries with unanchored findings, pending finding delivery or requests for human action open.
   Do not close another person's thread or a thread with an unanswered actionable reply.
3. After confirming publication, resolve eligible GitHub review threads through a supported thread-resolution operation.
   GitHub review bodies and top-level issue comments have no resolved state. Leave them visible without a resolution write.
   For ADO, discover the supported status operation and mark eligible threads `Closed`, not `Fixed`.
   Do not delete comments or change the review vote to close a thread.
4. Apply this rule to new comments and owned prior output during reconciliation.
   Recheck lifecycle and discussion before each status write. Skip already resolved threads.
   Read back the resolved or closed state before recording completion.
   If permission, ownership or the operation is unavailable, record the specific delivery limit.
   Recover uncertain status writes through [write recovery](#ownership-and-recovery), not by reposting comments.

This procedure manages the review's own informational output.
For actions on received feedback, use [Feedback Autonomy](../feedback-autonomy/SKILL.md), including its separate resolution gate.

## Choose the action

Use the [shared severity policy](findings-contract.md#evidence-and-severity).
Decide from all current findings, including existing unresolved threads.
Read effective approvals by distinct provider reviewer IDs before voting.
Exclude pending/dismissed GitHub reviews and superseded votes.
For ADO, count individual positive reviewer votes, not assumed group membership.

| Situation | GitHub event | ADO vote |
| --- | --- | --- |
| Complete, clean or only non-blocking findings | `APPROVE` | Approved, or Approved with suggestions |
| Complete, serious findings, fewer than two distinct approvals | `REQUEST_CHANGES` | Waiting for author |
| Serious findings with two or more distinct approvals | `COMMENT` | No vote |
| Approval history unavailable when a blocking action is considered | `COMMENT`, disclose the limit | No vote |
| Incomplete or findings-only refresh | `COMMENT` | No vote |
| Requester's or posting account's own PR | `COMMENT`, no `Verdict:` | No vote |
| Report only | No provider writes | No provider writes |

The most restrictive row wins. Preserve serious findings when voting restrictions
prevent a blocking action. Do not call the review clean or discard those findings.
An approval sentence does not replace a confirmed provider vote.
Follow a caller's narrower write scope or submission cooldown.
No vote means no vote write, not a `NoVote` action that clears an existing vote.

## GitHub

Use serialized JSON in a file with `gh api --input`, not shell-quoted Markdown
or `-f body=@file`. For a new submitted review:

```text
POST repos/<owner>/<repo>/pulls/<n>/reviews
{ body, event, commit_id, comments: [{ path, line, side, body }] }
```

Pin `commit_id` to the revalidated head. `RIGHT` uses head lines, `LEFT` removed
base lines. Anchor within current diff hunks. For ranges, add `start_line` and
`start_side` with `start_line < line`. Omit them for one line.
A suggestion replaces exactly its selected range.

Recheck head and scope immediately before submission. Read back the submitted
review and comments, including event, commit, body and locations.
An `APPROVE` must read back as `APPROVED`. A pending draft is not delivery.
Fix rejected `422` payloads. Treat `403`/`429` as access/rate limits, not anchor errors.

## Azure DevOps

Discover the configured PR, changes, thread and voting schemas. Pass their actual
organization fields. Use `!<id>` or the canonical PR URL, never `#<id>`.

1. Read the current iteration's cumulative changes against the common commit.
   In tools exposing `get_changes`, use `compareTo: 0` and paginate.
2. Create one thread per finding with `filePath`, precise head-side ranges and
   that file's positive `changeTrackingId`. Supply both comparing iteration IDs
   when the schema supports them. In this configured schema, common-base comparison uses the current iteration
   for both `firstComparingIteration` and `secondComparingIteration`.
   A prior-iteration comparison instead uses its actual left iteration.
   Discover equivalent fields on other instances instead of assuming this shape.
3. Use 1-based line and character offsets. Whole-line start is `1`, end is the
   last line's character count plus one. Never use `0` or a guessed large offset.
4. Read back body, file/range and iteration association before counting a finding
   as inline. If fields are omitted, use a supported detail read.
   If association cannot be confirmed, retain delivery as unverified, not unanchored success.
5. Write one unanchored summary or update the caller's owned rolling summary.
   Apply the [non-actionable thread procedure](#close-owned-non-actionable-threads) after confirming its contents.
   Vote only after required comments are confirmed and the action table permits it.
   Do not add a reviewer to obtain voting access without separate authorization.

If the schema cannot anchor removed-only code or a specific comparison, retain
that finding in the summary with the limitation. Do not move all findings there.
Recheck source and review scope before each write. Record successful IDs because
ADO threads are not an atomic batch.

## Ownership and recovery

Before editing a rolling comment, require its saved ID, marker, PR scope and
expected provider author. Prefer stable actor IDs. When IDs are omitted, a
provider-returned login or `uniqueName` must map unambiguously to the authenticated
posting account. Display names, body attribution and a marker alone do not prove ownership.
Honor stricter caller identity requirements. Missing proof blocks that edit,
not independent analysis or other separately authorized actions.

Record each intended operation and returned ID. After a timeout, interruption or
read-back failure, inspect existing output before retrying. Recover only missing
or rejected writes. Do not rerun review areas or repost the whole batch.
If the head moves mid-batch, keep confirmed IDs and return pending items for revalidation.

Keep analysis completion separate from delivery completion. An unsupported vote,
label or anchor is a specific delivery limit, not a reason to erase finished coverage.
Never replace a live report with a stale preparation artifact.

## Human-review label and completion

When required and authorized, add the exact `human-review-required` PR label
without changing other labels. On GitHub, create a missing repository label
with description `Requires human review` only when permitted.
On ADO, use only a discovered PR-label operation, never a work-item tag.
Recheck the head before writing and read the label back.
Never remove the label. For `undetermined`, make no label change.
If writes are prohibited or unavailable, state that the required label remains unapplied.

Report-only returns the complete public report without provider writes.
After posting, apply the [non-actionable thread procedure](#close-owned-non-actionable-threads).
Return the report/review URL, finding-to-thread map, confirmed resolution states,
revision and remaining delivery limits. Do not echo the full posted report.
Remove owned payload scripts, but retain results needed for recovery.
