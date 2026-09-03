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

## Voice and severity

- **Attribute every message to the AI.** Start the summary, every GitHub inline
  comment, and every Azure DevOps thread with the exact inline prefix
  `[AI AGENT]: `. Never substitute another marker, change the casing, wrap it in
  backticks, put it on its own line, or indent it. Never write in the
  requester's first person or imply they wrote the review.
- **One claim per comment, normally compact.** State the defect, the
  verification (`Verified: …`) when you ran one, and the fix. Lead with consumer
  or runtime impact, not compiler internals. Normally two short paragraphs before
  any proof block: a short finding gets a sentence, and you spend extra length
  only where a contract proof or trade-off earns it.
- **Comment shape:**

  `[AI AGENT]: Concern and consumer/runtime impact.`

  `Verified: <probe or test> -> <decisive result>. Suggested fix: <specific change>.`

  Omit the verification sentence for reasoned API/design findings. Add a
  `suggestion` fence only when it is the exact replacement for the anchored
  range. When a failing test is useful permanent coverage, add its complete
  `rust` fence after these paragraphs, optionally inside `<details>`, then say
  `Please add this test to <module/file>.` Do not indent prose or fences.
- **Anchor precisely.** Attach to the right line and name the symbol; quote the
  exact value. Use in-body line references (`L56-59`) only to point at a
  *different* line than the anchor.
- **Label severity only when it is not obvious:** `nit:` for cosmetics,
  `non-blocking:` for a real but non-gating issue, `**Design note, no change
  requested:**` for an observation. State the verdict in the summary, not on
  each finding.
- **Set severity from impact, not category.** A public-contract or semver issue
  is usually blocking, but weigh novelty, real consumer impact, precedent,
  mitigation and scope: a new public conversion that merely inherits a
  pre-existing quirk is `non-blocking:` with a doc-note ask, not a block.
- **Acknowledge intent, then decide.** Name why the code is the way it is before
  correcting it, lay out the trade, and commit to a recommendation.
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
components, not a fixed template; it may be one sentence. After the
`[AI AGENT]: ` prefix, compose in this order when the components apply: what you
verified (the targeted tests/probes you ran and the CI status you relied on, not
a re-run of the full suite); a coverage line naming what was reviewed; the
design findings; the remaining correctness, dependency, performance, test and
doc findings; the verdict; and any `Design note, no change requested`.

If a review area's gate produced no finding, say so explicitly rather than
silently omitting it. Reach a verdict — `approve`, `approve with non-blocking
comments`, or `changes requested` — from the findings alone. If nothing
meaningful is wrong, say so; never manufacture findings.

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
  with the exact `[AI AGENT]: ` prefix; reject backticked, differently cased,
  line-separated, indented or substituted markers. Assert no body has leading
  whitespace and fences open at column zero. Inspect the generated
  `comments[].body` values, then fetch `headRefOid` once more and assert it
  equals `review.json.commit_id`; regenerate anchors or stop if it moved.
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
- Start each `content` with `[AI AGENT]: ` followed immediately by the first
  paragraph; ADO threads stand alone, so this is where attribution lives.
- Threads are posted one at a time and are not atomic: read them back
  (`ado-repo_pull_request_thread` `action:list`) to confirm each anchored, and
  recover any that failed rather than leaving a half-posted review.
- Cast the verdict with `ado-repo_pull_request_write` `action:vote`: `Approved`
  / `ApprovedWithSuggestions` (approve with nits) / `WaitingForAuthor` (changes
  requested). Do not vote on your own PR.

## Local diff (no PR)

Return the review as a report in chat — the local-validation line, findings
table and verdict — and post nothing.

For any mode, report the review URL (when posted) and a one-line-per-finding
table; summarise in chat rather than pasting the whole review back.
