# Run and report contract

Use the reviewed repository's metabench version, not an assumed current CLI.
The examples below require metabench support for `--export-json`, `--export-md`, `--baseline` and `--no-baseline`.
If installed help lacks these flags, use its documented equivalents or report that the comparison is blocked.

## Revision directories and prerequisites

Read package metadata at each revision with
`cargo metadata --locked --no-deps --format-version 1`.
Read the lockfile to resolve the Gungraun version for each selected target.
If metadata cannot run, use manifests for discovery and record the execution limit.
Use `gungraun-runner --version` and `valgrind --version` on the Linux host.
If the revisions require incompatible runner versions, report that limit rather than comparing different measurement tools.

Use separate, owned revision directories and external Cargo target directories.
For a committed revision, use `git worktree add --detach <owned-path> <revision>`.
For a local head, run against the caller's actual working tree.
If the authorized Linux host differs, preserve every local input in an owned copy.
Record that copy's identity and transfer method.
Never use a clean checkout as a substitute for dirty local inputs.
Record a digest of local inputs before and after each run.
Include tracked and untracked source, manifests, lockfiles, Cargo configuration and benchmark fixtures in that digest.
If the inputs change, mark the affected measurements stale.
Do not present stale measurements as results for the current local head.

Keep the toolchain, host, target triple, features, profile, compiler flags and relevant environment identical across each pair.
Use the caller's benchmark configuration or the repository's documented benchmark configuration.
Without either, use Cargo defaults plus each target's `required-features`.
Do not inherit the public API review's `--all-features` default.
If no common configuration can run both revisions, mark the pair non-comparable.
Use each revision's own committed dependencies and record lockfile differences.
Do not rewrite lockfiles, dependencies, benchmark inputs or generated registrations to make the pair run.
Set a separate external `CARGO_TARGET_DIR` for each revision.
Set `GUNGRAUN_HOME` to an owned directory for each run.
Clear inherited engine filters and baseline overrides in the child process only.

Save reports outside reviewed inputs in the session's artifact directory.
Use a unique run directory with separate package, target and revision paths.
A benchmark target name is not globally unique.
Preserve stdout, stderr, exit status, raw Gungraun summaries and the resolved command for each run.
Keep reports and their referenced artifacts together before removing build directories.

## Commands and failures

Resolve every placeholder before execution.
Use the same Cargo configuration arguments for the pair.
In each revision directory, run the target's help first:

```text
cargo bench --locked -p <package> --bench <target> <cargo-configuration-args> -- --help
```

For each supported target, save its native case listing before measurement:

```text
cargo bench --locked -p <package> --bench <target> <cargo-configuration-args> -- --gungraun --list
```

Match the listing to registrations and the final report.
If listing fails, retain that coverage gap even when measurement returns some rows.
For registered cases missing from output, record missing measurements rather than inferring removal.

For metabench versions with these flags, run the base command:

```text
cargo bench --locked -p <package> --bench <target> <cargo-configuration-args> -- --gungraun --no-baseline --export-json <base-json> --export-md <base-md>
```

When the base JSON is complete and usable, run the head command:

```text
cargo bench --locked -p <package> --bench <target> <cargo-configuration-args> -- --gungraun --baseline <base-json> --export-json <head-json> --export-md <head-md>
```

Use metabench's direct comparison rather than building another comparison engine.
The base JSON captures the reference values.
The head JSON contains current values, matched prior values, percentage changes and benchmark classifications.
Comparison occurs during the head invocation.
Do not invent a separate capture or compare subcommand.

Keep the base JSON unchanged for subsequent inspection.
Do not rely on implicit rolling baselines or native Gungraun baseline names across separate run directories.

Make sure that both commands finish successfully and produce fresh reports.
Reject smoke runs, empty selections and stale or `.partial` reports as complete measurements.
Do not pass `--test`, use `--no-run` as measurement evidence, or disable regression failures.
If an explicit baseline is incompatible, retain its error or warning.
If no usable base report exists, run the head with `--no-baseline` and mark the comparison unavailable.
If comparison warns but produces a complete head report, reuse it without another measurement.
If baseline loading prevents report publication, collect the head once with `--no-baseline` in a new output path.
Keep both attempts in the artifacts. State the comparison failure briefly in the visible report.

For a target present on only one side, run that side with `--no-baseline`.
Prove absence from the complete inventory, not from a failed command.
Keep a failed target in the report with a short failure reason and missing measurements.
Save its command and full error in the artifacts.
After a failure, continue independent targets when the host and inherited limits allow execution.
Do not retry deterministic workload failures or edit the reviewed code.

## Side-by-side results

Read the report schema and native-to-canonical identity mappings from the installed version.
A canonical identity is the group, benchmark and case name.
Use the union of inventories and report identities from both revisions.
Include failed cases from logs even when no metric exists.
Do not join by display name alone or merge similarly named cases.
For renamed cases, show separate identities unless an explicit mapping proves they represent the same workload.
If inputs, setup, teardown or measurement boundaries change, mark the affected pairs non-comparable.
An implementation change alone does not invalidate comparison of the same workload.

