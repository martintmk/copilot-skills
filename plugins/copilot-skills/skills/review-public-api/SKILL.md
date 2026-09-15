---
name: review-public-api
description: >
  Audit a Rust library's exported contract using cargo-public-api output only,
  then isolated rustdoc-based filtering of provisional claims. Use for a
  whole-crate or explicit output-only API audit, or review-lens's mandatory
  output-only pass; small PRs cover changed public items and their immediate
  family. Applies idiomatic Rust API practices and
  API-visible Pragmatic Rust Guidelines. Not for source-based findings,
  implementation correctness, docs quality/consistency, performance or posting.
---

# Review Public API

Apply the [fresh-worker entry gate](../review-lens/worker-isolation.md); an
assigned output-only worker executes here without redispatching itself. Follow
the [findings contract](../review-delivery/findings-contract.md) and
[shared context](../review-lens/review-context.md) for trust, matching, handoff
and cleanup only. Do not restart coordinator setup or invoke `review-lens`.
Source/diff, CI, repository-rule inspection and executable-reproduction
prerequisites do **not** apply; exact public paths replace source anchors.

## Evidence boundary: OUTPUT-ONLY, REPORT-ONLY

- Generate candidates solely from exact `cargo public-api` output. Never
  open/search source, manifests/lockfiles, build scripts, docs, tests/examples,
  source diffs, repository history, rustdoc JSON or rendered docs; do not supplement with
  `cargo metadata`, rust-analyzer or code search.
- Scope/use cases orient review. Tool help/versions, diagnostics, revision IDs
  and artifact metadata guide execution/matching, not API-quality claims.
  `packageComparison` establishes presence/mode only: neither inspect its source
  evidence nor infer absence from failed Cargo commands.
- Mandatory isolated [post-processing](rustdoc-post-processing.md) uses
  `review-public-docs` and only removes/narrows claims. It cannot add/strengthen
  them, increase severity or prove runtime behavior. Keep JSON/full bundles out
  of this worker; returned provenance cannot support new claims.
- Never edit, post, vote or invoke `review-delivery`. Keep captures/targets
  outside the unchanged reviewed worktree; remove only owned resources after
  consumers finish.

Assess consumer-visible contracts using idiomatic Rust conventions and
API-visible [Pragmatic Rust Guidelines][pragmatic-rust]. Output cannot prove
correctness, validation, panics, soundness, allocation/performance, redaction or
docs quality; docs prove intent, not behavior. State these limits once in
coverage. The caller/coordinator routes code/docs or docs/docs disagreements
with scope/artifact references to [`review-consistency`](../review-consistency/SKILL.md),
never as a substitute for mandatory filtering.

## Procedure

1. **Resolve scope.** Review Lens dispatches this pass every run. Proven **no
   Rust library scope** in the coordinator's factual package inventory permits
   `not-applicable` with provenance, without Cargo. Unknown selection or missing
   required extraction is `blocked`. Every Rust-library change, including small,
   docs-only and clean reviews, requires this procedure and filtering.

   Accept caller package/manifest, revisions, features, target, toolchain and
   artifacts; otherwise use the tool-selected package, `--all-features` and host
   target. Ambiguous selection needs the package name or a blocker, not manifest
   inspection. Before change extraction consume the exact
   [packageComparison](../review-lens/package-comparison.md); resolve `unknown`
   sides/mode through its context owner or block. Head-only audits need no
   baseline record.

   `<feature-args>` defaults to `--all-features`, replaced only by explicit
   `--features`/`--no-default-features` choices. `<scope-args>` contains established
   package/manifest/target options; preserve toolchain through installed help.
   Record effective configuration and inherited surface-affecting build flags.
   Reuse only matching revision/dirty state, package, configuration, toolchain
   and output options.

2. **Establish trust/tools once.** Builds can execute scripts/proc macros.
   Reuse trust and version records; otherwise require trusted provenance or
   isolated credential-free execution, not source inspection.

   ```text
   cargo public-api --version
   ```

   Only if missing, install the official tool with stable:

   ```text
   cargo +stable install cargo-public-api --locked
   ```

   Install a missing compatible nightly, honoring any caller-specified version:

   ```text
   rustup toolchain install nightly --profile minimal
   ```

   Unsafe execution, failed installation/extraction: return `blocked`, exact
   attempted command and decisive reason. Never troubleshoot by reading source.

3. **Capture the full present-side surface once.** Usually head; for proven
   `removed-package`, use exact baseline in an owned disposable worktree, never
   the absent head. Use an external target (`CARGO_TARGET_DIR`) and a documented
   lock-preserving option when supported. Block if reviewed inputs would change;
   external targets alone do not protect lockfiles. Capture verbatim:

   ```text
   cargo public-api --color=never --include function-parameter-names <feature-args> <scope-args> > <full-api-output>
   ```

   Keep the complete capture despite display truncation; inventory paths/families
   privately, not in handoff dumps. Optional readability view:

   ```text
   cargo public-api --color=never --include function-parameter-names <feature-args> -sss <scope-args> > <simplified-api-output>
   ```

   Reuse matching tool artifacts/cache when supported; no mandatory second
   build or JSON parsing here. `-sss` omits blanket, auto-trait and auto-derived
   impls: `Debug`/`Clone`/`Send` absence claims require full output. Retain JSON
   paths/provenance for downstream retrieval without opening them.

