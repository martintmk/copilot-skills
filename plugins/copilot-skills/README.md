# copilot-skills

Personal [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/use-copilot-agents/use-copilot-cli)
skills, packaged as a cross-engine Agency-style plugin.

## Install

With Agency:

```text
agency plugin install market:copilot-skills@martintmk/copilot-skills
```

For a single Agency session without installing:

```text
agency copilot --plugin market:copilot-skills@martintmk/copilot-skills
```

With GitHub Copilot CLI directly:

```text
copilot plugin marketplace add martintmk/copilot-skills
copilot plugin install copilot-skills@martintmk-skills
```

Update later with:

```text
copilot plugin update copilot-skills@martintmk-skills
```

## Skills

Each skill's linked instructions define its scope and procedure; this table is
the entry-point guide.

| Skill | Use it for |
| --- | --- |
| [`review-lens`](skills/review-lens/SKILL.md) | Every sub-review on every Rust PR, branch, commit or working-tree review, with public API as the dominant lens. |
| [`review-api-design`](skills/review-api-design/SKILL.md) | Changed public contracts, construction, traits, semver coupling, and error/panic conventions. |
| [`review-correctness`](skills/review-correctness/SKILL.md) | Reproduced behavioral defects across every changed correctness-sensitive path. |
| [`review-tests`](skills/review-tests/SKILL.md) | Lost or weakened coverage, unjustified behavior changes, test style and supported test utilities. |
| [`review-resilience`](skills/review-resilience/SKILL.md) | Recovery classification and version-correct resilience middleware. |
| [`review-perf`](skills/review-perf/SKILL.md) | Measured cost, allocations, hot paths and injectable clocks. |
| [`review-naming`](skills/review-naming/SKILL.md) | Sibling naming conventions and unnecessary abstractions. |
| [`review-telemetry`](skills/review-telemetry/SKILL.md) | Signal contracts, OpenTelemetry conventions, cardinality and duplicate instrumentation. |
| [`review-consistency`](skills/review-consistency/SKILL.md) | Agreement between code, public docs, examples and related guides, including stale defaults or conflicting instructions. |
| [`review-public-api`](skills/review-public-api/SKILL.md) | An output-only `cargo public-api` audit with isolated docs-based filtering. |
| [`review-public-docs`](skills/review-public-docs/SKILL.md) | A scoped, authoritative public-docs bundle from rustdoc JSON; no review verdict. |
| [`review-delivery`](skills/review-delivery/SKILL.md) | Final review delivery to GitHub, ADO or chat; not another review pass. |
| [`pr-review-queue`](skills/pr-review-queue/SKILL.md) | Sequential reviews of requested, self-authored and overlooked PRs, then reviews of new commits until merge. |
| [`pr-review-radar`](skills/pr-review-radar/SKILL.md) | Newly discovered PRs worth reviewing, sent to Teams self-chat. |
| [`pr-feedback-radar`](skills/pr-feedback-radar/SKILL.md) | New unanswered human PR feedback, prioritizing demonstrably blocking requests. |
| [`feedback-autonomy`](skills/feedback-autonomy/SKILL.md) | Handles eligible automation and same-human PR-author instructions; finishes independent work before batching remaining approvals. |
| [`teams-self-message`](skills/teams-self-message/SKILL.md) | Delivery to the user's own Teams chat. |

## Review pipeline

Ask for "review this PR" or "review my changes" to use `review-lens`, or name a
focused skill to review only that area.

1. **Establish context once.** The coordinator resolves revisions, rules,
   execution trust, CI and existing discussion using
   [shared review context](skills/review-lens/review-context.md).
2. **Run and isolate every sub-review.** Review Lens dispatches all ten review
   and retrieval skills on every invocation, including small, docs-only and
   manifest-only changes. Each gets a fresh agent session. The coordinator sends
   a minimal factual handoff, not its conversation or other reviewers' reasoning.
   Independent work may run in parallel; dependent work stays sequential but
   never shares a reviewer context.
3. **Reuse evidence.** Matching excerpts, targeted results and scoped docs
   bundles are shared instead of repeating setup, builds or investigation.
