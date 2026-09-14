---
name: review-api-design
description: >
  Review a Rust change's public contract from the diff and source: visibility and
  layering, semver cascade, constructors, builders and defaults, strong types and
  enums, trait implementability and sealing, macros as public API, and error
  type, message and panic conventions, including internal errors. Also owns
  dependency and feature checks. Use for downstream contracts, manifest or
  error-policy changes, or review-lens's public-surface pass.
  Complements review-public-api, which audits an existing surface from cargo
  public-api output without reading source. Not for runtime defects or naming
  style.
---

# Review API Design

Review what downstream consumers can construct, implement, match, store and
depend on. Unlike `review-public-api`'s output-only audit of an existing surface,
this pass reads the diff, source, manifests, callers and docs.

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md); reuse supplied context.

Own public contracts, dependencies/features and error type, message and panic
conventions, even for internal errors. Runtime defects belong to `review-correctness`,
recovery classification to `review-resilience`, family-only naming to
`review-naming`, and emitted signal contracts to `review-telemetry`.
Hand standalone code/docs or docs/docs disagreements to `review-consistency`;
use its evidence without replacing this pass's public-contract inventory.

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
  over a raw options bag; typed inputs; use the constructor family as design
  evidence and leave naming-only mismatches to `review-naming`. A builder is the
  right answer when required parameters vary by generic shape and enumerating
  constructors would multiply them. A headline single-entry generic constructor
  can still be the deliberate design, so weigh
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
- **Derives need a consumer use.** Question derives without a demonstrated need.
  Removing a public trait implementation can break consumers; check coherence
  and inference effects before declaring an added implementation non-breaking.

## Errors, including internal errors

- **Canonical error at a public or foundational boundary** — one error struct
  with `is_*` accessors over a zoo of error types buys semver room to add cases
  later; relax this for internal-only crates. Prefer
  returning a crate-native error.
- Classify error kinds with a typed enum rather than matching formatted text.
- Error and `expect(..)` messages are lowercase with no leading capital; say
  "error", not "exception".
- Identify affected error boundaries for `review-resilience` when a change may
  lose or alter the repository's recovery information.

## Dependencies, features and semver

- Question new dependencies, especially proc-macro-heavy ones in leaf crates.
  Prefer std, a suitable existing ecosystem crate or an existing repository
  dependency; a trivial implementation can beat a heavy derive dependency.
- Keep features minimal and off by default, with an empty/minimal `default`;
  gate test-only surfaces and fakes behind the repository's test feature.
  Establish the actual enabled features and compile relevant gated modules in
  the configuration that ships.
- Require the minimum dependency version that works; avoid unrelated mass
  bumps and release cascades.

If crate A's types appear in crate B's public API, a breaking A is breaking for
B — trace it and name the crates. Dev-dependency bumps are usually invisible to
consumers. Verify the actual break where you can, and require the repository's
versioning and release treatment for an intentional break.

## Evidence and findings

For actionable findings, use the shared attribution and a concise bold diagnosis
title naming the affected contract. **Problem** identifies the exported item
and failed inventory disposition, dependency/feature decision, or internal error
boundary, with decisive evidence. **Why this matters** states the concrete
consumer or maintainer consequence. **Suggested fix** specifies the corrected
contract, dependency/feature choice or error policy. Discuss compatibility only
when it establishes that consequence or changes the fix.

Design findings may be argued precisely from the code, manifests and repository
rules without executing anything; verify a claimed break where that is cheap.

Coverage line: the public surface reviewed — crates, modules or API families —
stated even when the gate produced no finding, plus dependency/feature decisions
and internal error conventions reviewed.