Use JSON to make sure that the visible results cover every case and use the correct instruction counts.
Metabench's terminal and Markdown tables can omit metric columns to fit their width.
The head report contains only current identities, so recover removed benchmarks from the base JSON.
Use metabench's `prior`, `change_percentage` and `status` fields when present and compatible.
Keep the benchmark's aggregate status separate from each metric's direction of change.

Return one compact `### Benchmarks` block:

1. State status, short base/head revisions and measured case coverage in at most two sentences.
2. Show metabench's result output with before/after instruction counts.
   Reuse its terminal result block in a text fence or its exported Markdown table when it already provides the comparison.
   Omit build logs, progress output and repeated headers.
3. If native output lacks the instruction comparison or expands each case into many metric rows, use the compact table below instead.
   Read the saved JSON, not a new benchmark run.
   Include removed, failed and missing cases that native output omits.
4. Add only warnings that affect the comparison and one artifact reference.
   Save the original metabench output and full run evidence there.
   Do not repeat the same results as both native output and a custom table.

Use one row per benchmark case, with package and target labels above each table.
Keep engine or statistic variants distinct when a target reports more than one.
Use readable benchmark names, not aliases that require a separate lookup table.
Escape untrusted names as literal Markdown cell content.
For Gungraun Callgrind, use `Ir` when the installed schema identifies it as the instruction count.
Do not substitute estimated cycles or another metric when instructions are unavailable.
Keep integer counts and absolute deltas exact. Round displayed percentages to two decimal places.

| Benchmark / case | Instructions before | Instructions after | Change | Change % |
| --- | --- | --- | --- | --- |
| parser/checksum/small | 1000 | 1100 | +100 | +10.00% |

This row illustrates the format. It is not measurement evidence.
Show all cases, including unchanged results, not only regressions.
Do not expand the visible report into cache metrics, hit rates, estimated cycles or an all-metric table unless the user requests them.
Do not add selection tables, environment inventories, hashes, permission history or repeated blank lines to the visible report.

For compatible numeric pairs, calculate `Change = Head - Base`.
Use metabench's percentage when available, otherwise calculate `Change % = 100 * (Head - Base) / Base`.
For a zero base, show the absolute change and `N/A` for the percentage.
For missing values, show `N/A` and the reason, never zero or an unchanged result.
Distinguish added, removed, failed, unavailable and non-comparable cases or metrics.
For non-comparable pairs, show raw values but `N/A` for both change columns.
Put a shared reason below the table once, rather than repeating it in every row.
When a package or target is new, state that no before-change baseline exists.
Show its measured instruction counts, but do not claim improvement or regression.

Compare only matching identities, workloads, measurement boundaries, tools, units, metric directions and build configurations.
Include statistic and tool configuration in the comparison requirements.
For lower-is-better metrics, positive change means increased cost.
For higher-is-better metrics, positive change means improvement.
For unknown directions, report the numeric change without a performance judgment.
If metabench skips comparison or warns about incompatible metadata, preserve the warning and do not assign improvement or regression.
If its report schema differs, show separately decoded values without bypassing a rejected baseline.

Record the repository's configured regression threshold in the artifacts when one exists.
Otherwise retain the installed metabench default and record its value.
Do not confuse a percentage threshold with statistical significance.
Instruction counts and estimated cycles do not prove wall-clock speed or throughput.
Do not invent timing ranges or confidence intervals for Gungraun counts.

Save the compact report, original metabench output and complete, unrounded JSON for both revisions.
Keep every metric in JSON, including cache metrics and estimated cycles. Do not generate a second all-metric Markdown table by default.
Save selection reasons, full revisions, inventories, host, tool versions, configuration, commands, digests and failure details with those artifacts.
The compact report does not reduce measurement coverage or compatibility requirements.

If the compact results exceed provider size limits, link the complete compact report through an authorized artifact location.
Include the revisions, coverage and material limits beside that link.
If no accessible attachment or link is available, return the complete compact report locally and flag provider delivery as blocked.
When the compact results fit inline, unavailable artifact hosting does not block their delivery.
Give local artifact paths only to a caller who can access them, not as provider links.
Do not split it into extra provider comments without permission.
Never silently truncate rows or expose private artifacts through a public upload.

## Tool references

Use these references to locate the tool contracts.
Read documentation at the reviewed revision when its behavior differs.
Do not assume that upstream defaults match the installed version.

- [Metabench documentation](https://github.com/microsoft/oxidizer/tree/main/crates/metabench)
- [Metabench report and comparison implementation](https://github.com/microsoft/oxidizer/blob/main/crates/metabench/src/report.rs)
- [Metabench capture and baseline orchestration](https://github.com/microsoft/oxidizer/blob/main/crates/metabench/src/runner.rs)
- [Gungraun guide](https://gungraun.github.io/gungraun/latest/html/)
- [Performance review rules](../review-perf/SKILL.md)
