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
| [`crate-change-coordinator`](skills/crate-change-coordinator/SKILL.md) | Applies one instruction to an ordered set of Rust crates through sequential child sessions and pull requests, with inherited model configuration, green CI, Teams review announcements, and merge-gated progress. |
| [`review-lens`](skills/review-lens/SKILL.md) | Lightweight approval for proven mechanical PRs with no API changes, or evidence-led full reviews across eleven areas with compact reports and separate benchmarks. |
| [`review-api-design`](skills/review-api-design/SKILL.md) | Changed public contracts, construction, traits, semver coupling, and error/panic conventions. |
| [`review-correctness`](skills/review-correctness/SKILL.md) | Behavioral defects proved by focused reproductions or complete source traces, including sync/async deadlocks. |
| [`review-tests`](skills/review-tests/SKILL.md) | Test-first review of lost coverage, behavior changes even without test edits, test style and supported test utilities. |
| [`review-resilience`](skills/review-resilience/SKILL.md) | Recovery classification and version-correct resilience middleware. |
| [`review-perf`](skills/review-perf/SKILL.md) | Measured cost, allocations, hot paths and injectable clocks. |
| [`metabench-compare`](skills/metabench-compare/SKILL.md) | Runs existing Gungraun benchmarks before and after a change for up to three changed, metabench-enabled crates. Shows metabench results and before/after instruction and allocation counts. Keeps full metrics in artifacts. |
| [`review-naming`](skills/review-naming/SKILL.md) | Sibling naming conventions and unnecessary abstractions. |
| [`review-telemetry`](skills/review-telemetry/SKILL.md) | Signal contracts, OpenTelemetry conventions, cardinality and duplicate instrumentation. |
| [`review-consistency`](skills/review-consistency/SKILL.md) | Agreement between code, public docs, examples and related guides, including stale defaults or conflicting instructions. |
| [`review-public-api`](skills/review-public-api/SKILL.md) | An output-only `cargo public-api` audit, with claims checked against rustdoc. |
| [`review-public-api-changes`](skills/review-public-api-changes/SKILL.md) | The always-present breaking/additive/no-change public API report, with concise Rust signatures and a human-review signal for any API change. |
| [`review-integration-tests`](skills/review-integration-tests/SKILL.md) | The always-present package `tests/` report, including shared inputs, target discovery and a human-review signal for breaking behavioral changes. |
| [`review-public-docs`](skills/review-public-docs/SKILL.md) | A helper that returns scoped public docs from rustdoc JSON; no findings or verdict. |
| [`review-delivery`](skills/review-delivery/SKILL.md) | GitHub, ADO or local delivery for full reviews, mechanical approvals and benchmark follow-ups, with current inline anchors and partial-write recovery. |
| [`pr-auto-approve`](skills/pr-auto-approve/SKILL.md) | Monitor one GitHub PR until merged or handed off for human review; fast-track mechanical changes or proven small pipeline fixes, with compatible APIs, preserved coverage and revocable automated approval. |
| [`pr-review-eligibility`](skills/pr-review-eligibility/SKILL.md) | The small, side-effect-free decision: should this PR be automatically reviewed now? |
| [`pr-review-teams-channel`](skills/pr-review-teams-channel/SKILL.md) | GitHub and ADO reviews from Teams requests, with one child per PR, inline findings, one rolling report, 15-minute polling and a four-hour default cooldown. |
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

Use `review-lens` for full reviews, or name one area for a focused review.
For a PR without an explicit full-review request, it first tries
[lightweight mechanical approval](skills/review-lens/mechanical-approval.md).
This covers behavior-preserving docs and private edits, additive tests, proven regression repairs and narrow pipeline repairs.
Every changed path must qualify. No public API change is allowed, including additions.

Lightweight approval requires complete API/test evidence, preserved coverage and every current check to pass.
Absent, pending, failed or unknown checks withhold approval without starting a monitor.
Other changes or incomplete classification evidence continue through the full review with reusable evidence.
Skipped specialist areas and benchmarks are not claimed as reviewed.
Explicit full-review and benchmark requests retain their scope. The Teams coordinator continues to request full reviews.
This one-time route does not change `pr-auto-approve`, which owns its separate GitHub monitoring lifecycle.

Area skills own specialist evidence rules. The coordinator supplies revisions,
shared artifacts and inherited execution limits.
The [findings contract](skills/review-delivery/findings-contract.md) owns format,
severity and summaries. `review-delivery` owns posting for both review modes.

The [full Review Lens procedure](skills/review-lens/SKILL.md) accounts for all eleven
areas without requiring eleven agents. It keeps connected source changes with
one owner and delegates substantial independent work.
Shared discovery and matching artifacts avoid repeated setup.
Unknown relevance remains an open question, not an exclusion.
Initial routing uses the complete path inventory. Each owner then reads its tests and full source diff without duplicated coordinator reads.

