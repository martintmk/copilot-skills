---
name: review-naming
description: >
  Review Rust changes for names and shapes that diverge from their siblings, and
  for abstractions that earn nothing. Covers family and convention alignment,
  concise non-padded names, units already carried by a type, domain-appropriate
  terminology, and whether a new trait, wrapper or layer could be replaced by a
  small change to an existing type. Use for a focused naming or abstraction
  audit, or when review-lens routes new names here. Not for formatting, import
  order, or public contract and semver review.
---

# Review Naming

Find unexplained family divergence and abstractions that earn nothing, not
personal preferences. Follow [shared context](../review-lens/review-context.md)
and the [findings contract](../review-delivery/findings-contract.md).

`review-api-design` owns public contracts/defaults, `review-telemetry` signal
names, and `review-perf` measured cost. Supply family evidence to those owners
without a naming-only duplicate.

## Procedure

1. Establish sibling and workspace conventions around changed names/shapes.
2. Apply the questions below; choose concrete renames or smaller abstractions.
3. Prove divergence from code/family evidence, not execution or preference.
   State deliberate existing inconsistency and recommend the smaller change.

## Naming questions

- Do names, defaults, feature flags, constants and API shapes match siblings?
  State the convention: `Iso8601` has `display_iso_8601`, so `EcmaScript` should
  have `display_ecma_script`.
- Reuse workspace names for the same concept. When wrapping configuration,
  mirror upstream method names rather than inventing synonyms.
- Remove meaningless padding (`Metadata`, `Aware`, `Helper`, `Manager`) when a
  shorter domain noun is exact. Use everyday, precise terminology: "circuit"
  names something different from a circuit breaker.
- Does the type already carry units? Use `initial_backoff: Duration`, not
  `initial_backoff_ms`; omit subsystem prefixes that siblings omit.
- Property-reporting traits should name the property, not an action; methods
  should match their return types.
- Include affected user-facing references, docs, examples, feature names and
  the PR title in rename recommendations.

## Abstraction questions

- Would `Clone` or a method on an existing type eliminate a trait/wrapper/layer?
  Prefer that small change; replace hand-rolled std/derive behavior.
- Make an internal helper that never touches `self` a free function.
- Can one internal type and a small public API replace per-variant boilerplate?
  Prefer `should_promote(..)` to exposing an unmatched enum; route the exposure
  decision to `review-api-design`.
- Use foundational types directly instead of wrappers causing needless
  conversions and breaking changes.
- Off the hot path, constructors differing only by boxing can usually collapse
  into one that boxes internally.

## Proof and coverage

Quote the sibling establishing a convention and identify the conflict; for
abstractions, show the unnecessary layer and concrete removal. Explain confusion
or maintenance cost and specify the exact replacement. Use a `suggestion` under
the shared fix section for self-contained renames on the anchored line.

Usually `Nit` or `Non-blocking`. Names about to ship publicly are contract
decisions for API design review.

Coverage: names/abstractions reviewed and conventions not established.
