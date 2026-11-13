# Shared Findings Contract

This file owns finding shape, severity and public summaries for full and focused
reviews. Area skills add evidence rules, not another template.
`review-public-docs` returns data. `metabench-compare` returns measurements.
Neither produces findings.

## Attribution and comment shape

Every separately posted summary or finding starts with one exact attribution
on its own first line, followed by a blank line:

- `**Posted by an AI agent**`: summaries and serious findings.
- `**Posted by an AI agent · Non-blocking**`: issues that do not justify blocking.
- `**Posted by an AI agent · Nit**`: cosmetic issues.
- `**Posted by an AI agent · Design note, no change requested**`: observations.

Do not prepend a publication note. Embedded report blocks inherit the summary's
attribution. Remove their repeated attribution, internal status and handoff fields.

```markdown
**Posted by an AI agent · Non-blocking**

**<Concrete defect affecting a named surface>**

**Problem**
<Current defect and decisive evidence.>

**Why this matters**
<Consumer or runtime impact.>

**Suggested fix**
<Specific correction.>
```

Use the exact headings in that order, normally one or two sentences each.
The title names the defect, not just the topic or proposed fix.
A Design note uses an observation title and omits only **Suggested fix**.
Its **Problem** explains the evidenced tradeoff without asserting a defect.
Clean summaries need no empty finding sections.
Design notes request no change. Non-blocking issues and nits remain actionable.
[Review Delivery](SKILL.md#close-owned-non-actionable-threads) owns their thread status.

One root cause per finding. Name the symbol and keep `path:line` in location
metadata. Use body line references only for another location.
Output-only API findings use public paths instead of source lines.
Use exact-range `suggestion` fences only for a complete replacement of the anchor.
Include a focused regression test only when useful as permanent coverage.
Markdown prose and fences start at column zero.

## Evidence and severity

Keep the decisive evidence, impact and correction. Remove investigation
narration, repeated titles, raw logs, speculative evolution claims and generic advice.
Separate proven defects from conditional questions.
Never describe a source trace as an executed test.
Area-specific proof requirements still apply.

Only serious correctness, security, data-loss, reliability or materially broken
user-facing behavior can justify a blocking finding.
Ordinary bugs, test gaps, design/doc drift and maintainability concerns are
non-blocking unless their concrete impact meets that bar.
Intentional API or test changes can require a human-review label without being defects.

Choose `approve`, `approve with non-blocking comments` or `changes requested`
from all applicable findings, including existing unresolved threads.
Clean reviews and Design notes permit approval. All non-blocking issues, not
only nits, permit approval with comments.
Incomplete full or focused reviews have no overall verdict.
[Review Delivery](SKILL.md#choose-the-action) owns ownership, refresh, prior-approval
and report-only restrictions on the actual provider event.
Never lower a finding's severity or omit it merely to obtain approval.

## Area result and final summary

Return findings in impact order, followed by one coverage/status line.
Use `done`, `not applicable` or `could not review`, with a reason for the last two.
Include reviewed scope and material evidence limits.
No findings requires only attribution and that line.
Area agents do not choose the combined verdict or post their own results.
Route focused reviews through Review Delivery too.

For a bundled review, return each finding once and one compact scope/status entry per assigned area.
Reuse evidence references across entries. Keep the complete path inventory and trace details in artifacts.
Do not add clean-area essays, execution diaries or another copy of the same findings.

Report areas return their required block, coverage/status and tri-state label
decision: `required`, `not required` or `undetermined`.
Keep commands, file inventories, full logs and setup diagnostics in artifacts.
The public report includes only the evidence needed to understand its classification.

Full Review Lens reviews use the summary below.
The [lightweight mechanical summary](#lightweight-mechanical-summary) is a separate mode, not a claim of full coverage.
The public summary is a current report, not a concatenation of area outputs.
Keep its length proportional to findings and changed contracts, not the number of reviewed paths.
Do not remove findings, lower severity or hide material limits to meet a length target.

1. Start with one attribution, outcome, reviewed topics and material limits.
   For incomplete coverage, add **Warning: Incomplete review**, name missing
   topics with one-clause reasons, and state that no overall verdict is given.
2. Record full-review and revalidated revisions separately after a findings-only
   refresh. Say that later commits received no new-finding review.
3. Include exactly one `### Public API changes`, `### Integration tests` and
   `### Benchmarks` block, in that order. Keep required classifications, concise
   Rust signatures and meaningful behavioral deltas.
   If benchmarks are ready, include compact results or link the separately delivered report with its revisions and limits.
   While an assigned comparison is queued or running, state `Pending`, that state and the pinned revisions.
   Use the [compact API report](../review-public-api-changes/SKILL.md#report), not the full capture inventory.
4. Put findings without confirmed inline delivery after those blocks, with the
   anchor or delivery limitation. For confirmed inline findings, give a count
   and thread links, not duplicate bodies.

For unavailable API/test evidence, use **Could not assess** and the reason.
For benchmarks, retain `partial`, `blocked` or `not applicable` with the reason.
`Pending` is a delivery state, not a completed benchmark result.
It does not make finished source performance coverage incomplete or prevent an otherwise supported verdict.
Do not claim measured performance while it is pending.
Follow the [benchmark report format](../metabench-compare/run-and-report.md#side-by-side-results).
The later [benchmark follow-up](SKILL.md#benchmark-follow-up) contains the complete compact results without repeating the full review.
Keep full metrics and run details in artifacts, not the public summary.
Unless requested, keep the eleven-area ledger internal.
Public coverage names the reviewed scope and every material gap without listing every clean area.
Do not expand the compact report, drop benchmark cases or link inaccessible local paths.
Old report blocks and measurements remain pinned to their assessed revisions
until their required evidence is refreshed. Never present them as current.

Remove obsolete preparation claims such as "pending/unposted", "report only" or
"no provider writes" when publishing. Keep a current pending-benchmark notice and actual delivery limits.
Do not repair contradictory prose by prepending another disclaimer.
Internal hashes, agent IDs, permission histories and exhaustive inventories stay
out of the public summary.

```markdown
**Posted by an AI agent**

**Warning: Incomplete review**

I reviewed the source-level contract, correctness, tests and documentation.
Public API output could not be assessed because the required capture failed.
No overall verdict is given. Tests and benchmarks were not run.

### Public API changes

**Could not assess**

Matching base/head API output is unavailable.

### Integration tests

**No Integration Test Changes**

### Benchmarks

Blocked: Gungraun requires a Linux host.
```

Standalone reviews outside Review Lens do not require the three report blocks.
Requester-owned or posting-account-owned PRs omit public `Verdict:` framing.
Missing measurements limit performance claims, not finished source-review coverage.

## Lightweight mechanical summary

Use this shape only after the [mechanical eligibility procedure](../review-lens/mechanical-approval.md) succeeds.
Preserve the exact AI attribution. Name the category, decisive evidence and assessed head.
Use accessible evidence links, not local artifact paths.
State that this is not a full review and that benchmarks were not run.
Keep full API/test evidence in artifacts rather than adding the three full-review blocks.

```markdown
**Posted by an AI agent**

Lightweight mechanical approval at <head>: <category and concise evidence>.
No public API changes. Existing test expectations and coverage are preserved.
All current checks passed. This is not a full eleven-area review. Benchmarks were not run.
```

In report-only mode, use "Eligible for lightweight mechanical approval. No vote submitted." instead of claiming approval.
If delivery is blocked, state eligibility and the delivery limit without claiming an approval.
If eligibility is blocked, return the exact unmet gate locally instead of using the approval template.
Reuse an authorized rolling report when the caller provides one. Do not create a second summary.
Review Delivery owns the actual provider event and confirmation.
