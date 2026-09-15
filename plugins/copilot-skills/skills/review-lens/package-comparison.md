# Package presence and one-sided comparisons

Read when establishing or consuming API/docs change-comparison scope. Only
proven absence permits a logical empty side, never failed/missing extraction.
Resolve presence before package-selected Cargo commands.

## Establish presence once

The source-capable coordinator or standalone caller produces a versioned
`packageComparison` per affected package. Record:

- Repository identity, exact comparison base/head including dirty-state identity,
  effective features/default-feature mode, target and toolchain.
- Per side: package/library identity, manifest location,
  `presence: present | absent | unknown`, producing context, exact revision,
  complete inventory/tree artifact and hash.
- Derived `mode: paired | added-package | removed-package | unknown`.

Resolve counterpart identity across revisions, not just head name/path:

- Use complete successful package/workspace inventories and pinned tree/manifest
  evidence. When execution is permitted,
  `cargo metadata --locked --no-deps --format-version 1 --manifest-path <workspace-manifest>`
  preserves locks; match members, declared names, manifest paths and library targets.
- Account for renamed/moved packages, membership changes, exclusions and standalone
  manifests. One workspace's missing match does not prove no counterpart.
  Without a Cargo workspace, a complete pinned tree/manifest inventory can prove
  the first package; do not run Cargo against a nonexistent workspace.
- Unavailable revisions, missing artifacts, selector errors, feature/target gating,
  sparse omissions, registry/auth failures and unsuccessful inventories mean
  `unknown`, not `absent`.
  "cannot specify features for packages outside of workspace" and PR descriptions
  claiming "new crate" prove nothing.

API/docs workers consume this compact factual record without opening underlying
source/manifests. An output-only assignment lacking it asks the context owner or
blocks; never relax isolation to resolve ambiguity. Presence is scope provenance,
not API-quality, documentation, runtime or compatibility finding evidence.

## Comparison modes

| Base presence | Head presence | Mode and required evidence |
| --- | --- | --- |
| present | present | `paired`: real matching captures and baseline-to-head comparison |
| absent | present | `added-package`: logical empty base versus complete real head capture |
| present | absent | `removed-package`: complete real baseline capture versus logical empty head |
| unknown on either side, or absent on both | any | `unknown`: resolve scope or block |

One-sided modes remain explicit change comparisons, not head-only fallback:

1. Retain both exact revisions and presence proof. Generate/reuse only present-side
   artifacts. Never select/build the absent package, run two-revision builds
   against it, create placeholder crates or substitute published/unrelated packages.
2. Represent absence as a logical empty set, not fabricated `cargo public-api`
   output/rustdoc JSON or a successful extraction. Never parse presence as docs.
3. Cover **all** emitted head additions or baseline removals. Cite exact API
   output and use only present-side real rustdoc JSON. Presence alone proves
   neither defect nor semver verdict.
4. Preserve fresh workers, artifact matching, full coverage and mandatory API
   filtering. Successful one-sided comparisons are `completed`, not
   `not-applicable`/partial merely because the absent side has no artifact.

Apply per package; one new crate cannot erase existing-package blockers.
Moved/renamed counterparts require explicit per-side selectors and paired evidence.

## Regression cases

| Case | Required outcome |
| --- | --- |
| Proven new crate; head capture succeeds | Complete added comparison; no baseline extraction |
| Scaffold emits only `pub mod example_lib` | Review module and real crate-root docs; empty descendants are valid; fresh API filtering runs |
| Proven removed crate; baseline capture succeeds | Complete removed comparison using baseline API/docs and deleted-item markers |
| Existing/renamed crate; baseline build fails | Block, never invent empty baseline |
| Metadata failure or membership change only | Unknown until context owner resolves presence |
| Revision/configuration mismatch | Reject reuse; refresh affected evidence |
| New crate plus failed existing-crate comparison | Combined review remains incomplete |

Recovery retains the original diagnostic and matching successful present-side
captures. Affected workers must finish under the proven mode; coordinators cannot
merely rewrite old `blocked` results or a queue's `reviewComplete`.
