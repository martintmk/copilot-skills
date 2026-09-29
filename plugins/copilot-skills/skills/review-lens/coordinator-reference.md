# Review Lens coordinator reference

Details the coordinator needs for less common situations. Area skills do not
read this file.

## Package presence

The public API area compares a library's surface at base and head. When a
library was added, removed, moved or renamed, one side may have no package.
Establish this before the public API area starts, and pass the result.

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
   | unknown | any | Resolve it, or the public API area cannot review this library. |

A package is absent only when a complete inventory of that revision shows it.
A failed build, a failed `cargo metadata`, a feature error or a PR description
saying "new crate" does not prove absence. Never invent an empty side for a
package that exists.

Decide per library. A new crate does not remove problems with an existing one.

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
