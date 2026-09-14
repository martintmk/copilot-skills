---
name: review-delivery
description: >
  Post an AI-attributed code review to a GitHub pull request, Azure DevOps pull
  request, or a local report, with correct anchoring, severity labels, comment
  shape, and verdict. Shared delivery mechanics for the review-* skills, which
  produce findings but do not own posting. Use whenever a review is finished and
  needs to be delivered, including a clean approval, or when a posted review
  failed to anchor. Not for producing findings, and not for non-review PR
  comments.
---

# Review Delivery

Turn finished findings into one posted review. Every message is explicitly
attributed to an AI agent, anchored to the code it is about, and honest about
what was verified.

## Findings contract

Every `review-*` skill emits findings in this shape, so a single review reads the
same way no matter which areas ran. Each skill adds its own area-specific field
and evidence rule within the two sections below; none of them redefine this shape.

1. **Order by impact**, most consequential first. Return only actionable
   findings — never pad a review to look thorough.
2. **Anchor each finding as `path:line`** and name the symbol it is about.
3. **Use the shared comment shape below.** Put the claim and concrete consumer-,
   operator- or runtime-visible consequence under **Why this matters**. Put the
   specific correction under **Suggested fix**, with a `suggestion` block only
   when it is the exact replacement for the anchored range.
4. **Label severity from impact, not category**, in the attribution line rather
   than as a prose prefix, using only this vocabulary:
   - no severity qualifier — blocking;
   - `Non-blocking` — a real issue that should not gate the change;
   - `Nit` — cosmetic;
   - `Design note, no change requested` — an observation being recorded.
5. **Keep evidence honest and compact.** Distinguish reproduced results from
   reasoned claims under **Why this matters**, quoting the decisive result when
   it establishes the issue. Do not add a routine `Verified:` paragraph or
   narrate the investigation. State material uncertainty rather than guessing,
   and raise an unprovable behavioral suspicion as a question, not a finding.
6. **End the area's output with one coverage line** naming what it actually
   reviewed and what it could not assess — even when nothing was wrong. Do not
   repeat coverage in every comment.
7. **Say so plainly when there are no findings.** The coverage line is then the
   whole body after the AI attribution. Never manufacture findings.

**Verdict vocabulary**, used by the summary and by report-only skills:
`approve`, `approve with non-blocking comments`, `changes requested`, or
`blocked` when the review could not run at all (state the decisive diagnostic).

## Voice and severity

- **Attribute every message to the AI.** Start the summary, every GitHub inline
  comment, every Azure DevOps thread, and each local report with
  `**Posted by an AI agent**` on its own line, followed by a blank line. For a
  finding that needs a severity qualifier, put it inside the same bold line:
  `**Posted by an AI agent · Non-blocking**`, `**Posted by an AI agent · Nit**`,
  or `**Posted by an AI agent · Design note, no change requested**`. Use this
  attribution on each finding in a report too. Do not prepend another marker,
  wrap the line in backticks, quote or indent it, or imply the requester wrote
  the review.
- **One claim per comment, two short sections.** Normally one or two sentences
  under each heading. Lead with consumer or runtime impact, not compiler
  internals. Include concrete evidence when it establishes the issue, but omit
  investigation narration, redundant detail, and speculative API-evolution
  arguments. Brevity must not weaken the underlying investigation or hide a
  material limitation.

### Comment shape

```markdown
**Posted by an AI agent · Non-blocking**

**Why this matters**
<Concrete problem and consumer/runtime impact, with decisive evidence when useful.>

**Suggested fix**
<Specific correction and, only when useful, why it fits.>
```

Choose the severity qualifier from impact; the example is not a default.
Use these exact headings for every actionable finding, including standalone
reports. Fold area-specific evidence into **Why this matters** and recommendations
into **Suggested fix** rather than adding per-field sections. A design note with
no change requested uses only **Why this matters**; do not invent a fix. Clean
summaries and coverage-only reports need attribution, not empty finding sections.

Put an exact-range `suggestion` fence under **Suggested fix**. When a failing test
is useful permanent coverage, include its complete focused `rust` fence there,
optionally inside `<details>`, and name the module/file where it belongs. Keep
longer proof material only when needed, optionally collapsed under **Why this
matters**. Do not indent prose or fences.

### Judgment and anchoring

- **Anchor precisely.** Attach to the right line and name the symbol; quote the
  exact value. Use in-body line references (`L56-59`) only to point at a
  *different* line than the anchor.
- **Keep severity in the attribution line.** Use the qualifiers in the findings
  contract, with no qualifier for a blocking finding. State the verdict in the
  summary, not on each finding.
- **Set severity from impact, not category.** A public-contract or semver issue
  is usually blocking, but weigh novelty, real consumer impact, precedent,
  mitigation and scope: a new public conversion that merely inherits a
  pre-existing quirk is `Non-blocking` with a doc-note ask, not a block.
- **Acknowledge intent when it affects the recommendation.** Explain a relevant
  constraint or trade-off briefly, then commit to a correction; do not add a
  rationale paragraph by default.
- **Retract plainly** when a re-run shows you were wrong.
- **On the requester's own PR**, act as an investigative assistant, not a
  gatekeeper: use `event:"COMMENT"` / no ADO vote, and drop `Verdict:` framing.

