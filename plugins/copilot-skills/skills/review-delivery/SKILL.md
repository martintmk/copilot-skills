---
name: review-delivery
description: >
  Deliver finished, merged review findings once to GitHub, Azure DevOps or a
  local report, with precise anchors and the shared AI-attributed comment
  format. Use when a review is ready to deliver, including a clean review,
  or to recover failed posting. Not for generating findings, formatting an
  area's intermediate output, or non-review PR comments.
---

# Review Delivery

Run in the coordinator's single fresh delivery worker under the
[isolation protocol](../review-lens/worker-isolation.md), also for direct
delivery requests; an already assigned worker does not dispatch itself again.
For finding format alone, read the [findings contract](findings-contract.md);
area workers never post.
Accept the completed findings, coverage, reviewed revisions and delivery mode.
Do not repeat the investigation or override an explicit report-only request.

For a Review Lens result, require its snapshot-matching `coverageManifest`
covering every entry in the [required roster](../review-lens/SKILL.md).
Missing workers, skipped passes or `blocked` records prevent publication, even
when the supplied findings are empty. A moved descendant head may additionally
provide the coordinator's complete `findingRefresh` record. This permits only a
best-effort `COMMENT`, not completed current-head coverage, approval or changes
requested. This gate does not broaden a directly requested single-area review.

## Prepare the review

1. Apply the shared contract to the merged findings. Omit newly duplicated
   discussion points if a stale snapshot needs refreshing; do not rerun the
   area passes. Summarize outcome, coverage (including public surface), material
   limitations and the supported verdict.
2. Confirm the head/diff state still matches the evidence. Normally, a moved
   head or changed local file returns affected claims to the coordinator before
   posting; re-anchoring alone does not validate old evidence. For a descendant
   head, accept a complete coordinator-produced `findingRefresh` only when it
   identifies the exact reviewed/current heads and target, classifies every
   original finding, omits `resolved` and `uncertain` findings, and updates
   `updated` evidence and anchors against current source. The summary must say
   that the added commits did not receive full specialist coverage.
3. Check every body against the contract: exact bold attribution on its own
   first line, followed by a blank line. Every finding, including a design note,
   requires a concise bold title, **Problem** and **Why this matters**, in order.
   Actionable findings use a diagnosis title and end with **Suggested fix**.
   Design notes use an observation title and describe the constraint or
   trade-off with evidence under **Problem**; omit only **Suggested fix**.
   Clean summaries and coverage-only reports do not need finding sections.
   Reject legacy, quoted, backticked, indented or run-in prefixes. Prose and
   fences start at column zero. Inspect serialized bodies, not only the source
   template.
4. After any best-effort finding refresh, always use GitHub `COMMENT` / no ADO
   vote, regardless of the original verdict. On the requester's own PR, also
   use GitHub `COMMENT` / no ADO vote and omit
   `Verdict:` framing. GitHub also rejects `APPROVE` and `REQUEST_CHANGES` when
   the authenticated poster is the author; use `COMMENT` in that case.

## GitHub

Post one review, not a series of independent inline reviews:

```text
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input review.json
```

The payload is `{ body, event, commit_id, comments[] }`. `event` is
`REQUEST_CHANGES`, `COMMENT` or `APPROVE`; `commit_id` pins the reviewed head.
Each comment is `{ path, line, side, body }`, plus `start_line` / `start_side`
only for ranges.

- **Serialize, do not shell-quote bodies.** Generate the payload with a
  file-based script and a JSON serializer, then post with `--input`. Inline
  `python -c` quoting mangles backticks, newlines and suggestion fences.
  Never use `-f body=@file` / `--raw-field`: it posts the literal path.
  Normalize multiline indentation (for Python, `textwrap.dedent(...).strip()`).
- **Anchor to the reviewed diff.** `line` is a head-side line for `RIGHT`
  (added/changed code); use `LEFT` only for removed code. The anchor must be
  inside a hunk, including context lines. Put out-of-hunk findings in the
  summary with the same finding shape, never at an unrelated anchor.
- **Ranges must be exact.** A single-line comment omits `start_line`; a range
  requires `start_line < line`. A `suggestion` replaces exactly that range.
- **Recheck immediately before posting.** Fetch `headRefOid` and compare it
  with `review.json.commit_id`. On further movement, return to the coordinator
  for another exact-delta finding refresh; never silently post against the
  superseded commit.
- **Read back the review and comments.** Confirm anchors and body structure.
  A `422` usually means bad anchors/validation; correct the rejected payload.
  A `403`/`429` means permissions/rate limiting, not a reason to change anchors.
  After an ambiguous timeout, look for the posted review before retrying so
  recovery does not duplicate it.

## Azure DevOps

Discover the configured metadata, diff, thread-create/list and voting tool
schemas; use their actual operation names and organization fields. A tool from
another ADO deployment is not evidence that the same tool exists here.

- Reuse the reviewed diff/iteration and head from context. Confirm the current
  head still matches before writing; if it moved, return to the coordinator.
- Create one thread per inline finding and one unanchored summary thread.
  Supply the target organization, project, repository and PR plus the content
  and head-side file/line range required by the discovered schema.
- For tools exposing `rightFileStartOffset` / `rightFileEndOffset`, offsets are
  1-based: whole-line start is `1`, end is the exact character count + 1.
  Do not use `0` or arbitrary large offsets.
- Threads are not atomic. Record successful thread IDs and read them back with
  the list/read tool; recover only missing or failed writes, including after an
  ambiguous response, rather than reposting the whole review.
- Vote only after intended threads are confirmed and only when permitted.
  Map the verdict to the tool's supported equivalents of approved,
  approved-with-suggestions or waiting-for-author. Never vote on the requester's
  or authenticated poster's own PR. If voting is unavailable, report that
  limitation; do not invent a tool or claim the vote succeeded.

## Report-only and completion

For local or report-only work, return attributed findings in the shared
finding format, followed by coverage and verdict; post nothing. Preserve the
same design-note and clean-summary exceptions. Do not replace full findings
with a table.

After external delivery, report the review URL and a compact one-line-per-finding
table in chat rather than pasting the whole review back. Report partial delivery
plainly. Remove owned payload scripts and return resource ownership to the
coordinator for shared cleanup.
