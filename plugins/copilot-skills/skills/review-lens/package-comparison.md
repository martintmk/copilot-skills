# Package presence and one-sided comparisons

A package that did not exist at the pinned baseline has no baseline API or
rustdoc artifact to build. Proven absence is a valid empty comparison side;
an unavailable revision, failed extraction or unknown package is not.
Resolve this distinction before invoking package-selected Cargo commands.

## Establish presence once

The source-capable context owner (Review Lens or the caller of a standalone
specialist) establishes a versioned `packageComparison` record for each
affected package. API/docs workers consume this factual scope record without
opening its underlying source or manifests.

Record the repository identity, exact comparison base and head (including
dirty-state identity), effective features/default-feature mode, target and
toolchain. For each side, retain package/library identity, manifest location,
`presence: present | absent | unknown`, and evidence provenance: the producing
context, exact revision, complete inventory/tree artifact and its hash.
Use `mode: paired | added-package | removed-package | unknown` as derived below.

The context owner must resolve identity across revisions, not merely look for
the head's package name or manifest path in the baseline:

- Use complete, successful package/workspace inventories and revision-pinned
  tree/manifest evidence. A lock-preserving
  `cargo metadata --locked --no-deps --format-version 1 --manifest-path <workspace-manifest>`
  can inventory a workspace when execution is permitted; match workspace
  members, declared package names, manifest paths and library targets.
- Account for renamed/moved packages, changed workspace membership, excluded
  packages and standalone manifests. A missing match in one selected
  workspace is not proof that the package had no counterpart.
- If the revision has no Cargo workspace, a complete pinned tree/manifest
  inventory can establish that this is the first package. Do not require Cargo
  to run against a nonexistent workspace just to prove that fact.
- A missing artifact, package-selector error, feature/target gating, sparse
  checkout omission, registry/auth failure or unsuccessful inventory is
  `unknown`, never `absent`. In particular, "cannot specify features for
  packages outside of workspace" does not by itself prove a new package.
- A PR description claiming "new crate" is not evidence. If identity or
  presence remains ambiguous, report the gap rather than invent an empty side.

For an output-only standalone assignment lacking this record, request the
facts from its context owner or report the blocker. Do not relax the worker's
source-inspection boundary to resolve presence itself. The compact record
guides comparison scope only: it supplies no API-quality, doc-text, runtime or
compatibility finding evidence.

## Comparison modes

| Base presence | Head presence | Mode and required evidence |
| --- | --- | --- |
| present | present | `paired`: real matching captures and the normal baseline-to-head comparison |
| absent | present | `added-package`: logical empty base versus the complete real head capture |
| present | absent | `removed-package`: complete real baseline capture versus a logical empty head |
| unknown on either side, or absent on both | any | `unknown`: resolve scope or block; never claim a completed comparison |

For `added-package` and `removed-package`:

1. Keep both exact revision identities and the presence proof. These modes are
   explicit change comparisons, **not** a fallback to a head-only audit.
2. Generate/reuse artifacts only for the present side. Do not select or build
   the absent package, run a two-revision package build against it, create a
   placeholder crate, or substitute a published version or unrelated package.
3. Represent the absent side as a logical empty set in the comparison record.
   Never fabricate `cargo public-api` output or rustdoc JSON for it, label it a
   successful extraction, or parse the presence record as documentation.
4. Scope additions to all actually emitted head API items, and removals to all
   actually emitted baseline items. API findings still cite exact tool output;
   documentation still comes only from the present side's real rustdoc JSON.
   Package presence alone neither proves a defect nor supplies a semver verdict.
5. Preserve the normal fresh-worker, artifact-matching, full-coverage and
   mandatory API-filtering gates. A completed one-sided comparison is
   `completed`, not `not-applicable` or partially assessed merely because the
   absent side has no build artifact.

Apply these rules per package in a multi-package review. Proving one new crate
does not waive paired comparisons for existing crates or erase their blockers.
Moved/renamed counterparts use explicit per-side selectors and paired evidence,
not an invented package addition/removal.

## Regression cases

| Case | Required outcome |
| --- | --- |
| New crate; baseline absence proven; head capture succeeds | Complete `added-package` comparison without attempting baseline extraction |
| New scaffold emits only `pub mod example_lib` | Review the emitted crate module and real crate-root docs; an empty descendant set is valid, and fresh API filtering still runs |
| Removed crate; head absence proven; baseline capture succeeds | Complete `removed-package` comparison using baseline API/docs and deleted-item markers |
| Existing or renamed crate; baseline build fails | Remain blocked; do not treat the failed capture or new name/path as an empty baseline |
| Only metadata lookup fails or workspace membership changes | Presence remains unknown until the context owner resolves it |
| Presence proof or capture is for another revision/configuration | Reject reuse and refresh the affected evidence |
| One new crate plus an existing crate whose comparison fails | Keep the combined review incomplete; do not hide the existing-crate blocker |

When recovering a previous absent-package build failure, retain its diagnostic
and reuse matching successful present-side captures. The affected workers must
finish under the proven comparison mode; a coordinator must not just rewrite
old `blocked` results or a queue's `reviewComplete` flag.
