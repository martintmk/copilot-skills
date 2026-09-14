---
name: pr-review-radar
description: >
  Find unreviewed open PRs in specified GitHub and Azure DevOps repositories:
  recent PRs, foundational APIs, infrastructure, or mentions of the user. Send
  only previously unreported PRs to Teams self-chat. Use for review
  opportunities, repository monitoring, or recurring review digests; not code
  review, unanswered-feedback discovery, or acting on feedback.
---

# PR Review Radar

Find open PRs the user has not reviewed; send one Teams digest only when there
are new matches. This is discovery, not code review or permission to act on a
PR.

Follow the [radar run procedure](radar-run.md) for setup, collection discipline,
state recovery, and delivery. This skill owns eligibility and PR-level
deduplication. Its only fallback repository list is its own saved list.

## Persistent state

Use `state.json` in the `pr-review-radar` state directory with this shape:

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
    "https://github.com/owner/repository/pull/123": {
      "reportedAt": "2026-09-04T08:00:00Z"
    }
  }
}
```

The canonical browser URL is the PR identity. A PR is new only when its URL is
absent from `reported`; changes to its title, branch, head commit, labels, or
review status do not make it new again.

## Collection

For every configured repository:

1. List all open PRs, including those older than seven days.
2. Exclude draft PRs.
3. Exclude PRs authored by the current user.
4. Exclude PRs already present in `reported`.
5. Exclude PRs the user has already reviewed:
   - **GitHub:** any submitted PR review authored by the current GitHub login
     counts, regardless of review state, including commented or dismissed
     reviews. Pending reviews and ordinary issue comments do not count.
   - **Azure DevOps:** a non-zero reviewer vote or a non-system PR thread comment
     authored by the current Azure DevOps identity counts as a review.
6. For remaining PRs, reuse the title, description, labels, review requests,
   discussions, and available linked work items. Inspect changed paths and
   targeted public-symbol/API or diff evidence to classify thematic matches;
   do not perform a full code review. Check all relevant surfaces for mentions.

## Selection

Select a candidate when at least one of these is true:

1. **Recent:** creation time is within the inclusive interval from the captured
   scan instant minus seven 24-hour periods through the scan instant.
2. **Foundational API:** it adds or materially changes shared telemetry,
   resilience, retry, timeout, circuit-breaker, HTTP client, transport,
   authentication, serialization, runtime, configuration, diagnostics, or other
   broadly consumed API surface.
3. **Infrastructure:** it fixes or materially improves CI/CD, build or release
   tooling, test infrastructure, developer tooling, package publishing,
   deployment, repository automation, shared environments, or reliability of
   those systems.
4. **Mentioned:** the current user is explicitly requested as a reviewer or
   mentioned in the PR title, description, review request, linked discussion, or
   PR comments using an identity attributable to that user.

The seven-day limit applies only to **Recent**; older PRs can match any other
reason. Use creation time, not updated time. Do not classify from keywords when
changed paths or the diff contradict them. Foundational and infrastructure
matches require changed-file or API evidence; a recent PR needs no thematic
match.

Record every matching reason; never invent impact or downstream usage counts.

## Report

Use the shared HTML contract. Assign each PR to its highest-ranked matching
group, in this order:

1. Mentioned.
2. Foundational API.
3. Infrastructure.
4. Recent only.

Within a group, show newest creation time first, then canonical URL as a stable
tie-breaker. Title the digest **PR Review Radar**; count **new pull requests
awaiting review**.

Each **Why review:** explains why the change is interesting or consequential,
not merely its category. Include every matching reason, grounded in the
description, changed paths, API evidence, or discussion. For example:
“Changes the shared HTTP retry policy and public error classification. Opened
two days ago; you were requested as a reviewer.”

On a complete scan with no new match, report locally: no new PRs awaiting review.

## Completion

Use the shared delivery transaction to commit each included canonical URL with
`reportedAt` set to the confirmed delivery timestamp. Report the number of PRs
sent; distinguish a successful send from any state-persistence error.
