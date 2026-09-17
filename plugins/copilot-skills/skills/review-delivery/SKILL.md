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

For Review Lens, enforce its
[roster/publication gate](../review-lens/SKILL.md#coverage-manifest-completion-and-publication-gates):
require matching `coverageManifest` records even for empty findings. Missing or
skipped work prevents publication; blocked rows permit an incomplete review only
when the coordinator sets `reviewPublishable=true` under that gate. This does not
broaden single-area requests. A valid descendant `findingRefresh` permits only
best-effort COMMENT, not approval, changes requested or completed current-head
coverage.

## Prepare the review

1. Validate merged findings under the contract; omit newly duplicated discussion
   points during refresh. Summarize outcome, public-surface coverage, limitations
   and supported verdict, without rerunning area passes.
   For a publishable incomplete Lens review, put the incomplete-coverage warning
   immediately after the attribution, name every blocked skill/area and concise
   diagnostic, and say that no verdict is issued.
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

## Automatic approval

For authorized posting of a complete, current-head review, apply the
[verdict rules](findings-contract.md#area-result-and-final-summary): clean and
nit-only reviews must approve automatically, not merely recommend approval in
the summary or default to COMMENT. No additional confirmation is needed.
Evaluate all merged findings, including unresolved duplicates not posted again;
zero new inline comments alone is not evidence of a clean review.

A publishable incomplete Lens review always uses GitHub `COMMENT` or no ADO vote.
It may publish supported findings from completed areas, including substantive
findings, but must not approve, request changes, or present a combined verdict.

Map the supported combined verdict to the provider action:

| Verdict | GitHub event | Azure DevOps vote |
| --- | --- | --- |
| `approve` | `APPROVE` | Approved |
| `approve with non-blocking comments` | `APPROVE`, retaining the comments | Approved with suggestions |
| `changes requested` | `REQUEST_CHANGES` | Waiting for author |
| `blocked` | `COMMENT` only when Lens marks the incomplete review publishable | No vote |

Report-only mode never writes. Ownership and finding-refresh restrictions above
override this mapping; incomplete required coverage never authorizes approval or
request changes. A summary's approval wording is not a substitute for a provider
vote.

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
- Read back review/comments to confirm the submitted event, pinned commit,
  anchors and serialized body structure. An expected `APPROVE` must read back
  as `APPROVED`; COMMENT is not successful approval delivery.
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
  Apply the verdict mapping above using the provider's supported vote values,
  then read back the posting actor's vote to confirm the intended value.
  Unavailable voting must be disclosed; a missing required vote is incomplete
  delivery, never invented success.

## Report-only and completion

Local/report-only: return full contract-formatted findings, coverage and permitted
verdict; **post nothing**. Do not substitute a table for findings.

After posting, return review URL and one-line-per-finding table, not the full
review. Disclose partial delivery. Remove owned payload scripts and return
resources to the coordinator for cleanup; retain caller-owned recovery evidence.
