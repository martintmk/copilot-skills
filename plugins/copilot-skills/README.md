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
| [`engineering-workplan`](skills/engineering-workplan/SKILL.md) | Outcome-driven plans, bounded agent instructions and evidence-based steering for large engineering work, rewrites and migrations. |
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
| [`pr-auto-approve`](skills/pr-auto-approve/SKILL.md) | Monitor one GitHub PR until merged; fast-track mechanical changes or proven small pipeline fixes, with compatible APIs, preserved coverage and revocable automated approval. |
| [`pr-review-queue`](skills/pr-review-queue/SKILL.md) | Sequential reviews of requested, self-authored and overlooked PRs, with age-bounded monitoring except for target-authored PRs. |
| [`pr-review-radar`](skills/pr-review-radar/SKILL.md) | Newly discovered PRs worth reviewing, sent to Teams self-chat. |
| [`pr-feedback-radar`](skills/pr-feedback-radar/SKILL.md) | New unanswered human PR feedback, prioritizing demonstrably blocking requests. |
| [`feedback-autonomy`](skills/feedback-autonomy/SKILL.md) | Handles eligible automation and same-human PR-author instructions; finishes independent work before batching remaining approvals. |
| [`minimal-local-validation`](skills/minimal-local-validation/SKILL.md) | Runs `cargo check` only for changed Rust crates and leaves comprehensive workspace validation to CI. |
| [`teams-self-message`](skills/teams-self-message/SKILL.md) | Delivery to the user's own Teams chat. |

## Large engineering work

`engineering-workplan` turns an objective into a measurable destination,
compatibility requirements, dependency-ordered slices, coordinator/worker prompts,
and an evidence ledger. It can also use progress reports and repository evidence
to produce bounded corrective instructions without silently changing the goal.
It prepares instructions; it does not start agents or implement the rewrite.

```text
Use engineering-workplan to prepare instructions for replacing our legacy
storage engine. Inspect the repository, preserve the public API and persisted
data compatibility, keep releases shippable, and identify decisions I must make.
Produce the workplan and prompts for the first ready slices. Do not implement it.

Use engineering-workplan to steer the migration from the existing workplan and
these progress reports. Identify unproven requirements and write the next bounded
instructions without weakening the acceptance criteria.
```

The skill adapts lessons from the
[Copilot runtime rewrite](https://github.blog/ai-and-ml/generative-ai/migrating-the-github-copilot-runtime-to-rust-using-copilot/):
protect independent test oracles, separate faithful translation from optional
redesign, give shared boundaries one owner, budget expensive checks, and verify
the complete production outcome rather than code volume or compilation alone.

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
3. **Complete and deliver:** require a matching coverage record from every
   worker, then one fresh delivery worker. Missing work prevents publication.
   Blocked areas make the review incomplete but do not suppress findings from
   completed areas: delivery posts a COMMENT with a prominent blocked-area
   warning and no verdict or vote. At least one area must complete. A later
   descendant-head refresh checks existing findings only after a complete review
   and forces comment-only delivery, not a claim of full new-head coverage.

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

`pr-auto-approve` speeds approval of mechanical PRs and minimal, evidence-backed
fixes for concrete pipeline failures. Small backward-compatible API additions
are allowed; breaking changes and weakened integration/E2E coverage are not.
Approval is a real GitHub APPROVE review with a short automation-attributed
message, not a full Review Lens review.

It monitors one PR until merged, rechecking new commits and base changes. If
later changes violate the rules, it dismisses its own approval, applies
`human-approval-required`, and updates one short status comment across revisions.
Ordinary pending checks wait quietly without adding a human-review label.
There are no inline findings or comment-heavy reviews. Monitoring continues
after approval or escalation with no age cutoff; unmerged closures pause PR
writes until reopening. Setup confirms a cadence and verifies a durable trigger.
Installing or editing the skill starts nothing; it never merges.

```text
Use pr-auto-approve to monitor https://github.com/OWNER/REPO/pull/NUMBER
until merged, checking every 10 minutes.
```

The radars discover and notify, not review or act. Each owns its eligibility
and notification history; `teams-self-message` owns delivery. Digests use
`Why review` / `Why respond`, not the code-review finding format.

`pr-review-queue` keeps [one folder per PR](skills/pr-review-queue/state-machine.md)
with observations, pending work, reviews and recovery evidence. Its finite loop
fetches candidates, compares the cache, then runs full reviews sequentially.
Explicit requests are oldest-first; initial eligibility also includes all
target-authored PRs, including drafts, and otherwise-unreviewed published PRs
over 24 hours and at most seven days old. Requests and watched changes on other
authors' PRs also stop after seven days. Target-authored PRs stay watched until
merge; closed or abandoned PRs are retired and do not resume if reopened.

Setup requires explicit repositories/cadence and bounded MCP/official-CLI
preflight; missing combined capability blocks rather than enabling raw HTTP.
Installing the skill starts nothing. Explicit scope narrowing retains inactive
history/cadence without polling those repositories. Existing version-1 state
migrates without discarding receipts, requests or unfinished work.

Only verified review delivery permits request acknowledgment, including a
verified incomplete COMMENT carrying the required coverage warning: GitHub
clears the processed generation only; ADO retains assignments/votes and records
delivery locally. Incomplete delivery records suppressed coverage debt rather
than advancing the verified baseline, avoiding duplicate same-snapshot comments
while allowing retry after the blocker changes. Ambiguous writes block recovery,
while proven zero-write PR-local failures can be quarantined without blocking
later candidates.
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
