# copilot-skills

Personal [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/use-copilot-agents/use-copilot-cli)
skills, packaged as a cross-engine Agency-style plugin. The GitHub review queue
uses the GitHub Copilot app's native session capabilities.

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
| [`review-lens`](skills/review-lens/SKILL.md) | Full Rust PR, branch, commit or working-tree reviews that run every review area, with public API as the dominant lens. |
| [`review-api-design`](skills/review-api-design/SKILL.md) | Changed public contracts, construction, traits, semver coupling, and error/panic conventions. |
| [`review-correctness`](skills/review-correctness/SKILL.md) | Reproduced behavioral defects across every changed correctness-sensitive path. |
| [`review-tests`](skills/review-tests/SKILL.md) | Lost or weakened coverage, unjustified behavior changes, test style and supported test utilities. |
| [`review-resilience`](skills/review-resilience/SKILL.md) | Recovery classification and version-correct resilience middleware. |
| [`review-perf`](skills/review-perf/SKILL.md) | Measured cost, allocations, hot paths and injectable clocks. |
| [`review-naming`](skills/review-naming/SKILL.md) | Sibling naming conventions and unnecessary abstractions. |
| [`review-telemetry`](skills/review-telemetry/SKILL.md) | Signal contracts, OpenTelemetry conventions, cardinality and duplicate instrumentation. |
| [`review-consistency`](skills/review-consistency/SKILL.md) | Agreement between code, public docs, examples and related guides, including stale defaults or conflicting instructions. |
| [`review-public-api`](skills/review-public-api/SKILL.md) | An output-only `cargo public-api` audit, with claims checked against rustdoc. |
| [`review-public-docs`](skills/review-public-docs/SKILL.md) | A helper that returns scoped public docs from rustdoc JSON; no findings or verdict. |
| [`review-delivery`](skills/review-delivery/SKILL.md) | Final review delivery to GitHub, ADO or chat; not another review pass. |
| [`pr-auto-approve`](skills/pr-auto-approve/SKILL.md) | Monitor one GitHub PR until merged or handed off for human review; fast-track mechanical changes or proven small pipeline fixes, with compatible APIs, preserved coverage and revocable automated approval. |
| [`pr-review-eligibility`](skills/pr-review-eligibility/SKILL.md) | The small, side-effect-free decision: should this PR be automatically reviewed now? |
| [`pr-review-queue-github`](skills/pr-review-queue-github/SKILL.md) | Sequential GitHub reviews of labeled or individually requested PRs, using native Copilot app automation and linked PR sessions. |
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
focused skill to review only that area. Area skills contain only review rules:
what to look for, what evidence counts and how to report. The coordinator
supplies the change, context and execution constraints. Only the findings
contract is shared.

`review-lens` adds the coordination:

1. **Pin the change once:** base and head, repository rules, CI and existing
   discussion.
2. **Set execution constraints:** local work and PRs from branches in the
   target repository may run; fork PRs are reviewed by reading. When code may
   run, install the pinned toolchain and fetch dependencies once.
3. **Run nine areas in dedicated high-reasoning agents** that inherit the
   coordinator's permissions and execution constraints: public contract, correctness, tests,
   performance, naming, telemetry, resilience, consistency and public API
   surface. `review-public-docs` is a helper that supplies rustdoc text.
4. **Retry once and merge:** areas that read source finish even when they
   cannot run code; unproven runtime concerns become questions. Only the public
   API surface needs a successful build. Fixable failures get one retry.
5. **Deliver once** through `review-delivery`, with a plain-language summary.

| Outcome | Delivery |
| --- | --- |
| Complete | Approves when clean or nit-only; otherwise approves with comments or requests changes. |
| Incomplete | Comment with a warning that lists unreviewed topics; no verdict or vote. |
| Refreshed after new commits | Comment rechecking existing findings only. |
| Requester's or poster's own PR | Comment, no verdict. |
| Report only | Nothing posted. |

