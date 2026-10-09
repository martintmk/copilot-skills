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
scope and unassessed areas, and a status (`done`, `not applicable` or
`could not review`, with a reason for the last two). Separate uncertainty/design questions from proven
findings. For no findings, say so in that line, the whole body after attribution.
Area workers neither decide the combined verdict nor deliver separately.

The coordinator merges root causes and coverage without losing any area. A
missing area cannot hide behind a clean summary. Lead the final summary with the
outcome, what was reviewed and material limits, in plain words for the PR
author; leave evidence with its finding. When a Review Lens review is
incomplete, put a bold warning right after the attribution, list each topic not
reviewed with a one-clause reason, and say no overall verdict is given.
Findings from reviewed areas still count; never imply the other topics were
checked or clean.

Every Review Lens summary also includes its mandatory **Public API changes**
and **Integration tests** report blocks. Include both after the overview,
including in clean, incomplete and refreshed reviews. A report that could not
finish says **Could not assess** with its reason; it never silently becomes a
clean result.

Use this summary shape:

```markdown
**Posted by an AI agent**

**Warning: Incomplete review**

I reviewed <topics>. I could not check:

- <Topic>: <plain one-clause reason>

No overall verdict is given. The comments below come from the reviewed areas.

### Public API changes

**Could not assess**

The public API capture could not be produced with the required toolchain.

### Integration tests

**No Integration Test Changes**
```

Verdicts: `approve`, `approve with non-blocking comments` or
`changes requested`. An unfinished review has no verdict. For a complete review, automatically select `approve` when there are no findings and
`approve with non-blocking comments` when the only actionable findings are
cosmetic `Nit`s. Design notes requesting no change do not prevent approval.
Do not ask for an additional approval confirmation.

An incomplete Review Lens result has no public `Verdict:` and posts as
COMMENT with no vote. The warning and list of unreviewed topics replace the
verdict; do not suppress supported findings because another area could not
finish.

Decide from the complete merged result, including still-applicable unresolved
findings whose duplicate posts were omitted, not the number of comments created.
Never relabel substantive issues as nits to qualify or approve unassessed areas.
Delivery's report-only, ownership and finding-refresh restrictions take
precedence. On requester-owned or authenticated-poster-owned PRs, omit `Verdict:`
framing; delivery applies COMMENT/no-vote behavior.
