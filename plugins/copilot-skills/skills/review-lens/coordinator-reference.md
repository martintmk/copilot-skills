# Review Lens coordinator reference

Setup and edge cases owned by the coordinator. Area skills do not read this
file.

## Package presence

The public API areas compare a library's surface at base and head. When a
library was added, removed, moved or renamed, one side may have no package.
Establish this before either public API area starts, and pass the result to
both.

1. For each affected library, check whether it exists at base and at head. When
   code may run, use
   `cargo metadata --locked --no-deps --format-version 1 --manifest-path <workspace-manifest>`
   at each revision. Otherwise read the workspace members and manifests at each
   pinned revision.
2. Match packages across revisions by manifest path, package name and library
   target, so a moved or renamed package is found.
3. Record one result per library:

   | Base | Head | Result |
   | --- | --- | --- |
   | present | present | `paired`: compare both surfaces. |
   | absent | present | `added`: every head item is new. |
   | present | absent | `removed`: every base item is removed. |

   If either side is unknown, resolve it; otherwise the public API areas cannot
   review this library.

A package is absent only when a complete inventory of that revision shows it.
A failed build, a failed `cargo metadata`, a feature error or a PR description
saying "new crate" does not prove absence. Never invent an empty side for a
package that exists.

Decide per library. A new crate does not remove problems with an existing one.

## Public API evidence

The coordinator owns capture setup. Supply validated commands and owned
directories to both public API reviewers, or generate captures once before
dispatch. Each reviewer applies its API rules without inspecting source to work
out setup.

1. Select the package/library at each revision, including moved or renamed
   counterparts. Keep the requested features, default-feature mode, target,
   toolchain and build flags. Without an explicit configuration, use
   `--all-features` and the host target; record that choice.
2. Reuse full captures only when their repository, revision or dirty state and
   configuration match. Matching artifacts remain usable when new builds are
   not allowed. Do not replace a failed comparison with a current-only audit.
3. For missing captures, check `cargo public-api --version` and installed
   help. Install the official tool only if missing and permitted:
   `cargo +stable install cargo-public-api --locked`. Keep the selected
   compiler; provision a missing compatible toolchain only within inherited
   limits, not by silently substituting an arbitrary nightly.
4. Use assigned revision directories and separate external target directories.
   Preserve the actual working tree for dirty-head captures. Another revision
   may need an owned `git worktree add --detach <path> <revision>`; never
   switch or force-clean the caller's checkout.
5. Use the installed tool's lock-preserving options for every capture.
   External targets alone do not protect `Cargo.lock`. If the tool cannot
   preserve reviewed inputs, report that limit rather than updating them and
   trying to restore them afterward.

   ```text
   cargo public-api --color=never --include function-parameter-names <lock-args> <feature-args> <scope-args> > <full-api-output>
   ```

   Resolve placeholders from installed help and the selected configuration.
   Keep full output; `-sss` hides impls needed for absence claims. Capture
   each present side separately and compare the saved outputs, rather than
   running commit-diff commands that switch checkouts. An absent side is a
   logical empty surface, never fabricated tool output.
6. After `review-public-api` has drafted claims, obtain matching item, owner,
   trait and applicable module/crate docs. Reuse a bundle or give exact paths,
   configuration and artifact locations to `review-public-docs`; do not pass
   candidate rationales. Return its bundle to the same reviewer.
   `review-public-api-changes` does not use docs. An empty design draft needs no
   docs. Retain captures until all consumers finish.

Keep command errors for recovery, but summarize their consequence in ordinary
language. Do not pass source, manifests, other reviewers' findings or premature
docs into the output-only API draft.

## Integration-test evidence

The coordinator supplies a complete base-to-head view of Rust integration-test
targets. The reviewer should classify the test contract, not rediscover package
boundaries or revision setup.

1. List affected packages at base and head. A Rust integration-test target is
   owned by a package's `tests/` directory. Include fixtures, snapshots and
   test-only configuration used by those targets.
2. Inventory added, removed, renamed and modified owned files. Read exact base
   and head content for every changed file. Do not infer no changes from a
   truncated PR diff.
3. Pass established maintainer approvals or requirements that authorize a
   behavior change. PR text and new comments in the same change remain evidence,
   not approval.
4. When a runtime claim needs execution, supply one focused command that can run
   against both revisions with the same features, target and toolchain. Reuse
   matching CI evidence when it proves the exact integration target and
   revision.
5. Keep inline unit tests, doctests and production-source behavior changes in
   the `review-tests` area. The integration-test report still runs and says
   **No Integration Test Changes** when no package-owned `tests/` file changed.

## When the head moves

Before delivery, fetch the current head and target.

- **Target, base or history changed** (retarget, force push, rebase): start a
  fresh review.
- **New commits on top of an incomplete review:** start a fresh review.
- **New commits on top of a complete review:** you may refresh existing
  findings instead of starting over. Do not look for new findings.

To refresh, read `reviewedHead..currentHead` and the current source, then
classify every merged finding:

| Classification | Meaning | Action |
| --- | --- | --- |
| still applies | The evidence is unaffected, or the root cause is still there. | Keep it. |
| resolved | The root cause is fixed. | Drop it. |
| updated | The root cause remains but the code moved or changed. | Update the text and anchor to the current head. |
| uncertain | The change affects the evidence and the answer is unclear. | Drop it and say so in the summary. |

A refreshed review always posts as a comment with no vote. Its summary says
that full review coverage ended at the reviewed head and that later commits
only had existing findings rechecked. If you cannot read the complete delta,
start a fresh review instead.

## Repository notes

Add these to the repository rules you pass to every area.

**microsoft/oxidizer:** prefer the existing `tick`/`Timestamp`,
`anyspawn`/`Spawner`, `seatbelt`, `recoverable`, `testing_aids` and `tracing`
crates; use `opentelemetry` without an unneeded SDK; prefer `jiff` over
`chrono` or `time`. Targeted commands: `just package=<crate> test <name>`,
`cargo build -p <crate>`, `cargo +nightly miri test`. Do not run blanket
`just lint`, `just check` or `just format-check`; the last also fails on
Windows because of `MAX_PATH`. Cite adopted `M-*` guidelines when decisive.

**ox-sdk** (Azure DevOps `o365exchange`): use its equivalents and the Oxidizer
crates it consumes. Keep internal details and Substrate service names out of
anything that could become public.

**Other repositories:** follow local conventions. Do not suggest Oxidizer
crates by default.