The [findings contract](skills/review-delivery/findings-contract.md) owns the
format at every handoff: AI attribution, bold title, **Problem** (evidence),
**Why this matters** (impact), and **Suggested fix** for actionable findings.
Design notes use an observation title and omit only the fix.

`review-public-api` drafts only from `cargo public-api` output, then reads the
rustdoc of criticized items to remove or narrow claims, never to add them.

## PR tracking and automation

`pr-auto-approve` speeds approval of mechanical PRs and minimal, evidence-backed
fixes for concrete pipeline failures. Small backward-compatible API additions
are allowed; breaking changes and weakened integration/E2E coverage are not.
Approval is a real GitHub APPROVE review with a short automation-attributed
message, not a full Review Lens review.

It monitors one PR until merged or labeled `human-review-required`, rechecking
new commits and base changes. On escalation, it dismisses its own approval,
applies the label and posts or updates one short PR comment explaining the
specific reason human review is required and the action needed. It then cancels
its owned monitoring trigger. Already-labeled PRs are not monitored, regardless
of who applied the label; removing it does not automatically resume monitoring.
Ordinary pending checks wait quietly without adding a human-review label.
Fast approval requires at least one reported check and every check to complete
successfully, not just required checks; absent, failed, skipped, neutral or
unverified checks also wait without escalation solely for their check result.
There are no inline findings or comment-heavy reviews. Monitoring continues
after approval with no age cutoff; unmerged closures without the human-review
label pause PR writes until reopening. Setup confirms a cadence and verifies a
durable trigger.
Installing or editing the skill starts nothing; it never merges.

```text
Use pr-auto-approve to monitor https://github.com/OWNER/REPO/pull/NUMBER
until merged or labeled human-review-required, checking every 10 minutes.
```

The radars discover and notify, not review or act. Each owns its eligibility
and notification history; `teams-self-message` owns delivery. Digests use
`Why review` / `Why respond`, not the code-review finding format.

[`pr-review-eligibility`](skills/pr-review-eligibility/SKILL.md) owns the decision
without tools or side effects. Every queued review requires the exact label
`human-review-required` **or a current individual review request for the target**.
An explicit request bypasses the label, not the age or draft limits; team requests,
assignments and mentions do not qualify. Other eligible reasons are target
authorship (including drafts), watched follow-ups, and published PRs over 24 hours
and at most seven days old with at most one distinct human reviewer. Copilot and
other verified bot/app reviews do not count; repeated submissions by the same
human count once. All other authors' PRs must be non-draft and no older than seven
days, including requests and watched changes; target-authored PRs remain watched
until closure or merge while labeled or individually requested.
Losing both the label and individual request pauses review and publication, not
history; re-adding the label alone does not repeat a completed review. The policy
distinguishes new heads/requests from handled work and incomplete coverage, and
keeps retired PRs retired after reopening.

[`pr-review-queue-github`](skills/pr-review-queue-github/SKILL.md) is the thin
Copilot app coordinator. It fetches GitHub facts with `gh`, invokes eligibility,
and runs Review Lens sequentially in linked PR sessions. Native same-session
automation supplies recurrence; app history, status and child notifications
supply progress and recovery. There is no separate queue cache, lock, scheduler,
provider preflight framework or two-stage review runner.

Setup requires explicit GitHub repositories and, for recurrence, a confirmed
cadence. Installing/editing starts nothing. A one-shot run leaves schedules alone.
Review Lens still owns full reviews and verified publication; native pending
review drafts do not count as submitted reviews. The queue never manually removes
reviewers, replies to discussion or applies fixes. Changed inputs are deferred,
incomplete coverage is not completion, and uncertain writes are reconciled before
retrying.

This replaces `pr-review-queue`; there is no ADO queue in this version. Before
switching an existing monitor, stop its old schedule and settle in-flight work;
retain its audit history and carry verified outcomes into the coordinator session.
Other skills' ADO support is unchanged.

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
