---
name: pr-feedback-radar
description: >
  Find new unanswered human requests directed at the user on recently opened,
  still-open GitHub and Azure DevOps PRs. Prioritize demonstrably blocking
  responses and send a deduplicated Teams self-chat digest. Use for unanswered
  feedback, response reminders, or recurring feedback digests; not code review,
  acting on feedback, bot activity, or already-answered comments.
---

# PR Feedback Radar

Find conversations awaiting the user; send only new actionable feedback. This
does not authorize answering, resolving, or editing. Unattended workers use
`feedback-autonomy` to decide whether to act.

Follow the [radar run procedure](../pr-review-radar/radar-run.md) directly, not
the review radar skill, for setup, collection discipline, state recovery, and
delivery. The feedback rules and comment-level state below remain independent.

## Repository fallback and window

Only with neither explicit repository input nor a saved list, copy and save
`repositories` from valid `pr-review-radar` state once. Never copy `reported`.
Thereafter use this radar's list; explicitly empty is not missing.

Only inspect PRs open at the captured scan instant, with creation time in the
inclusive interval from exactly seven 24-hour periods before it through that
instant. Do not substitute updated/comment time or admit older PRs with recent
activity. Include drafts, the user's own PRs, and PRs the user already reviewed:
a person may still be waiting for an answer.

## Persistent state

Use `state.json` in the `pr-feedback-radar` state directory with this shape:

```json
{
  "version": 1,
  "repositories": [
    {
      "provider": "github",
      "repository": "owner/repository"
    }
  ],
  "reported": {
    "github|https://github.com/owner/repository/pull/123|review-comment|987": {
      "sourceUpdatedAt": "2026-09-04T08:00:00Z",
      "contentHash": "sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
      "reportedAt": "2026-09-04T09:00:00Z"
    }
  }
}
```

Use a stable feedback key:

- **GitHub:** provider, canonical PR URL, feedback kind, and immutable database
  ID. Distinguish issue comments, review bodies, and review-thread comments.
- **Azure DevOps:** provider, canonical PR URL, thread ID, and comment ID.

SHA-256 hash the body after normalizing line endings and insignificant
surrounding whitespace; preserve case and meaningful content. New means an
absent key or changed hash for an edited comment that still expects a response.
Timestamp-only/whitespace-only edits, priority, review state, and PR head changes
are not new. A human follow-up has a new comment ID, so a previously reported
PR can reappear. Use edit time as `sourceUpdatedAt`, or creation time when the
provider has no separate edit time.

## Collection

For every configured repository:

1. List every eligible PR within the creation-time window.
2. Collect all PR conversation surfaces:
   - **GitHub:** issue comments, submitted review bodies, review threads and
     their replies, thread resolution state, review requests, and latest review
     states.
   - **Azure DevOps:** PR threads and comments, thread status, reviewer votes
     and requests.
3. Preserve immutable IDs, actor identity/type, creation/edit times,
   thread/reply relationships, resolution, and canonical links.
4. Read merge-review/reviewer policies as needed to prove blocking; cache only
   within their repository/branch scope.
5. User comments, review changes, and thread status are evidence of answers or
   blocking, not incoming feedback.

## Human-only filter

Apply the human filter before interpreting a comment's meaning.

- **GitHub:** require a `User` actor. Exclude `Bot`, `App`, `Mannequin`,
  `Organization`, deleted/missing actors, `[bot]` logins, generated events, and
  accounts identified by repository/organization metadata as automation or
  service accounts.
- **Azure DevOps:** include an incoming comment only when identity metadata
  identifies an individual user. Exclude system comments, build and project
  collection service identities, service principals, managed identities,
  extensions, pipelines, and other automation accounts.
- Exclude the user's comments from incoming feedback; retain them as answer
  evidence. Exclude ambiguous identities rather than guess. A bot relaying
  human words is not an eligible incoming human comment here.

