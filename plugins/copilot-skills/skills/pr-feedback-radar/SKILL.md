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

Discovery grants no answering, resolution or editing authority; unattended action
uses `feedback-autonomy`. For each scan, load the
[radar run procedure](../pr-review-radar/radar-run.md) directly, not the review
radar skill. This file owns feedback eligibility, deduplication and report shape.

## Scope and state

With neither explicit repositories nor a saved list, copy and save only
`repositories` from valid `pr-review-radar` state once, never `reported`.
Thereafter use this radar's list.

Include only PRs open at the scan instant and created in the inclusive interval
from scan minus seven 24-hour periods through scan. Updated/comment times cannot
admit older PRs. Include own, draft and already-user-reviewed PRs.

The `pr-feedback-radar` directory's `state.json` shape is:

```json
{
  "version": 1,
  "repositories": [{"provider": "github", "repository": "owner/repository"}],
  "reported": {
    "github|https://github.com/owner/repository/pull/123|review-comment|987": {
      "sourceUpdatedAt": "2026-09-04T08:00:00Z",
      "contentHash": "sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
      "reportedAt": "2026-09-04T09:00:00Z"
    }
  }
}
```

Keys combine provider and canonical PR URL with GitHub feedback kind and immutable
database ID (distinguish issue comments, review bodies, review-thread comments),
or Azure DevOps thread ID and comment ID.

SHA-256 hash bodies after normalizing line endings and insignificant surrounding
whitespace; preserve case/meaningful content. An absent key or changed hash is new
only while still awaiting response. Timestamp/whitespace-only edits, priority,
review state and head changes are not new. Human follow-ups have new IDs and may
reintroduce a reported PR. `sourceUpdatedAt` is edit time, falling back to creation
time only when no separate edit time exists.

## Evidence and human filter

Collect every eligible PR's GitHub issue comments, submitted review bodies,
review threads/replies, resolution state, review requests and latest review
states; or Azure DevOps threads/comments, status, reviewer votes and requests.
Retain immutable IDs, actor identity/type, creation/edit times, thread/reply
relationships, resolution and canonical links. Read merge-review/reviewer
policies as needed for blocking; cache within repository/branch scope.

Before interpreting requests, require proven human authorship:

- **GitHub:** require `User`; exclude `Bot`, `App`, `Mannequin`, `Organization`,
  deleted/missing actors, `[bot]` logins, generated events and
  repository/organization-identified automation/service accounts.
- **Azure DevOps:** require metadata identifying an individual; exclude system
  comments, build/project collection services, service principals, managed
  identities, extensions, pipelines and other automation.

Exclude ambiguous identities, bot-relayed human words and the user's own comments
from incoming feedback. User comments, review/status changes, reactions, requests,
commits and policy updates may corroborate expectation, answers or blocking, never
create items.

## Unanswered requests

Require another human's actual question, request/change, decision, re-review or
explicit acknowledgement request, directed through at least one of:

1. **Own PR:** a review comment/body or PR comment asks the PR author to change,
   explain, decide, confirm or answer.
2. **Direct address:** provider-identity mention, clear reply to the user's
   comment/review, or naming them as the person whose input is needed.
3. **Requested reviewer:** the user is actively requested and a human participant
   says ready for their review/re-review, asks their decision or responds to their
   earlier feedback.
4. **Follow-up:** a directed question/request after the user's last response in
   that conversation.

Authorship, assignment, review requests, participation or expertise alone never
supply the request. Omit ambiguous intent.

Close requests after a later material user response in-thread; for unthreaded
requests, require a later user comment clearly quoting, linking or answering that
specific request. Thread resolution, request withdrawal/dismissal or a later
human's explicit "no action/response needed" also closes them. Close when the
latest material exchange ends the request with acknowledgement, thanks, approval,
information, a rhetorical question or optional suggestion needing no
decision/action. Unrelated exchanges, commits, force-pushes, status changes,
reactions or elapsed time alone do not answer it.

Select the latest unanswered request per thread, retaining independent unresolved
threads as separate items. Deduplicate across provider surfaces; omit a whole
review summary when all its requests are represented by threads.

## Blocking and selection

Use **High priority - blocking** only with evidence the user's response currently
prevents someone proceeding:

- A human explicitly awaits or cannot proceed without the user's answer,
  decision, change, approval or re-review; this includes a reviewer blocked on
  review/dependent work by unanswered feedback on the user's PR.
- The user's active changes-requested/negative review blocks merge, and the
  author subsequently says it was addressed or requests their re-review.
- Applicable branch/reviewer policy specifically requires the user's outstanding
  approval or decision.

Record proof; say "one of the blockers" when other checks also block merge.
Unresolved threads, reviewer requests, urgency, unknown mergeability, failing CI
or insufficient approvals alone warrant only normal priority.

Record human author, expected response, unanswered-since time, direction/blocking
evidence, key, source edit time and hash. Exclude reported same-hash items.
Group new actionable items per PR; high priority requires an included blocking
item. Sort high priority first, then oldest included unanswered request, ascending
PR creation time, then canonical URL. Ranking/waiting time ignore unchanged old
feedback.

## Report

Use title **PR Feedback Radar**, count **pull requests with new feedback awaiting
your response**, and groups **High priority - you are blocking someone** and
**Response needed**.

**Why respond:** identify the human author/request, direction, oldest included
waiting time and exact blocking evidence for high priority; preserve separate
requests. Example: "Alex says the change is complete and asks you to re-review;
your active changes-requested review is one of the merge blockers. Waiting since
3 September."

Empty-result wording: no new human PR feedback awaiting the user's response.
Delivery entries contain included items' `sourceUpdatedAt`, `contentHash` and
`reportedAt`, never answered/unrepresented requests. Report PRs sent and how many
were high priority.
