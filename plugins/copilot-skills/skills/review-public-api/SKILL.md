---
name: review-public-api
description: >
  Audit a Rust library's exported contract using cargo-public-api output only,
  then check provisional claims against the items' rustdoc. Use for a
  whole-crate or explicit output-only API audit, or review-lens's public API
  surface area; small PRs cover changed public items and their immediate
  family. Applies idiomatic Rust API practices and API-visible Pragmatic Rust
  Guidelines. Not for source-based findings, implementation correctness, docs
  quality/consistency, performance or posting.
---

# Review Public API

Judge a library's public surface the way a consumer sees it: from
`cargo public-api` output alone. Apply idiomatic Rust conventions and the
API-visible [Pragmatic Rust Guidelines][pragmatic-rust].

Reading only the output keeps the review honest about what users actually get.
It also sets limits: output cannot prove correctness, validation, panics,
soundness, performance, redaction or docs quality. State these limits once in
coverage.

## Evidence boundary

- **Draft from output only.** Do not open source, manifests, lockfiles, build
  scripts, tests, examples, diffs, history or rendered docs while drafting. Do
  not use `cargo metadata`, rust-analyzer or code search to find claims.
- **Docs can only remove or narrow claims.** After the draft, read the
  rustdoc of the items you criticize. Docs can show a choice was deliberate.
  They never add findings, raise severity or prove runtime behavior.
- **Report only.** Never edit, post or vote.

## Review rules

1. **Use complete, matching output.** The public surface must match the
   selected package, revision, features, target and toolchain. A simplified
   `-sss` listing hides blanket, auto-trait and derived impls. Claims that
   `Debug`, `Clone` or `Send` is missing need full output.
2. **Compare the right surfaces.** A change review needs both base and head,
   not an unrelated published version or a current-only audit. Match renamed
   or moved packages using the supplied package facts.

   | Package | Compare |
   | --- | --- |
   | exists at base and head | both real surfaces |
   | new at head | an empty base with the full head surface; every item is new |
   | removed at head | the full base surface with an empty head; every item is removed |

   An empty side needs proof of package absence, never a failed build. Local
   changes need output from the actual working tree, not just `HEAD`.
   Missing required output leaves that surface unreviewed; request the
   missing evidence from the caller rather than changing scope or reading
   source.
3. **List every emitted family before judging:** modules and re-exports, types
   and fields, traits and impls, functions and methods, constants and statics,
   macros, errors, builders and iterators. A standalone audit covers the whole
   surface. A change review covers changed items and signatures, using
   unchanged neighbors only as context. Separate existing concerns from new
   ones. A crate that emits only its root module is valid output.
4. **Draft, then check against docs.** Draft findings with the smallest
   decisive output excerpts. Request matching rustdoc text for each item the
   draft depends on, its owner and trait, and applicable module or crate docs.
   Check every claim, including design questions and claims that an API is
   already suitable.

   | Docs show | Action |
   | --- | --- |
   | nothing relevant | Keep the claim. |
   | the claim's premise is wrong, or the choice is a documented deliberate exception | Remove the claim. |
   | the answer to a design question | Remove the question. |
   | part of the claim is wrong | Keep only the part the output still proves. |
   | conflicting or missing docs | Keep the claim; note the gap in coverage. |

   Examples: facade docs can defeat "accidental foreign re-export"; documented
   thread-local intent can refute an assumed `Send` requirement. Rewrite
   narrowed titles and sections to match what survives. If the draft has claims
   and the required docs could not be retrieved, report that limit; never
   return an unchecked draft. This differs from successfully retrieved docs
   with no explanation. An empty draft needs no docs.

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

## Evidence and output

Use exact public paths and the smallest decisive excerpts, including the lines
that show something is absent. Explain the concrete cost to users, interop,
type identity or compatibility, and a better public shape. Cite useful `M-*`
guideline IDs.

Findings about tidiness are usually `Non-blocking` or `Nit`. Blocking needs
substantial, demonstrated impact on users or compatibility. Alternatives that
depend on context are conditional design questions.

Write each finding in the
[findings contract](../review-delivery/findings-contract.md), with exact public
paths instead of source lines. Coverage names the families and configurations
reviewed, docs checked, and evidence limits. Say explicitly when there are no
findings; missing captures or unfinished doc checks are not a clean result.

[pragmatic-rust]: https://microsoft.github.io/rust-guidelines/