The coordinator uses [bounded handoffs](skills/review-lens/coordinator-reference.md#bound-handoffs-and-follow-ups)
with access to complete evidence. Ready review results and required follow-ups take priority over optional preparation.
It assembles the report as results arrive, while raw evidence and path inventories stay in artifacts.
Each test reviewer follows the
[test-first order](skills/review-lens/coordinator-reference.md#combine-test-review).
Both output-only API areas can share one source-free reviewer through the
[phased API handoff](skills/review-lens/coordinator-reference.md#protect-api-contexts).
The benchmark helper owns [measurement isolation](skills/metabench-compare/SKILL.md#2-prepare-and-run-the-comparison).
Pending benchmarks do not delay a ready review.
The review records their state, and [separate delivery](skills/review-delivery/SKILL.md#benchmark-follow-up) carries the later measurements.

The owning session invokes Review Delivery directly from the saved results.
Delivery checks current provider state and reads back writes.
Its [recovery procedure](skills/review-delivery/SKILL.md#ownership-and-recovery)
retries missing writes without repeating completed analysis.

Every full or focused review uses `simple-english` for its user-facing text.
Fast-path approval and escalation text uses it too.

| Outcome | Delivery |
| --- | --- |
| Eligible mechanical PR | Lightweight approval with evidence, unchanged API, preserved tests and successful current checks. No full-review claim or benchmark report. |
| Complete | Approves when clean or non-blocking. Only serious findings can request changes, and never after two distinct approvals. |
| Incomplete | Comment with a warning that lists unreviewed topics; no verdict or vote. |
| Refreshed after new commits | Recheck existing findings and keep them inline on current code. Comment-only, with full-coverage and refresh revisions separated. |
| Requester's or poster's own PR | Comment, no verdict. |
| Report only | Nothing posted. |

Each finding has AI attribution, a concrete title, **Problem**, **Why this matters**
and **Suggested fix**. Design notes omit only the fix.
The summary has one attribution and links inline threads instead of repeating
their bodies. Preparation logs, hashes and full file inventories stay in artifacts.
When resolution is permitted, delivery closes its own informational summary and design-note threads after reading their full discussion.
It keeps comments visible and leaves unresolved defects, nits, questions and requested decisions open.
GitHub top-level comments and review bodies have no resolved state.
The [thread procedure](skills/review-delivery/SKILL.md#close-owned-non-actionable-threads) defines ownership, provider status and recovery rules.

Every full Review Lens summary also includes three reports. The public API report
states **Breaking Changes**, **Public API Additions** or **No API Changes** and
groups additions by public family with concise callable signatures.
The [compact API report](skills/review-public-api-changes/SKILL.md#report) preserves every distinct break without copying whole declarations or repeated impl lists.
The integration-test report covers package `tests/` targets and their shared inputs.
It reports breaking, additive-only, behavior-preserving or no changes.
Missing inventory cannot establish no changes.
An omitted generic default in API output does not prove a breaking change.
The [renderer limits](skills/review-public-api/SKILL.md#renderer-limits) define that evidence boundary.
Review delivery adds `human-review-required` for any
public API change or a breaking integration-test behavior, independently of the
review verdict.

The benchmark report shows metabench results and before/after instruction and allocation counts for every selected Gungraun case.
It includes allocated bytes when measured, unavailable measurements and comparison limits.
[Metabench Compare](skills/metabench-compare/SKILL.md) defines crate selection and execution.
Its [report contract](skills/metabench-compare/run-and-report.md#side-by-side-results) defines the compact tables and saved artifacts.
Benchmark results do not independently require the human-review label.

```text
Use metabench-compare to compare this branch with its merge base.
Run the existing Gungraun targets for the three most-changed eligible crates.
Show metabench output and each benchmark's instruction and allocation counts before and after the change, with absolute and percentage changes.
```

## PR tracking and automation

`crate-change-coordinator` applies one bounded instruction to several Rust
crates. It creates one child session and one pull request for each crate. The
child inherits the coordinator's model configuration.

The coordinator waits for every check on the current pull request head to pass.
It then announces the pull request in the supplied Teams review channel. The
next child starts only after GitHub confirms the merge and the coordinator
archives the current child.

The child writes the Summary, Evidence, and Merge Danger sections with
`simple-english`. The coordinator monitors CI and feedback until merge and sends
new failures or feedback to the same child. The child applies `feedback-autonomy`
before acting on feedback. No separate child watcher is required.

```text
Use crate-change-coordinator with these crates:
recoverable, seatbelt, testing_aids

Review channel:
https://teams.microsoft.com/l/channel/...

Instruction:
Replace the deprecated tracing helper with the workspace helper.
```

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

Review Lens can also apply this handoff label when its mandatory reports find a
public API change or a breaking integration-test behavior. Auto-approve monitors
observe the label and stop through their normal terminal path.

```text
Use pr-auto-approve to monitor https://github.com/OWNER/REPO/pull/NUMBER
until merged or labeled human-review-required, checking every 10 minutes.
```

The radars discover and notify, not review or act. Each owns its eligibility
and notification history; `teams-self-message` owns delivery. Digests use
`Why review` / `Why respond`, not the code-review finding format.

[`pr-review-eligibility`](skills/pr-review-eligibility/SKILL.md) is a separate,
side-effect-free policy for label/request-based queues.
It owns their age, draft, reviewer-count and retry rules.
It does not gate the Teams channel monitor, which uses explicit channel requests.

[`pr-review-teams-channel`](skills/pr-review-teams-channel/SKILL.md) watches a
user-provided channel for the project's GitHub or ADO repository.
One same-session automation polls every 15 minutes from a saved coverage boundary.
New replies to old roots count. A recent page alone does not prove complete coverage.

Each PR has one owned child and one rolling report. The child owns PR writes.
New heads, retargeting, changed merge bases and explicit requests make work due.
Target-tip advancement alone does not repeat an unchanged review.
The four-hour default cooldown coalesces updates. Finished analysis survives
posting failures, and unchanged blockers do not trigger another review each tick.

Findings remain attached to current code blocks after commits.
Already tracked inline findings are reused. Summary-only findings move inline
when a valid anchor exists. Stale threads get a linked replacement only when
provider tracking no longer locates the affected block.

The monitor never edits code, pushes, merges, changes labels or adds reviewers.
Required label signals remain visible as unapplied.
Closed PRs retire from monitoring, and eligible finished children are archived,
not deleted. Editing skills does not start or reconfigure live monitors.

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
