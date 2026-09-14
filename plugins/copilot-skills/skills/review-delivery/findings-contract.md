# Shared Findings Contract

Read this to produce findings; the coordinator dispatches `review-delivery`
once in a fresh worker for final delivery.
Area skills add domain-specific evidence within this shape, not new templates.
`review-public-docs` is exempt: it returns a documentation bundle, not findings.

## Attribution and comment shape

Start each review summary, finding, standalone report and PR thread with the
exact bold attribution line below, followed by a blank line. Never imply the
requester wrote the review, or quote, indent or backtick the attribution.

```markdown
**Posted by an AI agent · Non-blocking**

**Why this matters**
<Concrete problem and consumer/runtime impact, with decisive evidence when useful.>

**Suggested fix**
<Specific correction and, only when useful, why it fits.>
```

The example's severity is not a default. The allowed first lines are:

- `**Posted by an AI agent**` - summaries, clean reports or blocking findings.
- `**Posted by an AI agent · Non-blocking**` - a real issue that should not gate.
- `**Posted by an AI agent · Nit**` - cosmetic.
- `**Posted by an AI agent · Design note, no change requested**` - an observation,
  with **Why this matters** only; do not invent a fix.

Use both exact section headings for actionable findings, normally one or two
sentences each. Clean summaries and coverage-only reports need attribution, not
empty finding sections. Keep questions explicitly conditional, not disguised
as defects.

## Evidence, correction and severity

One root cause per finding. Name the symbol and provide an exact `path:line`
anchor as location metadata; use body line references only for a different
location. Output-only API reports use exact public paths instead of source
anchors and must not inspect source to obtain line numbers.

Put decisive evidence under **Why this matters** and the correction under
**Suggested fix**. Preserve domain proof requirements and material uncertainty;
concision is not permission to weaken the investigation. Omit routine
`Verified:` paragraphs, investigation narration, redundant code restatements,
generic advice and speculative API-evolution arguments. Explain existing intent
or a trade-off only when it changes the recommendation.

An exact-range `suggestion` fence belongs under **Suggested fix**. Include a
complete focused failing test there only when useful as permanent regression
coverage, and name its destination. Longer necessary proof may be collapsed
under **Why this matters**. Keep Markdown and fences at column zero.

Set severity from concrete impact, not category alone. A substantial public
contract break is usually blocking; a new conversion inheriting a pre-existing
quirk may need only a non-blocking documentation correction. Do not elevate a
preference into a defect or pad the review with tooling-owned formatting issues.

## Area result and final summary

Return actionable findings in impact order, followed by one coverage line:
what was reviewed and what could not be assessed. Keep uncertainty or design
questions distinct from proven findings. If nothing is wrong, say so; the
coverage line is the whole body after attribution. Area workers do not decide
the combined verdict or deliver separately.

The coordinator merges findings about the same root cause and consolidates
coverage without losing skipped/blocked areas or the public surface reviewed.
Lead the final summary with the outcome, then concise coverage and material
limitations. Evidence stays with its finding rather than being repeated in a
summary transcript.

Final verdicts are `approve`, `approve with non-blocking comments`,
`changes requested`, or `blocked` when the review could not run (include the
decisive diagnostic). Never infer approval from an unassessed area. On the
requester's own PR, act as an investigative assistant and omit `Verdict:`
framing; delivery handles the corresponding no-vote/COMMENT behavior.
