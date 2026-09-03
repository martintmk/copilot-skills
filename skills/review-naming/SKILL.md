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

A divergence from what already exists is itself a finding. Names are the API
consumers read first, and an unnecessary abstraction is a permanent tax.

## Alignment with what exists

- **Match the family.** Names, defaults, feature-flag names, constants and API
  shapes should match their siblings. State the convention when raising it:
  "family convention: `Iso8601` gets `display_iso_8601`, so `EcmaScript` should
  get `display_ecma_script`."
- **Align across crates.** When another crate in the workspace already names a
  concept, reuse that name rather than inventing a synonym — there is no reason
  for two names for one concept. Mirror the upstream method names when wrapping
  another crate's configuration.
- **Concise names.** Drop padding words that add no meaning (`…Metadata`,
  `…Aware`, `…Helper`, `…Manager`) when a shorter name is exact. Prefer the noun
  the domain already uses.
- **No unit in the name when the type carries it.** `initial_backoff: Duration`,
  not `initial_backoff_ms`. Likewise, do not prefix a field with the subsystem
  its siblings omit.
- **Name for capability versus action.** A trait that only reports a property
  should read as the property, and its method should match the type it returns.
- **Terminology precision.** Reject a term that names a different thing than the
  concept ("circuit" for a circuit breaker), and prefer the everyday term the
  domain actually uses.
- **A rename is a change to more than the symbol.** Update user-facing
  references, docs, examples, feature names and the PR title with it.

## Unnecessary abstraction

- **Would a change to an existing type remove the need?** Adding `Clone` or a
  method to an existing type usually beats a new trait, wrapper or layer.
- **Is it hand-rolling std or a derive?** Prefer the derive or std method.
- **An internal helper that never touches `self` should be a free function.**
- **Prefer one internal type plus a small public API** over per-variant
  boilerplate; expose a method such as `should_promote(..)` rather than leaking
  an enum consumers never match on.
- **Do not wrap a foundational type** just to add a layer; at some point core
  types should be used directly to avoid needless conversion and breaking
  changes.
- **Two constructors that differ only by boxing** can usually collapse into one
  that boxes internally, when the call is not on the hot path.

## Judgement

Naming and abstraction findings are argued from the code and the surrounding
family, not executed. Quote the sibling that establishes the convention — that
citation is the evidence. Where the existing code is deliberately inconsistent,
say so and recommend the smaller change.

These are usually `nit:` or `non-blocking:` unless the name ships in a public API
that is about to be released, in which case it is a contract decision and belongs
with the API design review.

## Findings

Report with the code anchor, the sibling or convention it diverges from, and the
exact replacement name or simplification. Prefer a `suggestion` block when the
fix is a self-contained rename on the anchored line. Do not flag names that are
merely different from your own preference.

When invoked directly rather than through `review-lens`, first read the
repository's own rules from the base revision and treat green CI as the
baseline; `review-lens` carries the workspace-specific adaptation.

Post through the `review-delivery` skill when the review targets a PR.
