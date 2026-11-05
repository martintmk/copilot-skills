# Review Lens coordinator reference

The coordinator owns setup and recovery. Area agents receive the facts they need,
not this entire file.

## Package presence

Map changed files to packages at both revisions. Include consumers only when
changed shared inputs or exposed dependency types can affect their public surface.
Record that reason. Workspace membership alone does not make a library affected.

Use `cargo metadata --locked --no-deps --format-version 1` when permitted.
Otherwise read the complete workspace/package inventory at each revision.
Match renamed or moved packages by manifest history, package name and library target.

| Base | Head | Result |
| --- | --- | --- |
| Present | Present | Compare both real surfaces |
| Absent | Present | Added library, every head item is new |
| Present | Absent | Removed library, every base item is removed |
| Unknown | Any | Resolve inventory before claiming coverage |

Prove absence from a complete inventory, not failed builds or PR descriptions.
An absent side is a logical empty surface, never fabricated tool output.
If no library is affected, both API areas can finish without captures.

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
   new findings. An empty draft needs no docs.

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
