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

Discovery only, not code review or permission to act. For each scan, load the
[radar run procedure](radar-run.md). This file owns eligibility, PR deduplication
and report shape; its only repository fallback is its own saved list.

## State

The `pr-review-radar` directory's `state.json` shape is:

```json
{
  "version": 1,
  "repositories": [{"provider": "github", "repository": "owner/repository"}],
  "reported": {
    "https://github.com/owner/repository/pull/123": {
      "reportedAt": "2026-09-04T08:00:00Z"
    }
  }
}
```

Canonical browser URLs are permanent identities: only absence from `reported`
makes a PR new, never title, branch, head, label or review-status changes.

## Eligibility

Scan every configured repository's open PRs regardless of age. Exclude drafts,
own PRs, reported URLs and already-user-reviewed PRs:

- **GitHub:** any submitted review by the current login counts, including
  commented/dismissed reviews; pending reviews and issue comments do not.
- **Azure DevOps:** the current identity's non-zero reviewer vote or non-system
  PR thread comment counts.

Use titles, descriptions, labels, review requests, discussions and available
linked work items; check all relevant mention surfaces. Inspect changed paths
and targeted public-symbol/API or diff evidence, not a full code review.

Select any matching lens:

- **Recent:** created in the inclusive interval from scan minus seven 24-hour
  periods through scan, not updated time.
- **Foundational API:** adds/materially changes shared telemetry, resilience,
  retry, timeout, circuit-breaker, HTTP client, transport, authentication,
  serialization, runtime, configuration, diagnostics or other broadly consumed
  API.
- **Infrastructure:** fixes/materially improves CI/CD, build/release tooling,
  test infrastructure, developer tooling, package publishing, deployment,
  repository automation, shared environments or their reliability.
- **Mentioned:** attributable user identity explicitly requested as reviewer or
  mentioned in title, description, review request, linked discussion or comments.

Only **Recent** has a seven-day limit; older PRs can match other lenses. Recent
needs no thematic match. Foundational/infrastructure require changed-file or API
evidence; keywords alone or contrary paths/diffs cannot establish them. Record
every matching reason; never invent impact or downstream usage counts.

## Report

Use title **PR Review Radar**, count **new pull requests awaiting review**.
Assign each PR to its first matching group in this order: **Mentioned**,
**Foundational API**, **Infrastructure**, **Recent only**. Within groups sort
newest creation first, then canonical URL.

**Why review:** explain interest/consequence, not just category; include every
matching reason grounded in description, paths, API evidence or discussion.
Example: "Changes the shared HTTP retry policy and public error classification.
Opened two days ago; you were requested as a reviewer."

Empty-result wording: no new PRs awaiting review. Delivery entries use included
canonical URLs with `reportedAt`; report the number of PRs sent.
