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
| [`minimal-local-validation`](skills/minimal-local-validation/SKILL.md) | Runs `cargo check` only for changed Rust crates and leaves comprehensive workspace validation to CI. |
| [`teams-self-message`](skills/teams-self-message/SKILL.md) | Delivery to the user's own Teams chat. |

## Review pipeline

Ask for "review this PR" or "review my changes" to use `review-lens`, or name a
focused skill to review only that area.

1. **Establish context once:** pin revisions/configuration, trusted rules,
   execution permission, CI and discussion using
   [shared context](skills/review-lens/review-context.md).
2. **Dispatch all ten specialists:** every invocation, even docs-only, uses
   [fresh workers](skills/review-lens/worker-isolation.md) with minimal factual
   handoffs. Reuse matching evidence, not reviewer conversations or reasoning.
   Dependent stages remain sequential and isolated.
3. **Complete and deliver:** require matching coverage or evidence-backed
   not-applicability from every worker, then one fresh delivery worker.
   Missing/blocked work is not completion. A later descendant-head refresh
   checks existing findings only and forces comment-only delivery, not a claim
   of full new-head coverage.

Complete current-head reviews automatically approve when clean or containing
only nits, retaining any nit comments (GitHub approval; ADO approved or approved
with suggestions). No extra confirmation is needed, including in the review
queue. Unresolved findings still count even when duplicate comments are omitted.
Report-only reviews never post; requester-owned/poster-owned PRs and descendant
finding refreshes remain comment-only/no vote. Incomplete coverage cannot approve.

The [findings contract](skills/review-delivery/findings-contract.md) owns the
format at every handoff: AI attribution, bold title, **Problem** (evidence),
**Why this matters** (impact), and **Suggested fix** for actionable findings.
Design notes use an observation title and omit only the fix. Clean summaries
and docs bundles are not findings; specialists need no provider-posting guide.

`review-public-api` remains output-only/report-only with mandatory isolated docs
filtering. `review-public-docs` returns scoped rustdoc bundles, not raw JSON,
findings or verdicts. Proven added/removed crates use the
[package-presence contract](skills/review-lens/package-comparison.md): a logical
empty side and complete real artifacts for the existing side. Failed extraction
for an existing crate is a blocker, never proof of absence.

## PR tracking and automation

The radars discover and notify, not review or act. Each owns its eligibility
and notification history; `teams-self-message` owns delivery. Digests use
`Why review` / `Why respond`, not the code-review finding format.

`pr-review-queue` keeps [one folder per PR](skills/pr-review-queue/state-machine.md)
with observations, pending work, reviews and recovery evidence. Its finite loop
fetches candidates, compares the cache, then runs full reviews sequentially.
Explicit requests are oldest-first; initial eligibility also includes all
published target-authored PRs and otherwise-unreviewed PRs over 24 hours and at
most seven days old. Completed PRs stay watched for new heads until merge.

Setup requires explicit repositories/cadence and bounded MCP/official-CLI
preflight; missing combined capability blocks rather than enabling raw HTTP.
Installing the skill starts nothing. Explicit scope narrowing retains inactive
history/cadence without polling those repositories. Existing version-1 state
migrates without discarding receipts, requests or unfinished work.

Only verified complete review delivery permits request acknowledgment: GitHub
clears the processed generation only; ADO retains assignments/votes and records
completion locally. Ambiguous writes block recovery, while proven zero-write
PR-local failures can be quarantined without blocking later candidates.
The queue does not reply to discussion or apply fixes.

`feedback-autonomy` independently gates actions by authorship and impact.
Eligible automation and stable-identity-matched PR-author instructions need no
manual identity confirmation. Other human/uncertain feedback, major changes
and conflicts need scoped authorization. Finish independent addressable work
before batching remaining approvals.

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

New skills need `name`/precise trigger and exclusion `description` frontmatter
and a row above. Give each rule one owner; link shared contracts and load
reference detail only at its relevant step. Keep specialist evidence/exception
requirements, and measure total text including references, not entry points alone.

When releasing, bump the plugin version in its manifest and both marketplace
entries together.