## Avoid low-signal comments

Formatting and import order (tooling owns it); speculation presented as fact;
repeated lint/CI output that reveals no design or correctness problem; generic
Rust advice not tied to a changed line; restating the code; and large proof
listings that are mostly harness setup.

## Output and verdict

A structured summary plus findings in impact order. Build the summary from
components, not a fixed template; it may be one sentence. After the standalone
`**Posted by an AI agent**` line and a blank line, lead with the overall outcome,
then concise coverage and material limitations. State the verdict when
applicable. Keep decisive evidence with its finding rather than repeating the
investigation in the summary. Findings placed in the summary because they cannot
be anchored still use the shared two-section comment shape.

If a review area's gate produced no finding, say so explicitly rather than
silently omitting it. Reach a verdict from the findings alone, using the verdict
vocabulary in the findings contract above. If nothing meaningful is wrong, say
so; never manufacture findings.

## GitHub PR

Post one **review**:

```
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input review.json
```

`review.json` is `{ body, event, commit_id, comments[] }`; `event` is
`REQUEST_CHANGES`, `COMMENT` or `APPROVE`; `commit_id` is the final verified head
SHA; each comment is `{ path, line, side, body }` (+ `start_line` / `start_side`
for a range). `side` is `RIGHT` for added/changed lines, `LEFT` only for a
removed line. Mechanics that bite:

- **Generate the JSON with a script written to disk**, then run it — a `.py`
  file that builds the payload and `json.dump`s it, posted with `--input`. Avoid
  inline `python -c` (shell quoting mangles bodies carrying backticks, newlines
  and ` ```suggestion ` blocks), and **never pass a body with `-f body=@file` /
  `--raw-field`** — that posts the literal path `@file`, not its contents. Build
  multiline bodies with `textwrap.dedent(...).strip()` so Markdown begins at
  column zero.
- **Line numbers are post-change lines on the head** — read them from the
  checked-out head or `gh pr view <n> --json headRefOid`, pin the review with
  `"commit_id": "<headRefOid>"`, and re-check the head has not moved immediately
  before posting. This also works for fork PRs.
- **The anchor must fall inside a diff hunk** (context lines count). If a
  finding's ideal line is outside the diff, put it in the summary rather than
  mis-anchoring.
- **Single line vs range.** For a single-line comment provide `line` and omit
  `start_line`; for a range, `start_line` must be strictly less than `line`. A
  ` ```suggestion ` block replaces exactly the anchored range.
- **Validate before posting.** Assert the summary and every inline comment start
  with one of the exact bold attribution lines above, followed by a blank line;
  reject old, substituted, backticked, quoted, indented or run-in prefixes.
  Assert every actionable finding has **Why this matters** then **Suggested
  fix**, each on its own line. Summary-only bodies and no-change design notes
  follow the exceptions above. Assert no body has leading whitespace and fences
  open at column zero. Inspect the generated `comments[].body` values, then
  fetch `headRefOid` once more and assert it equals `review.json.commit_id`;
  regenerate anchors or stop if it moved.
- **`APPROVE`/`REQUEST_CHANGES` are rejected on your own PR** → use
  `event:"COMMENT"` and state the verdict in the body.
- **Verify after posting.** Read the comments back. A `422` is usually a bad
  anchor or validation error (fix and repost); `403`/`429` mean permissions or
  rate limiting (back off), not a bad anchor. Confirm the returned bodies render
  with the intended structure, then clean up the payload script and probes.

## Azure DevOps PR

- Get the diff with `ado-repo_pull_request` `action:get_changes` (paginate;
  fetch PR metadata for author/head and the latest iteration). Every ADO call
  needs `orgName` plus `project` and `repositoryId`.
- Post each inline finding with `ado-repo_pull_request_thread_write`
  `action:create` — `orgName`, `repositoryId`, `pullRequestId`, `project`,
  `content`, `filePath`, and `rightFileStartLine`/`rightFileEndLine` (the tool
  anchors on the head side). ADO offsets are 1-based: for a whole line pass `1`
  for `rightFileStartOffset` and the exact character count + 1 for
  `rightFileEndOffset`; `0` and arbitrary large offsets are rejected. Put the
  summary in one more `create` with no file path.
- Start each `content` with the same standalone bold attribution line and blank
  line, then use the shared comment shape for findings. Apply the same body
  validation as GitHub before posting; ADO threads stand alone, so attribution
  and sections must be present in each finding thread.
- Threads are posted one at a time and are not atomic: read them back
  (`ado-repo_pull_request_thread` `action:list`) to confirm each anchored, and
  recover any that failed rather than leaving a half-posted review.
- Cast the verdict with `ado-repo_pull_request_write` `action:vote`: `Approved`
  / `ApprovedWithSuggestions` (approve with nits) / `WaitingForAuthor` (changes
  requested). Do not vote on your own PR.

## Local diff (no PR)

Return the review as a report in chat with the standalone AI attribution,
findings in the shared two-section shape, a coverage line and verdict; post
nothing. Do not replace the finding sections with a table.

For posted reviews, report the review URL and a one-line-per-finding table in
chat rather than pasting the whole review back.