4. **Gate completion, then deliver once.** Every required worker must return
   snapshot-matching coverage or an evidence-backed not-applicable result;
   missing/blocked work cannot be called complete. The coordinator merges
   duplicate root causes and dispatches one fresh `review-delivery` worker.

The [findings contract](skills/review-delivery/findings-contract.md) owns the
shared format. Every finding starts with AI attribution and a bold title,
followed by **Problem** (issue and evidence) and **Why this matters** (impact).
Actionable findings use a diagnosis title and end with **Suggested fix**
(correction). This applies to intermediate specialist results, filtered API
reports, standalone reports and PR threads, including non-blocking findings and
nits. Design notes still require **Problem**: use an observation title and
describe the constraint or trade-off with evidence, without asserting a defect.
Only **Suggested fix** is omitted when no change is requested. Clean summaries
and docs-only bundles do not need finding sections.

Specialists read that compact contract without loading provider posting
mechanics. The
[worker isolation protocol](skills/review-lens/worker-isolation.md) also covers
standalone requests, docs retrieval and API filtering. Workers do not
recursively redispatch themselves or reuse contexts across skills/passes. The
coordinator routes dependency and documentation checks to specialists rather
than doing inline review work.

The mandatory `review-public-api` pass stays output-only and report-only: it
receives no source-based findings, and its docs-based filtering stays isolated.
`review-public-docs` is also always dispatched and returns a scoped bundle to
docs consumers, never raw JSON or findings. Matching artifacts need not be
rebuilt. Directly invoking a focused skill still runs only that workflow.

## PR tracking and automation

The radars discover and notify; they do not review PRs or act on feedback.
Their Teams digests keep workflow-specific `Why review` / `Why respond` fields,
not the code-review finding format. `teams-self-message` owns message delivery;
each radar owns its eligibility and notification state.

`pr-review-queue` performs the reviews. It asks for monitored GitHub/ADO
repositories and a cadence, preflights MCP and provider-CLI capabilities, and
handles explicit requests oldest first. It also covers all published PRs by the target
author and otherwise-unreviewed PRs older than 24 hours but no older than seven
days. After a successful review, new commits stay eligible regardless of age.
GitHub requests are cleared only after verified delivery; ADO assignments and
votes are retained, with request-cycle completion tracked locally. It does not
reply to later discussion or apply fixes. Creating the skill does not start a
schedule. MCP gaps can be filled by provider CLIs; missing combined capability
blocks execution rather than falling back to direct HTTP. Preflight has a
bounded discovery budget and [focused CLI recipes](skills/pr-review-queue/provider-cli-recipes.md)
for identity, pagination and authoritative history. If one provider blocks
setup, explicitly requesting a narrower scope (for example GitHub only)
preserves the inactive configuration and the confirmed cadence; inactive
providers are not monitored. The queue's runner gates posting on Review Lens's
complete coverage manifest and keeps delivery/acknowledgment journals separate.

`feedback-autonomy` handles eligible automation and actionable instructions
posted by the same human who created the PR, using provider identity data without
a manual confirmation step. Other human or uncertain feedback remains gated.
It finishes independent addressable comments and build problems before batching
remaining approvals. Major changes and conflicts still need scoped direction,
which an explicit PR-author instruction can supply.

## Layout and maintenance

```text
.claude-plugin/marketplace.json       Claude marketplace entry
.github/plugin/marketplace.json       Copilot marketplace entry
plugins/copilot-skills/
  .claude-plugin/plugin.json          Shared plugin manifest
  agency.json                        Engine/platform metadata
  README.md                          Entry-point guide
  skills/<name>/SKILL.md              Trigger, scope and procedure
  skills/<name>/*.md                  Shared rules or on-demand reference
```

For a new skill, add YAML frontmatter with `name` and a precise `description`,
including when not to use it. Reuse shared rules rather than copying them;
keep domain-specific evidence and exceptions with their owning skill. Put
reference-only detail in a linked procedure when it would otherwise burden
every invocation. Add the skill to this guide.

When releasing, bump the plugin version in its manifest and both marketplace
entries together.
