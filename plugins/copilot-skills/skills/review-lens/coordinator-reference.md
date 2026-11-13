# Review Lens coordinator reference

The coordinator owns setup, routing and recovery. Reviewers receive the facts
they need, not this entire file.

## Evidence-led routing

Route work by its required evidence, not by area names.
Use the complete initial pass from [Review Lens](SKILL.md#1-pin-facts-and-prepare-shared-evidence).
An evidence bundle contains related paths, contracts and unresolved questions.
Routing selects the owner and evidence needed. It does not replace the area's review procedure.

### Record coverage

Keep one internal record for each of the eleven areas in Review Lens.
Use artifact references and short reasons rather than copied source or transcripts.
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
Unfinished traces stay unreviewed, even when the owner found no defects.

### Establish applicability

Start from the complete changed-path inventory at the pinned revisions.
For removals and renames, inspect both old and new ownership, references and discovery paths.
Do not treat a missing head file or a path-only rename as proof that no contract changed.
Use filenames and keyword searches to find evidence, never to prove absence.

Follow connections from changed definitions, values and shared inputs.
Use the table to identify indirect effects, not as another area checklist:

| Changed input | Connection to inspect |
| --- | --- |
| Macros or generated code | Macro and derive consumers, expansion inputs, generated items and callers |
| Types, traits or generic code | Re-exports, aliases, bounds, impls and downstream type or dispatch relationships |
| Features, build inputs or dependencies | Resolved versions, enabled features, target conditions, build scripts and included files that select or generate code |
| Values, errors or control flow | Data reaching unchanged signal emitters, error conversions, recovery classification or middleware |
| Shared fixtures, helpers, snapshots or configuration | Owning test targets and their consumers, including shared inputs outside package `tests/` directories |

Add a consumer only through a specific changed input or contract connection.
Stop expanding when evidence shows that the area's downstream contract is unaffected in the selected configuration.
Record that boundary. Do not audit every dependency or workspace package.
If the boundary is unresolved, record the question instead of assuming either relevance or absence.

Apply these routing decisions separately from completion:

| Applicability | Next action |
| --- | --- |
| Relevant | Assign the confirmed connection and remaining traces to an owner. Complete the area's procedure before returning `done`. |
| Unknown | Resolve the named question with the smallest relevant read or matching artifact. If unresolved, retain the affected coverage gap. |
| Absent with evidence | Cite the inspected changes, connected boundaries and configuration that rule out relevance at both revisions. Use `not applicable` only under the area's rules. |

Relevant does not mean defective. A relevant area with no findings returns `done`, not `not applicable`.
Routing uncertainty does not authorize a wider review without a specific connection to investigate.
Missing or truncated diffs, inaccessible source, incomplete inventories and missing required captures cannot support absence.
Make no claims about configurations outside the evidence.
If missing evidence prevents required review, return `could not review` for the affected area.

Do not exclude test review merely because no test changed.
Follow [Review Tests](../review-tests/SKILL.md).
The mandatory API, integration-test and benchmark blocks remain required, including proved no-change or inapplicable results.
Area skills own proof rules and classifications. Routing cannot replace them.

### Choose review owners

1. Review small connected source changes in the coordinator.
   Apply each relevant area skill rather than replacing them with a general checklist.
2. Keep a single continuous trace with one owner.
   Do not split it merely to create parallel work or separate area labels.
3. Delegate substantial independent bundles when their evidence needs separate context.
   Give each bundle a bounded objective and a clear completion condition.
   Compatible source areas can share a reviewer when they use the same paths and context.
   Require a separate result and coverage record for each assigned area.
4. Escalate complex risks to a reviewer with the relevant expertise.
   Examples include unsafe invariants, concurrency, cross-package contracts and version-specific recovery behavior.
   Separate independent specialist work when combined context obscures its proof.
   Do not add a second reviewer solely to repeat a completed trace.
5. Select reviewer count from evidence scope, required isolation and inherited runtime limits.
   Do not impose an arbitrary agent cap or promise a numerical speedup.
   If a required specialist is unavailable, retain supported work and report the uncovered scope.

Grouping changes ownership, not the proof threshold.
Use each [area skill](SKILL.md#review-areas) for its evidence rules.
Keep severity and output shape in the [findings contract](../review-delivery/findings-contract.md).

### Protect API contexts

The source-reading coordinator cannot perform either output-only API review.
Use separate source-free contexts for `review-public-api` and `review-public-api-changes`.
Do not reuse a source reviewer or a transcript that contains source.
Instructing that reviewer to ignore prior source does not restore isolation.

Supply package-presence facts, selected configuration and matching captures.
Do not pass source excerpts, source diffs, source-derived findings or source-reading discussions.
The classification context never receives docs.
The design context receives docs only after its output-only draft, through the
[API evidence procedure](#public-api-evidence).

If [package presence](#package-presence) proves that no library is affected, record the area-defined results without launching output reviewers.
The mandatory API report still states the proved result.
If captures are required but clean contexts are unavailable, mark the affected areas `could not review`.
Do not substitute source-based API inference.

### Bound handoffs and follow-ups

Give each reviewer only the material needed for its assigned scopes:

- The assigned skills, bounded objective and required [per-area result](../review-delivery/findings-contract.md#area-result-and-final-summary).
- Exact revisions or local input digest, packages, configuration and owned paths.
- Applicable repository rules, execution limits and relevant CI or discussion facts.
- Existing thread IDs, matching artifacts and supplied commands.

Apply the API context restrictions before sending any handoff.
Do not send full transcripts, raw logs or unrelated reviewers' reasoning.
Keep full evidence in artifacts and send paths with short diagnostics.
Reviewers return results without posting, restarting shared setup or launching more reviewers.

Use a retained reviewer ID for a bounded follow-up when supported.
Send only changed facts or missing evidence for that same task.
For a one-shot reviewer, collect its required inputs before launch.
If follow-up is still needed, use one replacement for the missing part.
Do not repeatedly message a finished one-shot reviewer or repeat completed coverage.

### Schedule evidence by dependency

Start independent source inspection without waiting for API captures or benchmark results.
Only the areas that need a missing artifact wait for it.
Keep build and artifact directories separate.
Run independent bundles in parallel only within inherited execution limits.

The [benchmark measurement policy](../metabench-compare/SKILL.md#2-prepare-and-run-the-comparison) controls measurement isolation.
If it forbids concurrent work, finish or pause owned work before measurement.
Resume source inspection after the isolated measurement period.
Do not stop unrelated processes to make room.
Artifact preparation does not waive isolation or authorize extra builds.

## Package presence

Map changed files to packages at both revisions.
Retain the complete package inventory with exact revisions or the local input digest.
Keep this inventory separate from the smaller affected-package set.
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

Prepare captures once for both public API areas. They must not rediscover build setup.

1. Select package/library, features, default-feature mode, target, toolchain and flags.
   Without a caller configuration, use `--all-features` and the host target.
   Keep both revisions comparable.
2. Reuse artifacts only when revision or local input digest and configuration match.
   Use separate owned revision and target directories. Never switch the caller's checkout.
3. Read installed `cargo public-api --help` and version.
   Install the official tool only after a missing-tool failure and within permissions:
   `cargo +stable install cargo-public-api --locked`.
4. Generate lock-preserving rustdoc JSON with the
   [public-docs generation procedure](../review-public-docs/SKILL.md#2-generate-rustdoc-json-once-per-revision).
   Reuse this JSON later for doc retrieval.
   Check the selected compiler directly. `rustup toolchain list` alone does not
   establish absence of a custom Microsoft toolchain.
5. When installed help supports it, render full output from saved JSON:

   ```text
   cargo public-api --rustdoc-json <json-path> --color=never --include function-parameter-names
   ```

   Save stdout as the full API capture. `--locked` belongs on Cargo's JSON
   generation command, not on a `cargo-public-api` version that rejects it.
   Do not use `-sss`, which hides impls needed for absence claims.
6. If JSON generation fails on `-Z`, use only the selected compiler's supported
   path within inherited limits. Do not silently substitute nightly, enable
   bootstrap against caller policy, or repeat the same failed command.
   Return one setup blocker and continue independent source review.
7. After the output-only API draft, supply matching docs for cited items, owners,
   traits and applicable modules. Ask `review-public-docs` for missing text, not
   new findings. Return the bundle to the same API reviewer.
   An empty draft needs no docs.

Keep complete output and errors in artifacts. Supply reviewers with capture paths,
exact configuration, package-presence facts and short diagnostics.
Do not pass source or premature docs into an output-only draft.

## Integration-test evidence

Supply a complete base-to-head inventory of package `tests/` targets, fixtures,
snapshots and owned configuration. Include added, removed, renamed and modified
files and exact base/head contents. A truncated PR diff proves no absence.

Pass established requirements or maintainer decisions separately from PR claims.
When runtime proof is needed, supply one focused command for both revisions with
matching configuration. Reuse matching CI evidence when it proves the exact target.

The report area owns package `tests/` classification. `review-tests` owns whole-diff
coverage findings, inline unit tests and doctests. Deduplicate their shared root causes.
Keep the full inventory in artifacts, not the public report.

## Benchmark evidence

Give `metabench-compare` exact revisions, package ownership, local inputs,
repository rules, host and owned paths. Its
[selection and measurement policy](../metabench-compare/SKILL.md) owns the
three-crate limit, scheduling and comparisons. Do not repeat that policy in prompts.

Send its report and raw artifacts to `review-perf`. Retain the complete report
for delivery, including blocked or inapplicable results.
The performance reviewer interprets the supplied results without rerunning benchmarks or expanding crate selection.
Keep unmeasured cost concerns conditional.
Missing measurements do not prevent source-based performance review from finishing.

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

For a findings-only refresh:

1. Read the complete `reviewedHead..currentHead` delta and current affected source.
   If this evidence is unavailable, request a fresh review.
2. Recheck every merged finding, including pending and previously posted findings.
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

Recheck before publication. If the head moves during partial delivery, retain
confirmed provider IDs and revalidate only pending findings before recovery.
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
