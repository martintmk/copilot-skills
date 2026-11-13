# Review Lens coordinator reference

The coordinator owns setup, routing and recovery.
Read the sections needed for the current step.
Give reviewers the required facts, not this entire file.

## Evidence-led routing

Route work by its required evidence, not by area names.
Use the complete inventory from [Review Lens](SKILL.md#1-pin-facts-and-prepare-shared-evidence).
An evidence bundle contains related paths, contracts and unresolved questions.
Routing assigns work. It does not complete the area's review procedure.
Initial routing can remain provisional until each source owner finishes its test-first pass and complete diff.
The coordinator does not repeat delegated source reads. It reconciles returned evidence and path coverage instead.

### Record coverage

Keep one internal review record with an entry for each of the eleven areas.
Use artifact references and short reasons rather than copied source or transcripts.
An input digest identifies reviewed local file contents.
Record these fields:

| Field | Required content |
| --- | --- |
| Applicability | Relevant (confirmed connection), absent with evidence, or unknown (unresolved connection) |
| Scope | Changed paths, affected packages and the connected contracts or consumers |
| Evidence | Decisive locations or artifact references, exact revisions or local input digest, and selected configuration |
| Owner | Coordinator or assigned reviewer, including ownership of any separate scopes |
| Open questions | Unresolved relevance, required traces, missing evidence and setup blockers |
| Completion | Pending until the area returns its result, then its status and material limits |

Assign every changed path to the relevant area scopes.
Keep connected traces together. Do not sample paths or stop after the first finding.
When evidence reveals another affected path or contract, update its scope and owner.

Before completion, reconcile each area's returned coverage with this record.
Use the [area result format](../review-delivery/findings-contract.md#area-result-and-final-summary) and these completion rules:

| Work | Required evidence for completion |
| --- | --- |
| Source-reading areas | Every changed path in scope traced, with material evidence limits stated |
| API design audit | Matching captures and the required docs for draft claims, unless no library is affected |
| API classification | Matching captures, unless no library is affected |
| Integration-test classification | Complete inventory and the contents required by the owner |

Use [package-presence evidence](#package-presence) to prove that no library is affected.
Missing execution proof limits findings, not completed source coverage.
Missing source or unfinished traces remain unreviewed, even when no defects were found.
Use `could not review` for required work that cannot finish.

### Establish applicability

Start from the complete changed-path inventory at the pinned revisions.
For removals and renames, inspect both old and new ownership, references and discovery paths.
Do not treat a missing head file or a path-only rename as proof that no contract changed.
Use filenames and keyword searches to locate evidence or assign provisional scope, never to prove absence.

Each source owner follows connections from changed definitions, values and shared inputs.
Use these connections to find indirect effects, not as another area checklist:

| Changed input | Connection to inspect |
| --- | --- |
| Macros or generated code | Macro and derive consumers, expansion inputs, generated items and callers |
| Types, traits or generic code | Re-exports, aliases, bounds, impls and downstream type or dispatch relationships |
| Features, build inputs or dependencies | Resolved versions, enabled features, target conditions, build scripts and included files that select or generate code |
| Values, errors or control flow | Data reaching unchanged signal emitters, error conversions, recovery classification or middleware |
| Shared fixtures, helpers, snapshots or configuration | Owning test targets and their consumers, including shared inputs outside package `tests/` directories |

Add a consumer only through a specific changed input or contract connection.
Stop following a connection when evidence shows that the downstream contract is unaffected in the selected configuration.
Record that boundary or the unresolved question.
Do not audit unrelated dependencies or workspace packages.

Apply these routing decisions separately from completion:

| Applicability | Next action |
| --- | --- |
| Relevant | Assign the confirmed connection and remaining traces to an owner. |
| Unknown | Resolve the named question with the smallest relevant read or matching artifact. If unresolved, retain the affected coverage gap. |
| Absent with evidence | Cite the inspected changes, connected boundaries and configuration that rule out relevance at both revisions. Use `not applicable` only under the area's rules. |

Relevant does not mean defective. A relevant area with no findings returns `done`, not `not applicable`.
Missing evidence cannot prove absence. This includes truncated diffs, inaccessible source, incomplete inventories and missing required captures.
Make no claims about unreviewed configurations.

Do not exclude test review merely because no test changed.
Follow [Review Tests](../review-tests/SKILL.md).
Area skills own proof rules and classifications. Routing cannot replace them.

### Choose review owners

1. Review small connected source changes in the coordinator.
   Apply each relevant [area skill](SKILL.md#review-areas).
2. Keep a single continuous trace with one owner.
   Do not split it merely to create parallel work or separate area labels.
3. Delegate substantial independent bundles when their evidence needs separate context.
   Compatible source areas can share a reviewer when they need the same paths and context.
   Keep a distinct result for each area.
4. If new evidence reveals independent complexity or an evidence boundary, split only the affected unfinished scope.
   Name its connection, remaining question and owner.
   If a continuous trace needs different expertise, transfer unfinished work rather than adding a second owner.
   Do not add reviewers to seek agreement or repeat completed work.
5. Select reviewer count from evidence scope, required isolation and inherited runtime limits.
   Bound context by relevant evidence, not arbitrary agent or token caps.
   If a required specialist is unavailable, retain supported work and report the uncovered scope.

Honor the user's model/effort configuration and tool rules.
This skill does not authorize model overrides, nested fleets or new permissions.
Fewer reviewers do not lower proof requirements or establish a measured speedup.

### Combine test review

Prefer one suitable owner for `review-tests` and `review-integration-tests` when they share evidence.
A one-shot reviewer cannot accept follow-up messages.

1. Supply both skills and the [integration-test evidence](#integration-test-evidence).
   Include the baseline tests and changed expectations required by [Review Tests, step 1](../review-tests/SKILL.md#procedure).
2. Enforce that step for each reviewer, regardless of the coordinator's earlier reading.
   Withhold implementation code and explanations until the reviewer records baseline expectations and open questions.
   This includes PR bodies, review bodies, discussion and other reviewers' conclusions.
   Include inline tests and indented or fenced doctests. A source hunk can mix test and implementation text.
   For one-shot reviewers, provide artifact paths and the required reading order rather than an implementation summary.
   If inherited context prevents this order, use a compliant context.
   If none is available, mark affected coverage `could not review`.
3. Return separate scopes and statuses, plus the integration report and its human-review decision.

Unlike output-only API review, test review continues into implementation after recording the baseline.
Keep the same reviewer for that work.

When complete inventory proves no integration-test changes, the coordinator can return that report without another agent.
Missing inventory is a coverage gap, not a no-change result.

### Protect API contexts

Source-free means no source code or source-derived reasoning.
Use one source-free reviewer for `review-public-api-changes` and `review-public-api` when the runtime supports the phases below.
The source-reading coordinator and `review-api-design` cannot share this context.

Before launch, make sure that the runtime excludes inherited source context and premature docs.
A new reviewer ID alone does not establish a clean context.
The [classification](../review-public-api-changes/SKILL.md#evidence-boundary) and [design](../review-public-api/SKILL.md#evidence-boundary) skills define permitted evidence.

If [package presence](#package-presence) proves that no library is affected, record the area-defined results without launching output reviewers.
The mandatory API report still states the proved result.
Otherwise, use this phased handoff:

1. Supply the owner skills, package-presence facts, selected configuration and matching captures.
   Finalize and save the classification report and its human-review decision without docs.
2. Draft the design findings from the same captures.
   Save the output-only draft, its evidence and requested doc paths before supplying any docs.
   Keep the original draft unchanged for recovery.
   An empty draft needs no docs, but does not excuse missing captures.
3. Retrieve matching docs through [Public API evidence](#public-api-evidence).
   Resume only the design phase under its [doc rules](../review-public-api/SKILL.md#review-rules).
   Docs can only remove or narrow draft claims. Never reopen classification.
4. Return a distinct result for each area, including the mandatory API report and human-review decision.
   A completed classification does not establish completed design coverage.

If the runtime cannot resume a one-shot reviewer, use one clean replacement for the doc phase.
Supply the design skill, saved draft, matching captures and facts, and scoped docs.
Do not repeat classification or draft new findings.
Do not supply docs early merely to fit both phases into one call.
Do not reuse a docs-exposed context for later output-only work.

If required docs remain unavailable, retain the draft internally and report incomplete design coverage.

If forbidden input reaches a reviewer, stop the phase and discard results produced after exposure.
Treat unknown context history as exposure.
Retain only results saved before exposure with a known evidence boundary.
Apply the [single replacement limit](#bound-handoffs-and-follow-ups) to unfinished work in a clean context.
Instructions to ignore prior input do not restore isolation.

If a clean context is unavailable, mark the affected coverage `could not review` and retain valid completed results.
Never substitute source-based API inference.

At merge, independent source evidence or a consumer probe can disprove an output-only claim.
Keep the original result unchanged and record the refuting evidence separately.
Remove or narrow the delivered claim without feeding that evidence back into the output-only phase.
Do not add output-only claims or declare missing capture coverage complete from source evidence.
Retain any material unresolved classification gap and derive the human-review decision from the surviving supported changes.

### Bound handoffs and follow-ups

Give each reviewer only the material needed for its scope:

- The assigned skills, bounded objective, completion condition and required [per-area result](../review-delivery/findings-contract.md#area-result-and-final-summary).
- Exact revisions or local input digest, packages, configuration, owned paths and connected contracts.
- Applicable repository rules, execution limits and relevant CI or discussion facts.
- Known findings with their evidence and thread IDs, matching artifacts and supplied commands.

Apply the [test-first](#combine-test-review) and [API context](#protect-api-contexts) restrictions before sending any handoff.
Separate shared facts from reviewer conclusions and unresolved questions.
Use known findings to avoid duplicate reports, not as proof for another area's claims.
Reuse fields from the [coverage record](#record-coverage) and [artifact identities](#shared-setup-and-reuse), not separate handoff reports.
Provide access to every assigned change and the context needed to trace it, within the phase's evidence boundary.
Send artifact paths and short diagnostics, not transcripts, raw logs, speculative reasoning or unrelated skill instructions.
Do not re-hash unchanged evidence at each handoff.
Keep the producer's freshness checks and the [final input comparison](#when-the-head-moves).

Reviewers return area results with decisive evidence paths, reviewed scope and exact blockers for unfinished traces.
They do not post, repeat shared setup or launch reviewers.
The coordinator updates the existing coverage record.
A reviewer exit, empty reply or unsupported operation does not establish completed coverage.

If the runtime supports follow-ups, reuse the reviewer ID for the same task.
Send only changed facts, missing evidence and the exact question or completion condition.
For a contradiction, include the conflicting claims and their evidence, not another reviewer's full reasoning.
Resume blocked work only when its input or blocker changes.
If a replacement is needed, use at most one for the unfinished scope.
Supply saved valid results and matching evidence under the phase's restrictions.
Do not chain replacements after the same failure or repeat completed coverage.
If work remains blocked, retain valid results and mark the unfinished scope `could not review`.

### Schedule evidence by dependency

Prepare the [shared discovery record](#shared-setup-and-reuse) before assigning dependent work.
Launch a reviewer only when all required inputs for its assigned phase are ready:

| Work | Required inputs |
| --- | --- |
| Source inspection | Pinned source, scope, rules and shared discovery relevant to the assigned trace |
| Test review | Baseline tests and changed expectations under the [test-first handoff](#combine-test-review) |
| Output-only API phases | Matching complete captures and package-presence facts under the [phased handoff](#protect-api-contexts) |
| Integration-test classification | The complete [owned test inventory](#integration-test-evidence) and required base/head contents |
| Benchmark comparison | Shared package and benchmark discovery, followed by the helper's [run prerequisites](../metabench-compare/run-and-report.md#revision-directories-and-prerequisites) |
| Benchmark follow-up | The helper's report and artifacts, including explicit blocked or partial results |

Keep source-based performance review separate from measured conclusions.
Finish source performance coverage without waiting for measurements.
Use a retained reviewer or the coordinator for later measurement interpretation under `review-perf`.
Do not repeat source review or delay a one-shot source reviewer for the helper's report.
Known blockers produce coverage limits, not repeated waiting reviewers.
Keep build and artifact directories separate.
Run independent bundles in parallel only within inherited execution limits.

The [benchmark measurement policy](../metabench-compare/SKILL.md#2-prepare-and-run-the-comparison) controls measurement isolation.
If concurrent work is forbidden, finish or pause owned agents, builds, tests and background commands before measurement.
This includes API capture and benchmark preparation commands.
Make sure that no other owned work remains active. If it cannot pause safely, wait for it to finish.
Resume other work only after the isolated measurement period ends.
Do not stop unrelated processes to make room.

## Shared setup and reuse

Inherit the session's permissions and execution limits. A child or worktree is not a sandbox.
Never switch the shared checkout, rewrite lockfiles or mutate reviewed inputs.
Do not fetch all dependencies or build the workspace before source review.

The coordinator collects package metadata, changed-path ownership and test/benchmark ownership once for both revisions.
The benchmark helper owns native-case listing and benchmark commands.
Keep the complete inventories separate from selected review scopes.
Supply matching records to consumers instead of repeating discovery per area.
Use [package presence](#package-presence), [integration-test evidence](#integration-test-evidence)
and [benchmark evidence](#benchmark-evidence) for their scope and ownership rules.

Record each artifact's identity beside its path:

| Identity | Required evidence |
| --- | --- |
| Inputs | Exact revision, plus the input digest for local changes |
| Scope | Package, manifest, library or target, and inventory coverage |
| Configuration | Features, default-feature mode, target, flags and relevant environment |
| Tools | Resolved compiler and Cargo versions, plus generator or measurement tool versions |
| Result | Producing command, completion status and known gaps |

Reuse an artifact only when every applicable identity field matches the requested work.
A filename, toolchain alias or unchanged `HEAD` alone does not establish a match.
Review local inputs in place, including staged, unstaged and untracked changes.
A clean checkout is not a substitute.
If inputs change during production, mark the artifact stale and rebuild only dependent evidence.
Missing identity evidence prevents reuse.

Each evidence owner still controls completeness, freshness and comparability.
Follow [Public Docs](../review-public-docs/SKILL.md#2-generate-rustdoc-json-once-per-revision) for JSON reuse.
Follow the [benchmark run contract](../metabench-compare/run-and-report.md) for digests, configuration and measurement reuse.
Do not apply API feature defaults to benchmarks or treat partial output as complete.

### Repair shared setup once

Record each setup result's host, artifact identity, error and affected consumers.
The coordinator owns one bounded repair attempt for a shared blocker.
Consumers reuse that result instead of repeating probes, installation or failed builds.
Share prerequisite results across packages only when their required setup matches.
Repair only a diagnosed cause within inherited permissions.
Before rerunning a failed command, identify the changed input or prerequisite that addresses its failure.
If nothing changed, retain the blocker without retrying.
Install only a required missing tool after command evidence and within inherited permissions.
Do not substitute tools to clear a blocker.
After a repair, regenerate affected evidence before launching its waiting consumers.
If repair fails, retain the exact cause and continue independent work.
Do not restart the repair budget per area.

## Package presence

Map changed files to packages at both revisions.
Retain the complete package inventory with exact revisions or the local input digest.
Workspace membership alone does not make a package affected.

Keep source-review scope separate from the API capture set.
Use [applicability routing](#establish-applicability) to identify source consumers.
Their public surface can remain unchanged.
For API captures, include affected libraries and consumers whose surfaces depend on changed inputs or contracts.
Record the connection or unresolved question for each included consumer.
Do not exclude a consumer while its possible surface change remains unresolved.

Use `cargo metadata --locked --no-deps --format-version 1` when permitted.
Otherwise read the complete workspace/package inventory at each revision.
Match renamed or moved packages by manifest history, package name and library target.

| Base | Head | Result |
| --- | --- | --- |
| Present | Present | Compare both real surfaces |
| Absent | Present | Added library, every head item is new |
| Present | Absent | Removed library, every base item is removed |
| Unknown | Any | Resolve inventory before claiming coverage |
| Any | Unknown | Resolve inventory before claiming coverage |

Prove absence from a complete inventory, not failed builds or PR descriptions.
An absent side is a logical empty surface, never fabricated tool output.
Only complete inventory and routing evidence can establish that no library is affected.
An affected library still needs matching captures even when source inspection predicts no API change.

## Public API evidence

An API capture is saved `cargo public-api` output.
Apply [package presence](#package-presence) before capture setup.
If no library is affected, skip captures under those rules.
Otherwise, prepare captures once for both public API areas.

1. Select package/library, features, default-feature mode, target, toolchain and flags.
   Without a caller configuration, use `--all-features` and the host target.
   Keep both revisions comparable.
2. Apply the [shared identity rules](#shared-setup-and-reuse) before reusing captures or JSON.
   Use separate owned revision and target directories.
3. Read installed `cargo public-api --help` and version once for the selected setup.
   Install the official tool only after a missing-tool failure and within permissions:
   `cargo +stable install cargo-public-api --locked`.
4. Generate lock-preserving rustdoc JSON with the
   [public-docs generation procedure](../review-public-docs/SKILL.md#2-generate-rustdoc-json-once-per-revision).
   Reuse this JSON later for doc retrieval.
   Make sure that the selected compiler resolves directly.
   `rustup toolchain list` alone does not establish absence of a custom Microsoft toolchain.
5. Render full output from saved JSON with the command below.
   Help can omit supported flags. If support is unclear, try this documented command once against matching saved JSON.

   ```text
   cargo public-api --rustdoc-json <json-path> --color=never --include function-parameter-names
   ```

   Save stdout as the full API capture. `--locked` belongs on Cargo's JSON
   generation command, not on a `cargo-public-api` version that rejects it.
   Do not use `-sss`, which hides impls needed for absence claims.
   Record the installed version, exit status and output.
   An unrecognized flag is a capability limit. Use a documented compatible path or report the blocker, not repeated flag guesses.
6. If JSON generation fails on `-Z`, use only the selected compiler's supported
   path within inherited limits. Do not silently substitute nightly, enable
   bootstrap against caller policy, or repeat the same failed command.
   Record the failure under [shared setup recovery](#repair-shared-setup-once) and continue independent source review.
7. At the doc phase of the [API handoff](#protect-api-contexts), request the draft's required docs from
   [Review Public Docs](../review-public-docs/SKILL.md#3-extract-the-requested-docs).
   Return its scoped bundle, not findings.

Keep complete output and errors in artifacts. Supply reviewers with capture paths,
exact configuration, package-presence facts and short diagnostics.

## Integration-test evidence

Prepare the complete inventory defined by [Review Integration Tests](../review-integration-tests/SKILL.md#procedure) once at both revisions.
Retain ownership links and exact base/head contents, including unchanged tests affected by changed inputs.
Share it with both test areas through the [test-first handoff](#combine-test-review), without repeating discovery.

Pass established requirements or maintainer decisions separately from PR claims.
Supply focused commands or matching CI evidence within inherited execution limits.
Follow the [test evidence rules](../review-tests/SKILL.md#evidence) and [integration classification rules](../review-integration-tests/SKILL.md#procedure).

Keep the full inventory in artifacts, not the public report.

## Benchmark evidence

Give `metabench-compare` exact revisions, shared package metadata, benchmark ownership,
local inputs, repository rules, host and owned paths. Include completed discovery and its gaps.
Its [selection and measurement policy](../metabench-compare/SKILL.md) owns the
three-crate limit, native-case discovery, scheduling and comparisons. Do not repeat that policy in prompts.

Send its report and paths to raw artifacts to `review-perf`. Retain the complete report
for delivery, including blocked or inapplicable results.
The performance reviewer interprets the supplied results without rerunning benchmarks or expanding crate selection.
Keep unmeasured cost concerns conditional.

Deliver the supported review before pending benchmark preparation or isolated execution.
Keep its `### Benchmarks` block as a [pending or separate-result notice](../review-delivery/findings-contract.md#area-result-and-final-summary).
Record the benchmark owner, pinned inputs, queued/running state and next action in the existing review record.
Pending means work remains assigned, not that it was completed or silently dropped.
Use `blocked` for a known execution blocker and `not applicable` only with the helper's absence evidence.
Pending measurements alone do not make finished source coverage incomplete or prevent its verdict.
They cannot support a performance claim.

If measurements already run under an isolation rule, finish the active command or reach a safe pause before delivery.
Do not overlap delivery work with isolated measurements or stop unrelated processes.
After delivery, continue the queued comparison under the same measurement policy.
Do not remove its inputs or artifacts before consumers finish.
Send the pending owner and state with the review receipt so a coordinating session does not drop the follow-up.
Use [Review Delivery's benchmark follow-up](../review-delivery/SKILL.md#benchmark-follow-up) for the later report and any supported performance findings.
Benchmark completion does not trigger a new full review or an automatic vote or label change.

## When the head moves

Record source head, target repository/ref, merge base and observed target tip
separately. The merge base defines the reviewed PR diff. The target tip supplies
integration context. Stale ADO `lastMergeTargetCommit` or CI metadata does not
prove that the actual target branch is unchanged.

| Change | Action |
| --- | --- |
| Retarget, changed merge base, force push or rebase | Fresh review |
| New commits after incomplete coverage | Fresh review |
| Fast-forward commits after complete coverage | Refresh existing findings, or run a full review if the caller requires it |
| Target tip only, unchanged merge base and PR diff | Preserve source coverage, refresh CI/integration claims |

Before local delivery, make sure that the input digest still matches the reviewed state.
If it changed, review the updated local state before delivery.
Update affected inventories and artifacts, then reconcile every area's scope and status.
Reuse completed work only when its inputs and relevant dependencies still match.
If this work cannot finish, report incomplete coverage without a verdict for the new inputs.

For a findings-only refresh:

1. Read the complete `reviewedHead..currentHead` delta and current affected source.
   If this evidence is unavailable, request a fresh review.
2. Make sure that every merged finding still applies, including pending and previously posted findings.
   Keep unchanged root causes, update moved evidence, drop fixed causes and
   report uncertain ones only as limits.
3. Preserve full-review revision separately from the revalidated current head.
   Do not search for new findings or claim full coverage of later commits.
4. Pass surviving findings and their symbol/thread IDs to Review Delivery.
   It maps them onto the full current PR diff and posts missing inline comments.
   A refresh changes the vote to comment-only, not the placement of findings.
5. Keep API/test classifications and benchmark results tied to their assessed
   revisions. Mark unrechecked blocks historical. Do not make a new label
   decision from stale reports or rerun benchmarks as part of a finding refresh.

Use Review Delivery's [fresh provider checks](../review-delivery/SKILL.md#inputs-and-preparation) before publication and during write recovery.
If the head moves during partial delivery, retain confirmed provider IDs and apply the table before recovering pending writes.
A partial delivery does not turn a required fresh review into a findings-only refresh.
Never publish a stale full report as current by adding a disclaimer.
Historical publication requires an explicit request and remains comment-only.

## Repository notes

Use these only for the named repositories and when consistent with current rules.

| Repository | Notes |
| --- | --- |
| `microsoft/oxidizer` | Prefer existing `tick`/`Timestamp`, `anyspawn`/`Spawner`, `seatbelt`, `recoverable`, `testing_aids` and `tracing`. Prefer `jiff` over `chrono`/`time`, and `opentelemetry` without an unneeded SDK. Cite decisive adopted `M-*` rules. |
| ADO `o365exchange` / `ox-sdk` | Use its equivalents and consumed Oxidizer crates. Keep internal service details out of public-facing output. |
| Other repositories | Follow local conventions. Do not introduce Oxidizer crates by default. |

For Oxidizer, use targeted commands such as `just package=<crate> test <name>`,
`cargo build -p <crate>` or `cargo +nightly miri test` only within inherited limits.
Do not run blanket `just lint`, `just check` or `just format-check`.
The formatting command can also fail on Windows path-length limits.
