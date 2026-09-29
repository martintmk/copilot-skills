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

Publish a finished review exactly once. You do not investigate or change
findings; you check their shape, write the summary, choose the provider action
and post.

## Inputs

You need:

- The merged findings in the [findings contract](findings-contract.md).
- Summary facts: what was reviewed, what was not and why, and what was not run.
- The outcome: **complete** with a verdict, **incomplete** (some areas could not
  be reviewed), or **refreshed** (existing findings rechecked on newer commits).
- The pinned head commit, the PR and the repository.
- The mode: post, or report only.

If any of these is missing, ask the caller. Do not guess, and do not run review
passes to fill gaps.

## Prepare

1. **Check every finding** against the findings contract: attribution line,
   title, **Problem**, **Why this matters** and, for actionable findings,
   **Suggested fix**. Reject violations before writing anything. Check the
   serialized bodies, not just your template.
2. **Write the summary for the PR author.** Start with the attribution line.
   Then say, in plain words, what was reviewed, what was not and why, and what
   was not run. Name areas by topic ("public API", "tests"), not by skill or
   internal status. Avoid words such as worker, snapshot, manifest, isolated or
   falsification. Leave evidence in its finding.
   - Incomplete: follow the attribution with **Warning: Incomplete review**,
     list each topic not reviewed with a one-clause reason, and say no overall
     verdict is given.
   - Refreshed: say full coverage ended at the reviewed commit and later
     commits only had existing findings rechecked.
3. **Check the head.** Fetch the current head. If it differs from the pinned
   head, stop and return to the caller. Moving an anchor is not revalidation.

## Choose the provider action

| Situation | GitHub event | Azure DevOps vote |
| --- | --- | --- |
| Complete, verdict `approve` | `APPROVE` | Approved |
| Complete, verdict `approve with non-blocking comments` | `APPROVE`, keeping the comments | Approved with suggestions |
| Complete, verdict `changes requested` | `REQUEST_CHANGES` | Waiting for author |
| Incomplete | `COMMENT` | No vote |
| Refreshed | `COMMENT` | No vote |
| Code was not allowed to run (read-only review) | `COMMENT`; the summary may state the recommended verdict | No vote |
| The PR belongs to the requester or to the posting account | `COMMENT`, no `Verdict:` line | No vote |
| Report only | Nothing is posted | Nothing is posted |

When several rows apply, the most restrictive one wins.

Clean and nit-only complete reviews approve automatically; do not ask for
confirmation or fall back to `COMMENT`. Decide from all merged findings,
including unresolved ones not posted again. Zero new comments does not mean a
clean review. Approval wording in the summary does not replace the vote.

## GitHub

Post one review:

```text
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input review.json
```

Payload: `{ body, event, commit_id, comments[] }`; events:
`REQUEST_CHANGES`, `COMMENT`, `APPROVE`. Pin `commit_id` to the reviewed head or
current head for a refreshed review. Comments: `{ path, line, side, body }`, adding
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
  `review.json.commit_id`; if it moved, stop and return to the caller.
- Read back review/comments to confirm the submitted event, pinned commit,
  anchors and serialized body structure. An expected `APPROVE` must read back
  as `APPROVED`; COMMENT is not successful approval delivery.
  Correct rejected `422` payloads; `403`/`429` are permission/rate limits, not
  anchor failures. Reconcile ambiguous timeouts before retrying to avoid duplicates.

## Azure DevOps

Discover configured metadata, diff, thread-create/list and voting schemas;
use actual operations/organization fields, never another deployment's assumed tools.

- Pin reviewed diff/iteration/head; recheck before writes and stop if it
  moved.
- Create one thread per inline finding plus one unanchored summary. Supply
  organization/project/repository/PR, content and schema-required head-side ranges.
- `rightFileStartOffset`/`rightFileEndOffset` are 1-based: whole-line start `1`,
  end exact character count + 1, never `0` or arbitrary large values.
- Threads are non-atomic. Record successful IDs and read back bodies, anchors
  and iteration association. Reconcile ambiguous responses; recover only
  provably unapplied writes, never repost the whole review.
- Vote only after intended threads are confirmed and when the table above allows a vote.
  Apply that table using the provider's supported vote values,
  then read back the posting actor's vote to confirm the intended value.
  Unavailable voting must be disclosed; a missing required vote is incomplete
  delivery, never invented success.

## Report-only and completion

Report only: return the full findings and summary in chat. **Post nothing.** Do
not replace findings with a table.

After posting, return the review URL and one line per finding, not the whole
review. Say clearly if delivery was partial. Remove payload scripts you created.