4. **Complete every PR/change/semver comparison.** Record mode, exact revisions,
   presence provenance and real capture paths:
   - `added-package`: logical empty base versus full real head, including crate
     module; all emitted items are additions.
   - `removed-package`: full real baseline versus logical empty head; all
     emitted items are removals.
   - `paired`: real counterparts and matching API diff. Failed captures or
     renamed packages never justify empty sides.

   Logical emptiness is metadata, not fabricated output. Never extract/build a
   proven absent package, including through commit-diff commands. For `paired`,
   reuse a matching capture or choose one supported alternative, capturing
   outside the reviewed repository:

   ```text
   cargo public-api --color=never --include function-parameter-names <feature-args> <scope-args> diff latest
   cargo public-api --color=never --include function-parameter-names <feature-args> <scope-args> diff <version>
   cargo public-api --color=never --include function-parameter-names <feature-args> <scope-args> diff <ref1>..<ref2>
   ```

   Record resolved baseline version/revision, not `latest`. Commit diffing
   checks out revisions: use an external disposable worktree, never the caller's
   worktree or `--force`. It is neither a sandbox nor a dirty-head copy. Paired
   dirty reviews require tool-supported comparison against the actual captured
   head, not `HEAD`. One-sided dirty reviews likewise require actual dirty
   captures. Unavailable required comparisons are `blocked`, not full-crate
   fallback audits.

5. **Inventory all emitted families before judging.** Include modules/re-exports,
   types/fields, traits/impls, functions/methods, constants/statics, macros,
   errors/builders/iterators. Standalone audits cover the full current surface;
   small diffs cover changed items/affected signatures, using immediate unchanged
   families only as context. Explain full-surface scope for broad changes;
   separate pre-existing concerns from regressions. Missing diffs prove neither breadth
   nor no change. One-sided modes cover the full real surviving surface.
   Crate-module-only scaffold output is valid, not failure or `not-applicable`.

6. **Draft, then always filter.** Apply lenses to the selected set with decisive
   emitted excerpts; separate conditional questions and omit unprovable claims.
   Run [post-processing](rustdoc-post-processing.md) once with its complete
   handoff, **even for clean/empty reports**. Return only its filtered report;
   unavailable/failed isolation is `blocked`, never permission to release drafts.

## Specialist questions

These are heuristics, not automatic defects. Honor guideline intent/exceptions;
require output-demonstrated consumer or compatibility cost.

### Surface and names

- One clear public path per item (`M-SINGLE-ITEM-PATH`)? Prefer essential root
  entry types and use-case modules. Flag sprawl, fragmented paths, `prelude`,
  `traits` or `errors` buckets only when output demonstrates the problem
  (`M-BALANCED-MODULES`, `M-NO-PRELUDE`).
- Foreign re-exports/dependency signatures commit semver. Prefer defining-crate
  types unless interoperability or umbrella roles justify exposure
  (`M-FOREIGN-REEXPORTS`, `M-DONT-LEAK-TYPES`). Question demonstrably redundant
  scaffolding, aliases, raw options and parallel APIs; do not infer accident.
- Rust casing: `snake_case` functions/modules, `UpperCamelCase` types/traits,
  `SCREAMING_SNAKE_CASE` constants. Prefer precise vocabulary without empty
  `Manager`/`Helper`/`Util`/`Common`/`Data` padding (`M-WEASEL-WORDS`,
  `M-SHORT-NAMES`).
- `as_` borrows, `to_` converts with possible copy/allocation, `into_` consumes.
  Prefer standard `From`/`TryFrom`/`AsRef`/`AsMut`. Getters use `value()`, not
  `get_value()`; collections use `iter`/`iter_mut`/`into_iter` and matching
  iterator names.
- Constructors are inherent associated functions, usually `new`; receiver
  operations are methods, unrelated computation free functions (`M-REGULAR-FN`).
  Align sibling verbs, word order, receivers and conceptual parameter order:
  important inputs first, ubiquitous context/closures last
  (`M-PARAMETER-CONSISTENCY`).

### Ergonomics and types

- Verify every public type's `Debug` in **full** output (`M-PUBLIC-DEBUG`).
  Readable types, especially errors/string wrappers, need `Display`
  (`M-PUBLIC-DISPLAY`). Consider meaningful `Clone`, `Copy`, `Default`,
  equality/order/hash, conversions, `Borrow` and formatting traits.