Reactions, status/review-request events, commits, and policy updates may
corroborate expectation or blocking; they cannot create a report item.

## Decide whether a response is expected

An eligible comment from another person must direct a question, request,
requested change, decision, re-review, or explicit acknowledgement request at
the user.

Direction must be supported by at least one of these:

1. **The user's PR:** an unresolved review comment, review body, or PR comment
   asks the author to change, explain, decide, confirm, or answer something.
2. **Direct address:** it mentions the user's provider identity, clearly replies
   to their comment/review, or names them as the person whose input is needed.
3. **Requested reviewer:** the user is an active requested reviewer and a human
   author or participant says the PR is ready for the user's review or re-review,
   asks for the user's decision, or responds to the user's earlier feedback.
4. **Follow-up:** a new question or request directed at the user follows their
   last response in that conversation.

Authorship, assignment, review requests, past participation, and expertise alone
do not establish expectation. A human comment must supply the request.

Treat the conversation as answered or no longer actionable when any of these is
true:

- The user posted a later material response in the same thread.
- For an unthreaded comment, a later user comment clearly quotes, links, or
  answers that specific request, not an unrelated one.
- The thread was resolved, the request was withdrawn or dismissed, or a later
  human comment explicitly says no response or action is needed.
- The latest material exchange closes the request with acknowledgement,
  thanks, approval, information, a rhetorical question, or an optional
  suggestion requiring no decision/action. Unrelated exchanges do not close it.

A commit, force-push, status change, reaction, or elapsed time alone is not an
answer. Deduplicate actions across provider surfaces: omit a whole review
summary when its requests are all represented by review threads.

Select the latest unanswered request per thread, not every comment. Keep
separate unresolved threads as separate items, grouped under one PR. Omit
genuinely ambiguous intent rather than send speculative reminders.

## Detect blocking feedback

Set **High priority - blocking** only with evidence that the user's response
currently prevents another person from proceeding:

1. A human explicitly says they are waiting on, blocked by, or unable to proceed
   without the user's answer, decision, change, approval, or re-review.
2. The user's active changes-requested or negative review is a merge blocker,
   and the author has since replied that it was addressed or has explicitly
   requested the user's re-review.
3. An applicable branch or reviewer policy specifically requires the user's
   outstanding approval or decision.
4. On the user's PR, a reviewer explicitly says their review or dependent work
   cannot continue until the user answers the pending feedback.

Record the evidence; if other checks also block merge, say "one of the blockers".

Unresolved threads, reviewer requests, urgency, unknown mergeability, failing
CI, and insufficient approvals alone do not prove blocking. Use normal priority
without evidence specific to this user's response.

## Selection and deduplication

Record each item's human author, expected response, unanswered-since time,
direction and blocking evidence, stable key, source edit time, and body hash.

Exclude already-reported same-hash items. Include a PR once when it has new
actionable items; make it high priority only if an included item proves blocking.

Sort high priority first, then oldest included unanswered request, ascending PR
creation time, and canonical URL. Ranking and waiting time use only included new
items, not unchanged old feedback.

## Report

Use the shared HTML contract with title **PR Feedback Radar** and count **pull
requests with new feedback awaiting your response**. Groups are
**High priority - you are blocking someone** and **Response needed**.

Each **Why respond:** identifies the human request, its direction at the user,
the oldest included request's waiting time, and exact blocking evidence for high
priority. Example: “Alex says the change is complete and asks you to re-review;
your active changes-requested review is one of the merge blockers. Waiting
since 3 September.”

Summarize the strongest concrete reasons without losing separate included
requests. On a complete scan with no new match, report locally: no new human PR
feedback awaiting the user's response.

## Completion

Use the shared delivery transaction to commit each included item's
`sourceUpdatedAt`, `contentHash`, and confirmed delivery `reportedAt`. Do not
mark previously answered or unrepresented requests as reported. Report the
number of PRs sent and how many were high priority; distinguish a successful
send from any state-persistence error.
