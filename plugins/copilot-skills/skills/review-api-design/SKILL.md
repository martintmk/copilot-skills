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

Audit downstream construction, calls, implementations, matching, storage and
dependencies using diff, source, manifests, callers and docs, unlike the
output-only `review-public-api`.

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md). Own public
contracts, dependencies/features and error/panic conventions, including internal
errors. Route runtime defects to `review-correctness`, recovery to
`review-resilience`, family-only naming to `review-naming`, signals to
`review-telemetry`, and standalone code/docs or docs/docs disagreements to
`review-consistency`. Their evidence never replaces this inventory.

## Procedure

1. **Inventory before judging.** Privately record every changed export/re-export,
   trait bound, public enum variant, feature, dependency type in a signature,
   serialization format and observable default. Assign each: intentional public
   contract, should be narrower, needs an evolution guard, or needs investigation.
2. **Complete the inventory**, even after finding an issue. For crate publication,
   repository moves or first crates.io releases, treat the entire reachable
   surface as new, including unchanged copied code. A public-method runtime bug
   does not satisfy this pass.
3. **Apply the questions below.** Establish enabled features and compile relevant
   gated modules in the shipping configuration. Verify cheap compatibility
   claims; return findings and coverage even when the inventory is clean.

## Public-contract questions

- **Visibility/layering:** Does each new `pub` have a downstream constructor,
  caller or matcher? Use the narrowest working visibility. For private types in
  public signatures or unexported public paths, export properly or narrow the
  item; never hide the boundary error with `#[allow(unreachable_pub)]`.
- **Dependency exposure:** Channels, SDK types and boxed futures in signatures
  become semver commitments. Prefer `async fn -> T` over returning a channel.
- **Macros:** Inventory exported attribute/derive syntax, accepted field types,
  generated bounds/paths, default keys, diagnostics, hygiene and expansion-time
  feature assumptions. Describe consumer-valid inputs or generated contracts,
  not proc-macro internals. Public generated-code plumbing belongs on a documented
  `#[doc(hidden)]` surface.
- **Strong types:** Prefer boundary newtypes (`BaseUri`, `Tenant(Uuid)`) over
  primitives; enforce their invariants fallibly at construction.
- **Enums:** Do variants express a matched domain contract or merely construction
  history? Keep unmatched enums internal behind convenience methods. Prefer
  `#[non_exhaustive]` for growing public enums where policy allows.
- **Construction:** Prefer typed setters and consuming `mut self -> Self` to raw
  options bags. Use constructor-family evidence. Builders avoid multiplying
  constructors when required parameters vary by generic shape, but weigh a
  deliberate single-entry generic constructor's ergonomics against niche
  coercion needs before splitting it.
- **Defaults/panics:** Require convenient, safe defaults; question unexplained
  `new`/`Default` or sibling divergence. Prefer `const` defaults to
  constant-returning functions. Invalid runtime collections/configuration should
  yield typed constructor errors, not panics, especially in no-crash foundations.
- **Escape hatches:** Make validation/classification/redaction bypasses explicit
  in names and type constraints. Broad `Into<Value>` contradicts a safe-primitive
  restriction. Gate test-only surfaces behind the test feature.
- **Threading/ownership:** Check foundational/service `Send`, `Sync` and `Clone`;
  infectious `!Send` must be enforced in thread-affine guards' returned types,
  not prose. For `impl Trait`, check ownership, lifetimes, cancellation,
  thread-safety and callers' need to name/store the return type.
- **Traits/derives:** Are manual implementations of normally generated traits an
  intended extension? Seal if not; otherwise minimize requirements and provide
  defaults for evolution. Require a consumer use for derives. Removing public
  impls can break consumers; check coherence/inference before calling additions
  non-breaking.

## Errors, dependencies and evolution

- At public/foundational boundaries prefer a crate-native canonical error struct
  with `is_*` accessors over many error types, leaving room for cases; relax for
  internal-only crates. Classify with typed enums, not formatted text.
- Error and `expect(..)` messages are lowercase, without leading capitals; say
  "error", not "exception". Identify recovery-information changes for
  `review-resilience`.
- Challenge new dependencies, especially proc-macro-heavy leaf dependencies.
  Prefer std, suitable existing ecosystem/repository crates, or trivial code
  over heavy derives.
- Keep features minimal, off by default with empty/minimal `default`; gate fakes
  and test surfaces. Require the minimum working dependency version, not
  unrelated mass bumps.
- If A's types appear in B's public API, breaking A breaks B: trace and name the
  crates. Dev-dependency bumps are usually consumer-invisible. Verify actual
  breaks where possible; require repository versioning/release treatment for
  intentional breaks.

## Proof and coverage

Static code, manifests and repository rules can prove design findings without
execution. Identify the exported item and failed inventory disposition,
dependency/feature decision or internal error boundary; give decisive evidence,
concrete consumer/maintainer consequence and corrected contract or policy.
Discuss compatibility only when it establishes impact or changes the fix.

Coverage: crates/modules/API families inventoried, dependency/feature decisions
and internal error conventions reviewed, even when no findings result.