- Heavy service handles usually need shared-ownership `Clone`, but output cannot
  prove clone cost (`M-SERVICES-CLONE`). Full-output `Send`/`Sync` absence is a
  concern only when the apparent role implies cross-thread use (`M-TYPES-SEND`).
- Collections: `iter`, `iter_mut`, iterator types, owned/shared/mutable
  `IntoIterator`, `FromIterator`, `Extend` (`M-COLLECTION-TRAITS`).
- Where visibly feasible, accept `impl AsRef<str/Path/[u8]>`,
  `impl RangeBounds<_>` and generic `Read`/`Write` without infecting stored types
  (`M-IMPL-ASREF`, `M-IMPL-RANGEBOUNDS`, `M-IMPL-IO`).
- Make lifetimes/ownership legible: borrow read-only values, consume owning
  conversions/builders; avoid needless caller `clone`, `String`, `Vec`,
  smart-pointer or lifetime requirements. Prefer `async fn` over `impl Future`
  when viable, allowing trait/performance-sensitive exceptions (`M-ASYNC-FN`).
- Prefer strong domain/std types to ambiguous strings, booleans, tuples or
  primitive clusters where names/families distinguish them (`M-STRONG-TYPES`);
  do not invent unrendered invariants.
- Avoid incidental `Arc`/`Rc`/`Box`/`Pin`, lock/borrow wrappers and deeply nested
  generics unless fundamental or consumer-justified (`M-AVOID-WRAPPERS`,
  `M-SIMPLE-ABSTRACTIONS`). Escalate service dependencies concrete -> generic ->
  `dyn Trait` only as needed; minimize bounds (`M-DI-HIERARCHY`).
- Essential behavior belongs inherently, not solely in extension traits
  (`M-ESSENTIAL-FN-INHERENT`). Require object usability only for clearly dynamic
  extension points, not every trait.

### Construction, errors and evolution

- Simple types need discoverable constructors/defaults, `Result`/`Option` for
  fallibility/absence. For many configuration permutations prefer
  `Type::builder()`, `TypeBuilder`, field-named chainable setters and `build()`;
  no public `TypeBuilder::new()` (`M-INIT-BUILDER`).
- Setters should not return `Result`; required cross-field validation belongs
  in fallible final `build()` (`M-BUILD-RESULT`). Output cannot prove validation
  is required.
- Prefer situation-specific error structs with `Debug`, `Display`,
  `std::error::Error`, standard `From` conversions and focused classification/
  accessors, not global catch-all enums (`M-ERRORS-CANONICAL-STRUCTS`,
  `M-FROM-ERROR`).
- Public fields commit representation: prefer private fields/accessors unless
  literal construction is intended. Assess rendered `#[non_exhaustive]` on
  growing structs/enums and resulting construction/matching ergonomics.
- Visibly downstream-implementable traits commit evolution. Missing private
  sealing machinery in output does **not** prove a trait unsealed.
- Question dependency types, complex bounds, associated types and concrete
  returns unnecessarily fixing implementation choices, balanced against callers'
  need to name, store and compose values.

### Features and comparisons

- Compare only separately emitted configurations. Default-feature output says
  nothing about optional surfaces; all-features output proves no individual
  combination's additivity.
- Assess additions/removals/signatures only within the selected review set;
  current listings alone cannot establish semver impact.
- Inventory visible macro names/signatures only; syntax, expansion hygiene,
  generated bounds and behavior remain out of scope.

## Proof and output

Use exact public paths and smallest decisive excerpts, including related emitted
lines needed to establish absence. Explain concrete usability, interoperability,
type-identity or compatibility cost and a specific better public shape, not an
implementation patch; cite useful `M-*` IDs/conventions.

Cleanliness is usually `Non-blocking`/`Nit`; blocking requires substantial,
demonstrated consumer/compatibility impact. Context-dependent alternatives are
conditional **Design questions**, not defects or simplified-output guesses.

Choose area or standalone role before drafting and preserve it through filtering.
Area results use the shared findings/coverage contract. Standalone outline
(omit `Verdict:` on the requester's own PR):

```text
**Posted by an AI agent**

# Public API review: <package>

Scope: <tool version, package, features, target, and optional baseline>
Verdict: <approve | approve with non-blocking comments | changes requested | blocked>

## Findings
<Shared finding blocks, exact public paths and API excerpts; impact order.>

## Design questions
<Conditional Design notes using the findings contract.>

## What is already clean
<Specific strengths, no generic praise.>

## Coverage and limitations
<Families/configurations, rustdoc filtering coverage and evidence limits.>
```

Distinguish excerpts with code formatting. Explicitly report no findings with
covered families/configurations when clean. Extraction failure returns
`blocked` and decisive diagnostics, not an API verdict. The coordinator owns
combined presentation/delivery; this worker returns only the filtered report.

[pragmatic-rust]: https://microsoft.github.io/rust-guidelines/
