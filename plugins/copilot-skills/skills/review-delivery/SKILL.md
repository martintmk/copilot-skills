---
name: review-delivery
description: >
  Publish completed review results to GitHub, Azure DevOps or a local report.
  Owns current inline anchors, rolling reports, votes, human-review labels,
  owned non-actionable thread closure and partial-write recovery.
  Use for full or focused reviews, eligible mechanical approvals and benchmark
  follow-ups. Not for generating findings, scheduling reviews, code changes
  or formatting intermediate area output.
---

# Review Delivery

Publish supported findings once at their code locations.
The caller owns analysis and scheduling. This skill owns publication and its receipt.
Reuse existing threads and the authorized report location without expanding write permissions.

## Inputs and preparation

Require the mode, assessed inputs, current revalidated inputs and explicit post/report-only scope.
For a PR, include provider identity, target ref and merge base.
For local inputs, include the caller's input digest.
Accept saved report/review/thread IDs, pending operations, report location, marker and submission cooldown.

| Mode | Required results |
| --- | --- |
| Full or focused review | Merged findings, coverage, outcome and full-review revision separate from later revalidated revisions |
| Full Review Lens review | Also applicable public-contract, generated-surface, integration-test or benchmark reports and policy decisions |
| Mechanical approval | The complete [lightweight evidence package](../review-lens/mechanical-approval.md#5-deliver-and-retain-evidence), not full-review coverage |
| Benchmark follow-up | Pinned benchmark report, `review-perf` interpretation and prior delivery IDs |

Resolve missing inputs with the caller without importing all artifacts or repeating analysis.
A pending benchmark block does not delay supported review results.
Without a PR target, use report-only mode and skip provider reads.
Report-only mode never authorizes provider writes.
Before drafting, make sure that supplied label decisions follow the [current-evidence rules](#human-review-label-and-completion).

1. Finish the saved draft from compact results under the [findings and summary contract](findings-contract.md).
   Preserve supported text and required blocks. Keep logs and internal narratives in artifacts.
2. Invoke `simple-english` for prose without changing exact attribution, headings, code or facts.
   Make sure that serialized bodies still match the intended text.
3. Apply the mode-specific rules below, reconcile existing output and choose the permitted action.
4. Publish through the provider procedure, then return the [confirmed receipt](#human-review-label-and-completion).

Before each provider write, including retries and thread-status changes:

- Read live lifecycle and make sure that the head, target ref and merge base still match the revalidated inputs.
- Stop writes for a closed or merged PR.
- If scope changed, return pending work to the caller's [head-movement procedure](../review-lens/coordinator-reference.md#when-the-head-moves).
- Require permission for that operation and honor its caller-defined cooldown.
- Reconcile uncertain prior writes under [ownership and recovery](#ownership-and-recovery).

Moving anchors alone does not revalidate stale findings.
For local reports, make sure that the input digest still matches before returning current results.

## Lightweight mechanical approval

Accept only complete proof from the [mechanical procedure](../review-lens/mechanical-approval.md), not an incomplete full review renamed as mechanical.
Require current, complete public-contract, generated-surface and test evidence
with combined `not required`.
Use the [lightweight summary](findings-contract.md#lightweight-mechanical-summary), keeping the reports as supporting evidence.

Before submission, refresh every mechanical prerequisite, including CI, labels, discussion and active review holds.
An unresolved substantive finding or changed eligibility returns this mode to the caller without voting.
Apply the action table and common write gates.
Do not start a monitor, dismiss reviews or add a human-review label through this mode.

## Benchmark follow-up

This follow-up does not require the other area reports or another full review.
Apply the common write gates and these additional limits:

1. If assessed inputs changed, retain measurements locally as historical.
   Historical publication needs explicit permission and cannot present results as current.
   A target-tip-only change follows the caller's head-movement rules.
2. Deliver the complete compact report under the [benchmark contract](../metabench-compare/run-and-report.md#side-by-side-results).
   Use one owned benchmark comment or an authorized report location.
   Summary-edit permission alone does not authorize another comment.
   If no location is permitted, retain the report and state the delivery limit.
3. Preserve the defect verdict and contract/test policy decisions.
   Measurements alone never change a verdict, vote or label.
   Route new supported `review-perf` findings through normal evidence, anchor and permission gates.
   Reconsider a verdict only from current findings and sufficient coverage.
4. After confirmed delivery, update the matching pending notice only when authorized.
   Link the report and assessed revisions without changing unrelated content or a newer summary.

## Reconcile inline findings

Use the full current PR diff, not only the delta since the previous review.
The commit delta revalidates findings. The cumulative diff supplies anchors.

1. Read existing threads, replies, status and locations.
   Match root cause and symbol, including findings in top-level reports.
   Return disputed, deferred or uncertain applicability to the caller instead of inventing a new finding or repeating analysis.
2. Reuse an unresolved thread that represents a revalidated finding on current code.
   Provider line tracking alone does not prove applicability.
3. Anchor unrepresented findings to the smallest relevant block in the cumulative diff, accounting for renames and moved lines.
   If an outdated thread no longer locates the block, link it from one current replacement.
   Do not replace threads the provider still maps correctly.
4. Omit findings the caller proved fixed. Keep uncertain claims only as coverage limits.
   Neither outcome authorizes editing or resolving another person's thread.
5. Put full findings in the summary only without a valid anchor or when inline delivery is blocked.
   State the exact limit and pending work. New commits and incomplete coverage are not anchor failures.

Preserve evidence and corrections when locations change. Never anchor to unrelated code.
Map generated-surface symbols to source anchors without changing their evidence.

## Rolling reports

Use the caller's report location and marker under [ownership and recovery](#ownership-and-recovery).
Recover an existing report ID before creating another.
Rebuild from current results under the summary contract, not from preparation logs.
Include the caller's required revision and coverage fields, then its marker.

Count only current applicable findings in unresolved inline threads.
Link those threads instead of repeating their bodies.
Include unanchored or pending findings separately with their delivery limits.
Keep fixed findings out of the count even when their threads remain open.
Skip identical bodies and unchanged provider actions.

If a separate rolling report is authorized, link it from the submitted review body instead of duplicating it.
The report does not replace a required provider vote or prove inline delivery.

## Close owned non-actionable threads

A non-actionable comment requests no change, answer or decision.
Keep summaries and design notes visible while closing eligible owned threads.
Non-blocking defects and nits remain actionable.

1. Require explicit thread-resolution scope and proof of root-comment ownership.
   Comment permission and AI attribution are insufficient.
2. Read the complete current thread before each status write.
   Close only an owned summary or design note without unresolved findings, pending finding delivery or unanswered actionable replies.
   A summary containing only links to separate finding threads can close. Leave those threads open.
3. After confirmed publication, use supported provider operations:
   Resolve GitHub review threads. Top-level comments and review bodies have no resolved state.
   Mark eligible ADO threads `Closed`, not `Fixed`.
   Never delete comments or change votes to close threads.
4. Apply the same rule to owned prior output and skip already resolved threads.
   Read back status before recording completion. Report unavailable permission, ownership or capability as a delivery limit.

Use the common recovery procedure for uncertain status writes.
Actions on received feedback belong to [Feedback Autonomy](../feedback-autonomy/SKILL.md), not this informational cleanup.

## Choose the action

Use the [shared severity policy](findings-contract.md#evidence-and-severity).
Decide from all current findings, including existing unresolved threads.
Resolve PR-author, requester and posting-account identities before voting.
If these identities are uncertain, withhold the vote and report the limit.
Read effective approvals by distinct provider reviewer IDs.
Exclude pending/dismissed GitHub reviews and superseded votes.
For ADO, count individual positive reviewer votes, not assumed group membership.

| Situation | GitHub event | ADO vote |
| --- | --- | --- |
| Proven eligible mechanical path | `APPROVE` | Approved |
| Human review policy is `required` | `COMMENT` | No vote |
| Complete, clean or only non-blocking findings | `APPROVE` | Approved, or Approved with suggestions |
| Complete, serious findings, fewer than two distinct approvals | `REQUEST_CHANGES` | Waiting for author |
| Serious findings with two or more distinct approvals | `COMMENT` | No vote |
| Approval history unavailable when a blocking action is considered | `COMMENT`, disclose the limit | No vote |
| Incomplete or findings-only refresh | `COMMENT` | No vote |
| Requester's or posting account's own PR | `COMMENT`, no `Verdict:` | No vote |
| Report only | No provider writes | No provider writes |

The mechanical row does not claim complete full-review coverage.
Its required evidence must be complete. Missing eligibility proof means no lightweight approval.
The most restrictive row wins. Preserve serious findings when voting restrictions
prevent a blocking action. Do not call the review clean or discard those findings.
An approval sentence does not replace a confirmed provider vote.
Follow a caller's narrower write scope or submission cooldown.
No vote means no vote write, not a `NoVote` action that clears an existing vote.

## Provider publication

Discover the available provider operations and schemas at runtime.
Use the provider's current cumulative diff and exact revalidated revision.
Prefer native review, comment, vote, label and thread-status operations over hand-built HTTP requests.
If only a lower-level API is available, follow its discovered schema.

For inline findings:

- Anchor to the smallest relevant current diff range.
- Distinguish added, changed and removed code with the provider's current side and iteration model.
- Map renamed or moved code from current source.
- Make a suggestion replace exactly its selected range.
- If the provider cannot represent a valid anchor, retain that finding in the summary with the limitation.

For each submitted review, comment, vote, label or status change, read back:

- Body and provider author.
- Revision or iteration association.
- File and range for inline findings.
- Effective event, vote, label or thread status.

An unsubmitted draft is not delivery.
Treat permission, rate-limit and schema failures as their actual limits, not as anchor failures.
Do not add reviewers or change unrelated provider state to obtain voting access.
When provider writes are not atomic, save each confirmed ID before the next write.

## Ownership and recovery

Before editing owned output, require its saved ID, PR scope, expected author and marker where the output uses one.
Prefer stable actor IDs. A provider login or `uniqueName` must map unambiguously to the authenticated account when IDs are absent.
Display names, attribution and a marker alone do not prove ownership.
Honor stricter caller identity rules. Missing proof blocks that edit, not separately authorized work.

Record intended operations before writing and save returned IDs.
After a timeout, interruption or failed read-back, inspect provider output before retrying.
Recover only confirmed missing or rejected writes, never an uncertain creation.
Read back each effect before marking it confirmed.
If inputs move, keep confirmed IDs and return pending operations for revalidation.

Keep analysis completion separate from delivery completion.
Unsupported votes, labels or anchors are explicit delivery limits, not lost analysis.
Never rerun review areas, repost a whole batch or replace live output with a stale draft to recover publication.

## Human-review label and completion

Combine current applicable public-contract, generated-surface and
integration-test policy decisions:

| Evidence | Decision |
| --- | --- |
| Proved public contract or generated-surface addition/break, or integration-test behavioral break | `required`, even if other evidence is incomplete |
| Every applicable report is complete without those changes | `not required` |
| Missing decisive evidence without a proven qualifying change | `undetermined` |

If none of these contracts is affected, complete scope evidence can establish
`not required` without empty reports.
Benchmarks do not decide this signal. Historical reports cannot authorize a current label write.
When required and authorized, add `human-review-required` without changing other labels.
On GitHub, create a missing repository label with description `Requires human review` only when permitted.
On ADO, use a discovered PR-label operation, never a work-item tag.
Never remove this label. For `undetermined`, make no change.
If the required write is unavailable or prohibited, report the label as unapplied.

After publication, apply the [owned-thread procedure](#close-owned-non-actionable-threads) and return one receipt:

- Report/review URL and confirmed provider IDs.
- Assessed and revalidated revisions, defect verdict and policy status.
- Compact coverage receipt and finding-to-thread map.
- Confirmed vote/event, label and resolution states.
- Pending operations, benchmark owner/state and remaining delivery limits.

Report-only returns the complete public report. After posting, return the receipt without echoing the full report.
Preserve recovery evidence and remove owned payload scripts.
Return ready results without waiting for temporary-build cleanup, but keep cleanup assigned and report failures separately.
