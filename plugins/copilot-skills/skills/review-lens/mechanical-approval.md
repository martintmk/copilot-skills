# Mechanical PR approval

Use this path for bounded, proven changes that need no full Review Lens review.
Here, a mechanical PR preserves contracts or restores established behavior.
No public API change is allowed, including compatible additions.
An unchanged API alone does not make a PR mechanical.

This procedure owns lightweight classification, not monitoring or provider writes.
Read the linked rules without invoking the monitor skill.
Review Delivery owns votes, attribution, permissions and write recovery.

## 1. Establish the scope

1. Pin the PR identity, head, target ref, merge base and target tip.
2. Read trusted repository rules and structural lifecycle, label and CI facts.
   Defer PR bodies, review bodies and discussion until the relevant test baselines are recorded.
3. Inventory all changed paths, including renamed, generated and binary inputs.
   If existing evidence already rules out mechanical scope, continue the full review without completing a redundant lightweight pass.
   Read baseline tests and changed expectations using the [test-first order](coordinator-reference.md#combine-test-review).
   Then read the complete merge-base-to-head diff, connected source and relevant discussion.
   Recover truncated content from the exact revisions. Unread changes block lightweight approval.
4. Account for every changed path. Small size, filenames, labels and author claims do not prove eligibility.

Keep the caller's execution limits. Reuse matching evidence before running focused commands.
Do not install tools, execute PR scripts with credentials or expand permissions to obtain approval.
Use direct inspection and only the isolated classification context required below, not an eleven-area reviewer fleet.

## 2. Prove a permitted change

Apply the [three bounded-impact and coverage invariants](../pr-auto-approve/SKILL.md#three-invariants).
This path is stricter: it permits no public API additions.
Every part of the diff must fit one or more categories below.
Mixed changes qualify only when every part qualifies.
One disqualifying change is enough to route to full review. Approval still requires proof for the complete diff.

| Category | Required evidence |
| --- | --- |
| Docs, comments, formatting or private mechanical edits | Trace the affected paths. Establish unchanged product behavior and documented contracts. Make sure that executable examples match existing behavior. |
| Added tests | Identify each new scenario and meaningful assertion. Show execution at the current revision without replacing or weakening existing coverage. |
| Regression repair | Identify the earlier PR and introducing commit. Show the prior intended behavior, the introduced failure and the narrow restoration. Require a regression test or faithful reproduction that fails on the regressed revision and passes at the current head. |
| Build or pipeline repair | Link the observed failure to the changed configuration or code. Require the repaired job or a faithful focused reproduction to pass at the current revision. Preserve supported behavior, execution coverage and trust permissions. |

A regression claim needs repository history and test or contract evidence, not the PR description alone.
If the earlier PR or failure cannot be established, use the full review path.
Behavior changes beyond a proven regression restoration do not qualify.
Redesign and unproven dependency or generated-output changes do not qualify.
A small diff that disables a failing test or bypasses a pipeline gate is not a repair.

## 3. Prove unchanged contracts

1. Prepare the [package inventory and API evidence](coordinator-reference.md#public-api-evidence).
   Run [Public API Changes](../review-public-api-changes/SKILL.md) with matching base/head evidence.
   Preserve its output-only boundary in a [source-free context](coordinator-reference.md#protect-api-contexts).
   Run classification only, not the separate API design audit or its doc phase.
   Require `No API Changes` for every affected library and relevant configuration.
   Use the inventory-based no-library case only when its proof is complete.
2. Trace contracts that API captures cannot prove, including defaults, errors, wire formats, CLI behavior and feature selection.
   Only a proven regression restoration from [the permitted categories](#2-prove-a-permitted-change) can change product behavior.
3. Prepare the [integration-test inventory](coordinator-reference.md#integration-test-evidence).
   Run [Integration Tests](../review-integration-tests/SKILL.md).
   Require a complete report with `human-review-required: not required`.
4. Apply the shared coverage invariant to unit tests, doctests and other suites too.
   Require the combined API/test human-review decision to be `not required`.

Do not run the remaining specialist areas or benchmarks for this path.
Do not mark them `done` or `not applicable`. They were not performed.
An explicit benchmark or specialist request retains its requested scope instead of taking this shortcut.

## 4. Decide without starting a monitor

Use only the opening approval prerequisites in [Decide](../pr-auto-approve/SKILL.md#decide).
Do not apply its monitor actions, escalation, approval dismissal or stop handling.
Use all current GitHub check runs and commit statuses, or the corresponding ADO build, status and policy results.
Require at least one reported check and successful completion of every reported check.
Bind check evidence to the current head or a verified merge revision.
Paginate complete results. Map provider states from the discovered schema.
Treat unavailable or unmappable results as unknown, not success.
Review Delivery also applies its own-PR, report-only, write-scope and cooldown restrictions.

| Result | Next action |
| --- | --- |
| Eligible | Deliver one lightweight approval through Review Delivery. Do not start the full review. |
| Not mechanical, changed contract, existing substantive finding or human-review label | Continue the full review with the collected evidence. Do not approve through this path. |
| Incomplete change, API, test or regression proof | Continue the full review and retain the evidence gap. Never turn missing proof into lightweight eligibility. |
| Otherwise eligible, but CI is pending, absent, failed or unknown, or another approval prerequisite is unmet | Withhold approval and return the exact blocker. Do not fall through to a full-review approval to bypass this gate. |

For report-only mode, return the eligibility result without provider writes.
A failed lightweight classification is not itself a defect or a reason to request changes.
Do not add a human-review label merely because CI is pending or evidence is unavailable.
When continuing a full review, reuse valid evidence and completed classification rather than repeating them.
Its existing report, coverage and label rules apply.
Honor an explicit full-review request even when this path cannot approve.

## 5. Deliver and retain evidence

Pass `review mode: mechanical`, the category and its decisive evidence to Review Delivery.
Include the exact revisions, both classification reports, combined label decision, current prerequisite results and authorized writes.
Include prior report/review/thread IDs for recovery.
Do not claim full-review coverage or supply invented benchmark results.

Immediately before submission, refresh revisions, lifecycle, labels, discussion and every approval prerequisite.
If any prerequisite changes, stop the pending vote and reassess the complete current PR.
Do not reuse a lightweight approval as a full-review baseline for findings-only refresh.
Read back the provider vote before claiming approval.

Retain the review mode, classification evidence, revisions and confirmed write IDs in session storage.
Recover uncertain writes before retrying. Do not duplicate a confirmed approval for the same eligible revision.
On a later requested run, reassess changed commits or targets from the complete diff.
This path neither schedules checks nor dismisses prior approvals.
