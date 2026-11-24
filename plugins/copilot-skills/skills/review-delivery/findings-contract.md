# Shared Findings Contract

This file owns finding shape, severity and public summaries for full and focused
reviews. Area skills add evidence rules, not another template.
`review-public-docs` returns data. `metabench-compare` returns measurements.
Neither produces findings.

## Attribution and finding content

Every separately posted summary or finding starts with one exact attribution
on its own first line, followed by a blank line:

- `**Posted by an AI agent**`: summaries and serious findings.
- `**Posted by an AI agent · Non-blocking**`: issues that do not justify blocking.
- `**Posted by an AI agent · Nit**`: cosmetic issues.
- `**Posted by an AI agent · Design note, no change requested**`: observations.

Do not prepend a publication note. Embedded report blocks inherit the summary's
attribution. Remove their repeated attribution, internal status and handoff fields.

This shape is a useful default:

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

Each actionable finding must make the problem, impact and proposed correction clear.
The model can use the default headings or a more compact structure that preserves those facts.
The title names the defect, not only the topic or proposed fix.
A Design note names the observation and requests no change.
Clean summaries need no empty finding sections.
Non-blocking issues and nits remain actionable.
[Review Delivery](SKILL.md#close-owned-non-actionable-threads) owns their thread status.

One root cause per finding. Name the symbol and keep `path:line` in location
metadata. Use body line references only for another location.
Generated-surface API findings use public paths instead of source lines.
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
Intentional public contract or test changes can require human review without being defects.

## Verdict and policy

Keep two independent results:

- The **verdict** comes from findings and coverage.
- The **policy status** records human-review requirements, mechanical
  eligibility and vote restrictions.

Policy can limit the provider action without changing finding severity or the
defect verdict. A clean review can still require human review for an intentional
public contract change. State both results when they differ.

Choose `approve`, `approve with non-blocking comments` or `changes requested`
from all applicable findings, including existing unresolved threads.
Clean reviews and Design notes permit approval. All non-blocking issues, not
only nits, permit approval with comments.
Incomplete full or focused reviews have no overall verdict.
[Review Delivery](SKILL.md#choose-the-action) owns ownership, refresh, prior-approval
and report-only restrictions on the actual provider event.
Never lower a finding's severity or omit it merely to obtain approval.

## Area result and final summary

Return findings in impact order with reviewed scope and material limits.
Use clear status language such as `done`, `not applicable` or `could not review`
when another skill or caller needs a structured status.
Area agents do not choose the combined verdict or post their own results.
Route focused reviews through Review Delivery too.

For a bundled review, return each finding once.
Add compact area statuses only for delegated, blocked or materially limited scopes.
Keep inventories and trace details in artifacts.

Applicable public-contract, generated-surface and integration-test
classifications return their report, coverage and policy decision:
`required`, `not required` or `undetermined`.
Keep commands, file inventories, full logs and setup diagnostics in artifacts.
The public report includes only the evidence needed to understand its classification.

Full Review Lens reviews use the requirements below.
The [lightweight mechanical summary](#lightweight-mechanical-summary) is a separate mode, not a claim of full coverage.
The public summary is a current report, not a concatenation of area outputs or a fixed template.
Keep its length proportional to findings and changed contracts, not the number of reviewed paths.
Do not remove findings, lower severity or hide material limits to meet a length target.

1. Include one attribution, outcome, reviewed scope and material limits.
   For incomplete coverage, add **Warning: Incomplete review**, name missing
   topics with one-clause reasons, and state that no overall verdict is given.
2. Include a compact coverage receipt from the coordinator's existing record.
   Name areas executed, ruled not relevant and blocked, with short reasons.
   Do not include worker IDs, per-file inventories or a second internal ledger.
3. Record full-review and revalidated revisions separately after a findings-only
   refresh. Say that later commits received no new-finding review.
4. Include source-level public-contract, generated Rust surface or
   integration-test reports when those contracts were affected.
   Keep classifications, concise signatures and meaningful behavioral deltas.
   Use the [compact API report](../review-public-api-changes/SKILL.md#report), not its full capture inventory.
5. Include benchmark results when requested, measured or material to the review.
   If assigned measurements are pending, state their pinned inputs and status.
6. Put findings without confirmed inline delivery in the summary, with the
   anchor or delivery limitation. For confirmed inline findings, give a count
   and thread links, not duplicate bodies.

For unavailable contract evidence, write **Could not assess** and name the reason.
A [best-effort review](../review-lens/coordinator-reference.md#best-effort-completion)
keeps the incomplete warning, supported findings and no-verdict statement.
Label source-based API observations separately from the generated-surface classification.
They do not complete that classification or determine its human-review signal.

Describe applicable benchmarks as pending, complete, partial, blocked or not applicable.
Do not make measured claims without measurements.
Use the [benchmark report format](../metabench-compare/run-and-report.md#side-by-side-results)
for completed results and deliver later results through [benchmark follow-up](SKILL.md#benchmark-follow-up).

Keep public output current and concise.
Do not publish stale reports as current, inaccessible local links, internal IDs,
permission history, full inventories, logs or setup diagnostics.
Do not publish an area ledger unless the user asks for it.

One valid incomplete summary is:

```markdown
**Posted by an AI agent**

**Warning: Incomplete review**

I reviewed the source-level contract, correctness, tests and documentation.
Generated Rust surface output could not be assessed because the required capture failed.
No overall verdict is given. Tests and benchmarks were not run.

### Generated Rust surface

**Could not assess**

Matching base/head API output is unavailable.
```

Standalone reviews outside Review Lens include only information relevant to their scope.
Requester-owned or posting-account-owned PRs omit public `Verdict:` framing.
Missing measurements limit performance claims, not finished source-review coverage.

## Lightweight mechanical summary

Use this content only after the [mechanical eligibility procedure](../review-lens/mechanical-approval.md) succeeds.
Preserve the exact AI attribution. Name the category, decisive evidence and assessed head.
Use accessible evidence links, not local artifact paths.
State that this is not a full review and that benchmarks were not run.
Keep full contract and test evidence in artifacts.

The following is an example:

```markdown
**Posted by an AI agent**

Lightweight mechanical approval at <head>: <category and concise evidence>.
No public contract or generated surface changes. Existing test expectations and coverage are preserved.
All current checks passed. This is not a full risk-based review. Benchmarks were not run.
```

In report-only mode, use "Eligible for lightweight mechanical approval. No vote submitted." instead of claiming approval.
If delivery is blocked, state eligibility and the delivery limit without claiming an approval.
If eligibility is blocked, return the exact unmet gate locally instead of using the approval template.
Reuse an authorized rolling report when the caller provides one. Do not create a second summary.
Review Delivery owns the actual provider event and confirmation.
