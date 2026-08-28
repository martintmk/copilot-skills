# PR Evaluation Protocol

This document defines the second phase of the PR ingestion skill: evaluating
queued PR files with the AI model after the repository maintenance scan has
refreshed non-AI metadata in `<output_folder>/<pr_number>.json`.

## Trigger

Evaluate a PR only when its file is selected by the queue built in the main
skill:

- `scanned_at` is `null`, or
- `scanned_at` is older than seven days from the current time.

The queue is built by reading every `<output_folder>/*.json` file after the
maintenance scan has refreshed deterministic metadata. Only files whose
`repository` and `interest_query` match the current invocation are eligible;
a file written for a different query has already been cleared and re-queued by
the maintenance scan, so this check is a safety net, not the primary filter.

## Agent model

Use one detached general-purpose agent per PR. Before the agent runs, retrieve
the PR's evaluation-specific data from the official GitHub MCP server:

- `pull_request_read` with `method = get` for current PR metadata
- `pull_request_read` with `method = get_files` for the changed-file list and patch context
- `pull_request_read` with `method = get_diff` when a file patch is missing or insufficient
- `pull_request_read` with `method = get_reviews` only if review context is needed to interpret the change

The agent receives only:

- the PR's JSON file path and its current contents
- repository name and PR number
- title and URL
- current diff metadata and changed files for that single PR
- interest query

It should not carry the whole repository context, previous PRs, or unrelated
scan history.

## Required output

Return a single JSON object with exactly these keys:

```json
{
  "brief": "One plain-text description of what the changes do.",
  "is_interesting": true,
  "interest_reason": "One evidence-based sentence tying the changes to the query."
}
```

Rules:

- `brief`: 1-160 characters, plain text only, no markdown, no line breaks.
- `is_interesting`: `true` only when the actual code changes directly match the query.
- `interest_reason`: 1-500 characters, evidence-based, no invented facts.
- If the evidence is partial or ambiguous, return `false` and explain the limitation.

## Validation and file write

Before writing, re-read `<output_folder>/<pr_number>.json` and confirm it still
matches the PR this agent was given: same `repository`, `pr_number`, and
`interest_query`, and the same `source_updated_at` the agent was handed at the
start. If any of these differ, the underlying data changed mid-evaluation;
discard this result rather than writing it, so a newer maintenance-scan pass can
re-queue it.

If the file still matches, merge the AI fields into it and set `scanned_at` to
the current timestamp:

```json
{
  "repository": "owner/name",
  "pr_number": 123,
  "interest_query": "Changes to authentication or authorization",
  "title": "Rework session token validation",
  "url": "https://github.com/owner/name/pull/123",
  "author_login": "octocat",
  "status": "open",
  "review_count": 2,
  "source_updated_at": "2026-08-20T10:00:00Z",
  "brief": "Reworks session-token validation and rejects tokens with stale signing keys.",
  "is_interesting": true,
  "interest_reason": "The changed auth path directly alters session-token validation.",
  "scanned_at": "2026-08-28T12:00:00Z"
}
```

Write the whole file atomically where the host allows it (write to a temp file
in the same folder, then rename over the target) so a crash mid-write never
leaves partial or malformed JSON on disk. A file that has not yet been
AI-evaluated keeps `brief`, `is_interesting`, `interest_reason`, and
`scanned_at` as `null`.
