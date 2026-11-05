---
name: review-api-design
description: >
  Review Rust public contracts from source and diffs: construction, visibility,
  traits, macros, features, dependencies, errors and compatibility. Use for
  contract or error-policy changes and Review Lens. Not for runtime defects,
  naming-only concerns or output-only API audits.
---

# Review API Design

Check what downstream users can construct, call, implement, match, store and
depend on. Read the diff, source, manifests, callers and docs. Also check
dependencies, features and error and panic conventions, including internal
errors.

Leave runtime defects, retry and recovery behavior, naming-only concerns,
telemetry and code/docs disagreements to their own reviews.

## Procedure

1. **List before judging.** Record every changed export and re-export, trait
   bound, public enum variant, feature, dependency type in a signature,
   serialization format and observable default. For each, decide: intended
   public contract, should be narrower, needs a guard for future change, or
   needs investigation.
2. **Finish the list** even after finding an issue. When a crate is first
   published or moved to a new repository, treat its whole reachable surface as
   new, including unchanged copied code.
3. **Ask the questions below.** Check which features are enabled and read gated
   modules in the shipping configuration. Verify cheap compatibility claims.

## Public-contract questions

- **Visibility and layering:** Does each new `pub` item have a downstream
  constructor, caller or matcher? Use the narrowest visibility that works. For
  private types in public signatures, export them properly or narrow the item;
  never hide the problem with `#[allow(unreachable_pub)]`.
- **Dependency exposure:** Channels, SDK types and boxed futures in signatures
  become semver commitments. Prefer `async fn -> T` over returning a channel.
- **Macros:** Check exported attribute and derive syntax, accepted field types,
  generated bounds and paths, default keys, diagnostics, hygiene and feature
  assumptions at expansion time. Describe what consumers can write and what is
  generated, not proc-macro internals. Public plumbing for generated code
  belongs on a documented `#[doc(hidden)]` surface.
- **Strong types:** Prefer boundary newtypes (`BaseUri`, `Tenant(Uuid)`) over
  primitives. Enforce their invariants with fallible construction.
- **Enums:** Do variants express a contract users match on, or only how values
  were built? Keep enums nobody matches internal behind methods. Prefer
  `#[non_exhaustive]` for public enums that will grow, where policy allows.
- **Construction:** Prefer typed setters and consuming `mut self -> Self` over
  raw option bags. Follow sibling constructor families. Builders avoid many
  constructors when required parameters vary by generic shape. Weigh a
  deliberate single generic constructor's ergonomics before splitting it.
- **Defaults and panics:** Require convenient, safe defaults. Question an
  unexplained `new` or `Default`, or one that differs from siblings. Prefer
  `const` defaults to functions returning constants. Invalid runtime
  configuration should return a typed error, not panic, especially in
  foundation crates.
- **Escape hatches:** Make bypasses of validation, classification or redaction
  explicit in names and types. A broad `Into<Value>` contradicts a
  safe-primitive restriction. Gate test-only surfaces behind the test feature.
- **Threads and ownership:** Check `Send`, `Sync` and `Clone` on foundation and
  service types. A thread-affine guard must enforce `!Send` in its returned
  types, not in prose. For `impl Trait` returns, check ownership, lifetimes,
  cancellation, thread safety and whether callers need to name or store the
  type.
- **Traits and derives:** Are manual implementations of normally generated
  traits intended as an extension point? Seal the trait if not. Otherwise keep
  requirements small and give defaults so it can evolve. Each derive needs a
  consumer use. Do not recommend `Copy` without a required consumer contract.
  Removing public impls can break users; check coherence and
  inference before calling an addition non-breaking.

## Errors, dependencies and evolution

- Follow the crate's established error taxonomy. Prefer a struct with focused
  classification accessors for each coherent error family, not a catch-all enum.
  Classify with typed values, not formatted text.
- Error and `expect(..)` messages are lowercase and say "error", not
  "exception".
- Challenge new dependencies, especially heavy proc-macro leaves. Prefer std,
  an existing workspace crate or a few lines of code.
- Keep features minimal and off by default with an empty or small `default`.
  Gate fakes and test surfaces. Require the lowest working dependency version,
  not unrelated mass bumps.
- If crate A's types appear in crate B's public API, breaking A breaks B. Name
  the affected crates. Dev-dependency bumps are usually invisible to users.
  Intentional breaks need the repository's versioning and release steps.

## Evidence

Code, manifests and repository rules are enough to prove design findings. Name
the exported item and what is wrong with it, give the decisive evidence, the
consequence for users or maintainers, and the corrected contract. Discuss
compatibility only when it shows impact or changes the fix.

## Report

Write each finding in the
[findings contract](../review-delivery/findings-contract.md).
Coverage names the crates, modules and API families checked, dependency and
feature decisions, and error conventions reviewed, even with no findings.
