---
name: review-api-design
description: >
  Review a Rust change's public contract from the diff and source: visibility and
  layering, semver cascade, constructors, builders and defaults, strong types and
  enums, trait implementability and sealing, macros as public API, and error type
  and panic conventions. Inventories every changed export before judging it. Use
  when reviewing what a change commits downstream consumers to, or when
  review-lens routes changed public surface here. Complements review-public-api,
  which audits an existing surface from cargo public-api output without reading
  source. Not for correctness defects or naming style.
---

# Review API Design

For a library change, most design attention belongs on what downstream consumers
can construct, implement, match, store and depend on across releases. Public
surface is cheap now and breaking once the crate ships it.

This skill reads the diff, source, manifests, callers and docs. Its sibling
`review-public-api` deliberately reads only `cargo public-api` output; use that
one to audit an existing surface, this one to review a change.

## The gate: inventory before judging

Build a private inventory of every changed exported item and re-export, trait
bound, public enum variant, feature flag, dependency type in a signature,
serialization format and observable default. For each entry record one
disposition: intentional public contract, should be narrower, needs an evolution
guard, or needs more investigation.

Finish the inventory even after finding the first issue. For a crate
publication, repository move or first crates.io release, treat the entire
reachable surface as new even if the code was copied unchanged. A correctness
defect that happens to touch a public method does **not** satisfy this pass.

## Lenses

- **"Does it need to be public?"** For every new `pub`, ask whether a downstream
  user constructs, calls or matches it. Prefer the narrowest visibility that
  works. A private type in a public signature, or a `pub` item reachable only via
  an unexported path, is a layering bug — decide the intended boundary, then
  export the type properly *or* narrow the item; do not `#[allow(unreachable_pub)]`
  it.
- **Dependency-in-signature.** A public signature exposing an implementation-only
  type — a channel, an SDK type, a boxed future — makes it your semver surface.
  Prefer `async fn -> T` over handing back a channel.
- **Macros are public APIs.** For exported attributes and derives, inventory the
  accepted syntax and field types, generated trait bounds and paths, default
  keys, diagnostics, hygiene and expansion-time feature assumptions. Describe
  defects in terms of the consumer's valid input or generated contract, not the
  proc-macro internals that caused them. Plumbing that is public only so
  generated code can reach it belongs under a documented `#[doc(hidden)]`
  surface.
- **Strong types.** Push primitives to newtypes at the boundary (`BaseUri`,
  `Tenant(Uuid)`). A newtype that encodes an invariant must enforce it at
  construction, fallibly, rather than leaving every caller to re-check.
- **Enum test.** Question a public enum whose variants encode *how the value was
  built* rather than a domain contract the consumer matches on; if it is not
  matched, expose a convenience method and keep it internal. Prefer
  `#[non_exhaustive]` on public enums that may grow, where repo policy allows.
- **Builders and constructors.** Setter methods and consuming `mut self -> Self`
  over a raw options bag; typed inputs; follow the surrounding constructor family
  for names. A builder is the right answer when required parameters vary by
  generic shape and enumerating constructors would multiply them. A headline
  single-entry generic constructor can still be the deliberate design, so weigh
  ergonomics against the niche coercion case before splitting it.
- **Safe, coherent defaults.** A convenient, safe default; question defaults that
  diverge between `new` and `Default`, or between siblings, for no reason.
  Prefer `const` defaults over functions returning constants.
- **Panic test.** A public constructor accepting a runtime collection or external
  configuration should return a typed error for invalid combinations rather than
  panic, especially in foundational infrastructure that promises not to crash the
  application.
- **Privacy escape hatches.** Any API that bypasses validation, classification or
  redaction must make that bypass unmistakable in its name and type constraints.
  Do not accept a broad `Into<Value>` while documenting only safe primitive
  inputs. Test-only surface belongs behind a test feature.
- **`Send`/`Sync` and `Clone`** for foundational and service types; `!Send` is
  infectious. For deliberately thread-affine guards, enforce `!Send` in the
  returned public type rather than in documentation.
- **Opaque returns still have contracts.** Check whether `impl Trait` returns
  preserve the ownership, lifetime, cancellation and thread-safety properties
  callers need, and whether callers need to name or store the type.
- **Trait implementability is a commitment.** For traits normally generated by a
  derive or attribute macro, decide whether manual downstream implementations are
  an intended extension point. Seal the trait when they are not; when they are,
  minimize required methods and provide defaults so the trait can evolve.
- **Derives are additive.** Adding a derive later is not breaking, so question
  derives added without a demonstrated need; removing one is breaking.

## Errors

- **Canonical error at a public or foundational boundary** — one error struct
  with `is_*` accessors over a zoo of error types buys semver room to add cases
  later (cf. jiff issue #8); relax this for internal-only crates. Prefer
  returning a crate-native error.
- Classify error kinds with a typed enum rather than matching formatted text.
- Error and `expect(..)` messages are lowercase with no leading capital; say
  "error", not "exception".
- Make errors recoverable-aware where the repo has a recovery convention.

## Semver

If crate A's types appear in crate B's public API, a breaking A is breaking for
B — trace it and name the crates. Dev-dependency bumps are usually invisible to
consumers. Verify the actual break where you can, and require the repository's
versioning and release treatment for an intentional break.

## Findings

Report in impact order with the code anchor, the consumer-visible consequence,
and a concrete fix. Design findings may be argued precisely from the code,
manifests and repository rules without executing anything; verify a claimed break
where it is cheap. Always state what public surface was reviewed, even when it
produced no finding.

When invoked directly rather than through `review-lens`, first read the
repository's own rules from the base revision and treat green CI as the
baseline; `review-lens` carries the workspace-specific adaptation.

Post through the `review-delivery` skill when the review targets a PR.
