# Shared Findings Contract

Read when producing or validating intermediate area findings, filtered API
reports, standalone reports or posted threads. This is their sole shared shape;
specialists add evidence, not templates. Do not defer fields until delivery.
`review-public-docs` returns a bundle, not findings, and is exempt.

## Attribution and comment shape

Start every summary, finding, standalone report and PR thread with an exact bold
attribution below on its own first line, then a blank line. Never imply requester
authorship. Reject legacy, quoted, backticked, indented or run-in prefixes.

```markdown
**Posted by an AI agent · Non-blocking**

**<Concrete defect affecting a named surface>**

**Problem**
<Current defect and decisive evidence.>

**Why this matters**
<Consumer/runtime impact.>

**Suggested fix**
<Specific correction.>
```

The example's severity is not a default. Allowed first lines:

- `**Posted by an AI agent**` - summaries, clean reports, blocking findings.
- `**Posted by an AI agent · Non-blocking**` - real issue that should not gate.
- `**Posted by an AI agent · Nit**` - cosmetic.
- `**Posted by an AI agent · Design note, no change requested**` - observation,
  not a change request.

Every finding, **including Design notes**, requires a concise bold title naming
the surface without relying on its anchor, then **Problem** and
**Why this matters**, in order. Actionable findings, including nits/non-blocking issues,
use a diagnosis title (what is wrong, not topic/fix) and end with **Suggested fix**.
Design notes use an observation title: **Problem** explains the evidenced
constraint/trade-off without asserting a defect; **Why this matters** explains
its significance. They omit **only Suggested fix**.

Use those exact headings, normally one or two sentences each. Explain rather
than repeat the title. Clean summaries/coverage-only reports need attribution,
not empty finding sections. Keep questions conditional, not disguised defects.
Markdown prose and fences start at column zero.

## Evidence, correction and severity

One root cause per finding. Name the symbol; put exact `path:line` in location
metadata, using body line references only for another location. Output-only API
uses exact public paths, never source inspection for line numbers.

Preserve domain proof and uncertainty: evidence/constraint under **Problem**,
impact under **Why this matters**, actionable correction under **Suggested fix**.
Omit routine `Verified:`, investigation narration, redundant code, generic advice
and speculative API-evolution arguments. Explain intent/trade-offs only when
they affect the recommendation or constitute a Design note.

Put exact-range `suggestion` fences under **Suggested fix**; include a complete
focused failing test only as useful permanent regression coverage, naming its
destination. Collapse longer necessary proof under **Problem**.

Severity follows concrete impact: substantial public contract breaks usually
block; new conversions inheriting old quirks may need only non-blocking doc
corrections. Preferences are not defects; omit tooling-owned formatting.

## Area result and final summary

Return actionable findings in impact order, then one coverage line: reviewed
scope and unassessed areas. Separate uncertainty/design questions from proven
findings. For no findings, say so in that line, the whole body after attribution.
Area workers neither decide the combined verdict nor deliver separately.

The coordinator merges root causes and coverage without losing public surface,
not-applicable or blocked areas; missing roster entries cannot hide behind a
clean summary. Lead the final summary with outcome, coverage and material
limitations; leave evidence with its finding.

Verdicts: `approve`, `approve with non-blocking comments`, `changes requested`,
or `blocked` with the decisive diagnostic when review could not run. Never
approve unassessed areas. On the requester's own PR, omit `Verdict:` framing;
delivery applies COMMENT/no-vote behavior.
