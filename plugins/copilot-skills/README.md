# copilot-skills

Personal [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/use-copilot-agents/use-copilot-cli)
skills, packaged as a cross-engine Agency-style plugin. The Teams review monitor
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
| [`pr-review-teams-channel`](skills/pr-review-teams-channel/SKILL.md) | Every 15 minutes, finds clear PR review requests in a user-provided Teams channel and runs full Review Lens reviews in linked PR sessions. |
| [`pr-review-radar`](skills/pr-review-radar/SKILL.md) | Newly discovered PRs worth reviewing, sent to Teams self-chat. |
| [`pr-feedback-radar`](skills/pr-feedback-radar/SKILL.md) | New unanswered human PR feedback, prioritizing demonstrably blocking requests. |
| [`feedback-autonomy`](skills/feedback-autonomy/SKILL.md) | Handles eligible automation and same-human PR-author instructions; finishes independent work before batching remaining approvals. |
| [`minimal-local-validation`](skills/minimal-local-validation/SKILL.md) | Runs `cargo check` only for changed Rust crates, leaves comprehensive validation to CI, and continues queued work or prompts without waiting for checks. |
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
supplies context and commands, selects the review model and passes down the
session's permissions and execution constraints. A focused request needs that
setup for only its selected skill. Only the findings contract is shared.

`review-lens` adds the coordination:

1. **Pin the change once:** base and head, repository rules, CI and existing
   discussion.
2. **Inherit execution limits:** reviewers use the same permissions, tools and
   constraints as their parent, not separate approval checks or branch-based
   policies. Prepare missing tools or artifacts only when needed and permitted.
3. **Run nine areas in dedicated high-reasoning agents:** public contract,
   correctness, tests, performance, naming, telemetry, resilience, consistency
   and public API surface. Set high reasoning in the launch settings, honor the
   configured model, and never silently downgrade. `review-public-docs` is a
   helper that supplies rustdoc text.
4. **Resolve gaps and merge:** source review can finish without executing code;
   unproven runtime concerns remain questions. Public API review needs matching
   captures and docs for its claims, not necessarily a new local build. Resume
   a reviewer once after fixing a concrete setup problem.
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
The coordinator supplies the captures or the commands to produce them; the
skill contains the review rules, not tool installation or checkout management.

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

Run each auto-approve monitor in its own session for one PR, separate from the
Teams review coordinator and its child sessions. Each monitor keeps its own
per-session state and trigger. The Teams review coordinator keeps its own outcomes and
automation in its dedicated coordinator session. Neither one clears the
other's schedule, writes its state or treats the other's reviews as completed work.
The `human-review-required` label is a GitHub handoff signal, not shared queue
state.

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
authorship (including drafts), watched follow-ups, and published PRs over 1 hour
and at most seven days old with at most one distinct human reviewer. Copilot and
other verified bot/app reviews do not count; repeated submissions by the same
human count once. All other authors' PRs must be non-draft and no older than seven
days, including requests and watched changes; target-authored PRs remain watched
until closure or merge while labeled or individually requested.
Losing both the label and individual request pauses review and publication, not
history; re-adding the label alone does not repeat a completed review. The policy
distinguishes new heads/requests from handled work and incomplete coverage, and
keeps retired PRs retired after reopening.

[`pr-review-teams-channel`](skills/pr-review-teams-channel/SKILL.md) is the thin
Copilot app coordinator for review requests posted in Teams. The user supplies
the channel link; the skill never hard-codes it, and it infers the repository
from the project. Native same-session automation checks the channel every 15
minutes through WorkIQ, starting from the current position without scanning
history. It reports gaps instead of claiming complete monitoring.

For each unambiguous request for an open PR in the project repository, it links
a PR session, posts one preparation reply in the thread, and has the session
publish a fresh Review Lens review to GitHub. New heads, base retargeting and
new explicit requests are reviewed again. It never edits code, pushes, merges or
changes labels or reviewers. It keeps only the state needed to avoid duplicate
reviews and deletes its own PR sessions once GitHub shows the PR merged or
closed. Installing/editing starts nothing.

This replaces `pr-review-queue-github`; there is no label-based queue in this
version.
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
