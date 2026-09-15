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

Read [worker isolation](../review-lens/worker-isolation.md) before entry, including
direct requests, and [findings contract](findings-contract.md) when validating
output. Accept finished findings, coverage, pinned revisions and authorized mode;
do not investigate or override report-only.

For Review Lens, enforce its [roster/completion gate](../review-lens/SKILL.md):
require matching `coverageManifest` records even for empty findings. Missing,
skipped or blocked work prevents publication. This does not broaden single-area
requests. A valid descendant `findingRefresh` permits only best-effort COMMENT,
not approval, changes requested or completed current-head coverage.

## Prepare the review

1. Validate merged findings under the contract; omit newly duplicated discussion
   points during refresh. Summarize outcome, public-surface coverage, limitations
   and supported verdict, without rerunning area passes.
2. Revalidate head/diff and local file state. Movement returns claims to the
   coordinator; re-anchoring alone is not validation. Accept only Lens's valid,
   complete coordinator-produced `findingRefresh` for exact reviewed/current heads
   and target/base, classifying every original finding. Omit `resolved`/`uncertain`;
   require current evidence/anchors for `updated`. Disclose where full coverage
   ended and that added commits lacked the full roster. Retargets/rewrites require
   fresh review or blocking.
3. Validate attribution and finding shape in **every serialized body**, not just
   the template; reject contract violations before writing.
4. Refreshed findings always use GitHub `COMMENT`/no ADO vote. Requester-own or
   authenticated-poster-own PRs also require COMMENT/no vote and no `Verdict:`
   framing, never self-approval or `REQUEST_CHANGES`.

## GitHub

Post one review:

```text
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input review.json
```

Payload: `{ body, event, commit_id, comments[] }`; events:
`REQUEST_CHANGES`, `COMMENT`, `APPROVE`. Pin `commit_id` to the reviewed head or
valid refresh's current head. Comments: `{ path, line, side, body }`, adding
`start_line`/`start_side` only for ranges.

- Use a file-based script and JSON serializer with `--input`, not shell-quoted
  bodies/inline `python -c`. Never `-f body=@file`/`--raw-field` (literal path).
  Normalize indentation, e.g. `textwrap.dedent(...).strip()`.
- Anchor inside reviewed diff hunks, including context lines. `RIGHT` uses head
  lines; `LEFT` only removed code. Put out-of-hunk findings in the summary with
  the same contract, never unrelated anchors.
- Single lines omit `start_line`; ranges require `start_line < line`.
  A `suggestion` replaces exactly its range.
- Immediately before posting, compare fetched `headRefOid` with
  `review.json.commit_id`; movement returns to the coordinator for exact-delta
  refresh, never silently posts stale evidence.
- Read back review/comments to confirm anchors and serialized body structure.
  Correct rejected `422` payloads; `403`/`429` are permission/rate limits, not
  anchor failures. Reconcile ambiguous timeouts before retrying to avoid duplicates.

## Azure DevOps

Discover configured metadata, diff, thread-create/list and voting schemas;
use actual operations/organization fields, never another deployment's assumed tools.

- Pin reviewed diff/iteration/head; recheck before writes and return movement
  to the coordinator.
- Create one thread per inline finding plus one unanchored summary. Supply
  organization/project/repository/PR, content and schema-required head-side ranges.
- `rightFileStartOffset`/`rightFileEndOffset` are 1-based: whole-line start `1`,
  end exact character count + 1, never `0` or arbitrary large values.
- Threads are non-atomic. Record successful IDs and read back bodies, anchors
  and iteration association. Reconcile ambiguous responses; recover only
  provably unapplied writes, never repost the whole review.
- Vote only after intended threads are confirmed and when permitted above.
  Map to supported approved, approved-with-suggestions or waiting-for-author.
  Unavailable voting must be disclosed; a missing required vote is incomplete
  delivery, never invented success.

## Report-only and completion

Local/report-only: return full contract-formatted findings, coverage and permitted
verdict; **post nothing**. Do not substitute a table for findings.

After posting, return review URL and one-line-per-finding table, not the full
review. Disclose partial delivery. Remove owned payload scripts and return
resources to the coordinator for cleanup; retain caller-owned recovery evidence.
