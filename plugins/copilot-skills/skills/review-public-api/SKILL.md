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

Before auditing, apply the [fresh-worker entry gate](../review-lens/worker-isolation.md).
An already assigned output-only worker runs here without dispatching itself again.

Review Lens dispatches this skill on every run. A coordinator-supplied factual
package inventory establishing **no Rust library scope** permits the assigned
worker to return `not-applicable` with that provenance, without running Cargo.
An unknown package selection or missing extraction is `blocked`. For a Rust
library scope, retain the full procedure and mandatory filtering even when
the change is small, docs-only or produces no API findings.

Review the public contract of a Rust library from the consumer's perspective.
`cargo public-api` output is the sole source for generating candidate findings.
Judge only what the output makes observable, applying idiomatic Rust API
conventions and the API-relevant [Pragmatic Rust Guidelines][pragmatic-rust].
Before returning the review, a separate agent uses generated rustdoc JSON only
to remove or narrow candidate statements that the API docs refute or answer.

## Hard boundary: output-only review, docs-only filtering

- Generate candidates only from exact `cargo public-api` output. Do not open or
  search Rust source, manifests/lockfiles, build scripts, docs, tests, examples,
  source diffs, generated rustdoc JSON, or repository history. Do not supplement
  evidence with `cargo metadata`, rust-analyzer, code search, or rendered docs.
- User-supplied scope and use cases can orient the review. Tool help, versions,
  diagnostics, revision identifiers and artifact metadata can guide execution
  and matching, but cannot prove an API-quality claim.
- The mandatory, isolated [post-processor](rustdoc-post-processing.md) is the
  only documentation exception. It uses `review-public-docs` to retrieve docs
  and may only remove or narrow claims, never add/strengthen them, increase
  severity, or establish runtime proof. Keep JSON and full docs bundles out of
  this agent. Returned filtering provenance is not evidence for new claims.
- This is **REPORT-ONLY**: never edit, post, vote, or invoke `review-delivery`
  for delivery. Do not change the reviewed worktree; keep captures and build
  targets outside it, and clean up only owned artifacts after consumers finish.

Read the compact [findings contract](../review-delivery/findings-contract.md)
for presentation. Reuse the applicable execution-trust, handoff/reuse,
configuration and cleanup rules in
[shared context](../review-lens/review-context.md); do not invoke `review-lens`
or repeat coordinator setup. Its source/diff, CI, repository-rule inspection
and executable-reproduction prerequisites do **not** apply here. Exact public
paths replace source anchors, and this skill's evidence boundary wins.

Output cannot establish implementation correctness, validation, panics,
soundness, allocation/performance, sensitive-data redaction, or documentation
quality. Docs establish documented intent, not implemented behavior. Name these
limits once in coverage rather than turning unknowns into findings.

Code-vs-doc and docs-vs-doc factual/contract disagreements belong to
[`review-consistency`](../review-consistency/SKILL.md), routed by the caller or
coordinator with scope and existing artifact references. That review does not
replace this skill's mandatory, removal/narrowing-only candidate filtering.

## Procedure

1. **Fix scope and reuse matching evidence.** Accept the caller's package,
   manifest, revisions, features, target, toolchain and existing artifact paths.
   Otherwise use the package selected by the tool, `--all-features`, and the
   host target. If package selection is ambiguous, request the package name or
   report the blocker; never inspect the manifest to choose one.

   In the commands below, `<feature-args>` is `--all-features` unless the user
   explicitly selected another configuration, in which case use exactly their
   `--features`/`--no-default-features` choices instead. `<scope-args>` contains
   only the established package/manifest/target options. Preserve the selected
   toolchain using the installed tool's documented option. Record the effective
   configuration, including inherited build flags that affect the surface.
   Reuse captures only when revision/dirty state, package, configuration,
   toolchain and output options match; a nearby configuration is not equivalent.

2. **Establish execution trust and tools once.** `cargo public-api` builds
   rustdoc JSON and may execute build scripts or procedural macros. Use trusted
   provenance or an isolated, credential-free environment, not source
   inspection, to decide whether execution is allowed. Reuse an established
   trust decision and tool-version record rather than probing them per view.

   ```text
   cargo public-api --version
   ```

   Only if missing, install the official tool with a stable toolchain:

   ```text
   cargo +stable install cargo-public-api --locked
   ```

   If a compatible nightly is missing, install the required toolchain (use a
   caller-specified compatible version rather than silently replacing it):

   ```text
   rustup toolchain install nightly --profile minimal
   ```

   If execution is unsafe, installation fails, or extraction fails, stop with
   `blocked`, the exact command when attempted, and the decisive reason. Do not
   troubleshoot by reading the crate.

3. **Capture the complete current surface once.** Use an external build target
   directory (for example via `CARGO_TARGET_DIR`) and the installed tool's
   documented lock-preserving option when supported. Stop if extraction would
   rewrite reviewed inputs; an external target alone does not protect lockfiles.
   Capture output verbatim:

   ```text
   cargo public-api --color=never --include function-parameter-names <feature-args> <scope-args> > <full-api-output>
   ```

   Retain the complete capture even if its display is truncated. Build a compact
   public-path/family inventory from it; do not dump the full inventory or schema
   into handoffs. If a simplified view is needed, use:

   ```text
   cargo public-api --color=never --include function-parameter-names <feature-args> -sss <scope-args> > <simplified-api-output>
   ```

   This is a readability aid, not a mandatory second build: reuse matching tool
   artifacts/cache where supported by installed help. Never parse JSON here to
   create a view. `-sss` omits blanket, auto-trait and auto-derived impls; any
   absence claim about `Debug`, `Clone`, `Send`, etc. requires the full output.
   Retain available JSON artifact paths and provenance for retrieval downstream,
   without opening the JSON.

4. **For every PR/change/semver review, obtain the matching API diff.** Reuse a
   matching capture or choose the appropriate supported form, capturing its
   output outside the reviewed repository:

   ```text
   cargo public-api --color=never --include function-parameter-names <feature-args> <scope-args> diff latest
   cargo public-api --color=never --include function-parameter-names <feature-args> <scope-args> diff <version>
   cargo public-api --color=never --include function-parameter-names <feature-args> <scope-args> diff <ref1>..<ref2>
   ```

   These are alternatives, not three required runs. Record the exact resolved
   baseline version/revision, not just a moving label such as `latest`.
   Commit diffing checks out revisions in place: use a disposable worktree
   outside the reviewed repository, never the caller's worktree or `--force`.
   It is not a sandbox and does not contain uncommitted changes. For a dirty
   head, require a tool-supported comparison of the actual captured head with
   baseline; do not substitute `HEAD`. If the required comparison is unavailable,
   return `blocked` for the requested change review, not a silent full-crate audit.

5. **Inventory, then select the review set.** Inventory all emitted families
   (modules/re-exports, types/fields, traits/impls, functions/methods,
   constants/statics, macros, errors/builders/iterators) before judging.
   Standalone audits cover the full current surface. Small API diffs cover
   changed public items and directly affected signatures; unchanged output is
   context for their immediate family, not a source of unrelated findings.
   A broad change can require full-surface coverage; say so and distinguish
   pre-existing concerns from regressions. An unavailable diff is not evidence
   that a PR is broad or that nothing changed.

6. **Draft and filter.** Apply the lenses below only to the selected set. Quote
   the decisive emitted lines, separate context-dependent design questions,
   and omit claims the output cannot prove. Do not publish the draft. Follow
   [post-processing](rustdoc-post-processing.md) once, with its complete handoff
   and return contract, even when there are no candidate findings. Return only
   the filtered report; an unavailable/failed isolated pass is `blocked`, never
   permission to release the provisional report.

## Review lenses

Apply common idiomatic Rust practices first, with the API-visible Pragmatic Rust
Guidelines as an overlay. The consolidated lenses below are heuristics, not
automatic defects: honor guideline intent and exceptions, and require a
concrete consumer or compatibility cost demonstrated by output.

### 1. Surface and navigation

- Look for one clear public path per item. Duplicate user-facing paths make type
  identity and discovery noisy (`M-SINGLE-ITEM-PATH`).
- Prefer essential entry types at the crate root and coherent, use-case-oriented
  modules. Flag roots with obvious undifferentiated sprawl, deeply fragmented
  paths, `prelude` modules, or generic buckets such as `traits` and `errors` only
  when the emitted surface itself demonstrates the problem
  (`M-BALANCED-MODULES`, `M-NO-PRELUDE`).
- Identify re-exported foreign items and external crate types in signatures.
  Prefer types from their defining crate, and avoid leaking dependency types
  unless interoperability or an umbrella-crate role justifies the commitment
  (`M-FOREIGN-REEXPORTS`, `M-DONT-LEAK-TYPES`).
- Question public scaffolding, aliases, raw option bags, and parallel ways to do
  the same thing when the output demonstrates redundant concepts. Do not call an
  item accidental without evidence of intent.

### 2. Names and idiomatic call shape

- Check Rust casing (`snake_case` functions/modules, `UpperCamelCase` types/
  traits, `SCREAMING_SNAKE_CASE` constants) and established vocabulary. Prefer
  short, precise names over `Manager`, `Helper`, `Util`, `Common`, or `Data`
  when those words add no distinction (`M-WEASEL-WORDS`, `M-SHORT-NAMES`).
- Check conversion names: `as_` borrows, `to_` performs a conversion that may
  allocate or copy, and `into_` consumes. Prefer standard `From`, `TryFrom`,
  `AsRef`, and `AsMut` implementations over ad-hoc equivalents.
- Getters normally use `value()`, not `get_value()`. Collection access follows
  `iter`, `iter_mut`, and `into_iter`; iterator type names should match.
- Constructors are inherent associated functions, usually `new`, while
  operations with a clear receiver are methods. General computation that neither
  constructs nor uses an instance should normally be a free function
  (`M-REGULAR-FN`).
- Compare sibling APIs for consistent verbs, word order, receiver style, and
  conceptual parameter order. Important parameters generally come first,
  ubiquitous context and closures last (`M-PARAMETER-CONSISTENCY`).

### 3. Consumer ergonomics and interoperability

- In the full output, verify that every public type implements `Debug`
  (`M-PUBLIC-DEBUG`). Types intended for reading, especially errors and
  string-like wrappers, should implement `Display` (`M-PUBLIC-DISPLAY`).
- Consider conventional traits where their meaning is evident: `Clone`, `Copy`,
  `Default`, equality/order/hash traits, `From`/`TryFrom`, `AsRef`/`AsMut`,
  `Borrow`, and formatting traits. Heavyweight service handles should usually
  have shared-ownership `Clone` semantics, though the output cannot verify the
  cost of cloning (`M-SERVICES-CLONE`).
- Use the full output to assess `Send` and `Sync`. Report their absence only when
  the type's apparent role makes cross-thread use part of the public expectation;
  auto-trait absence alone does not prove a defect (`M-TYPES-SEND`).
- For custom collections, look for `iter`, `iter_mut`, iterator types,
  `IntoIterator` for owned/shared/mutable forms, `FromIterator`, and `Extend`
  (`M-COLLECTION-TRAITS`).
- Prefer flexible function boundaries where feasibility is visible:
  `impl AsRef<str/Path/[u8]>`, `impl RangeBounds<_>`, and generic `Read`/`Write`
  inputs. Avoid infecting stored public types with those bounds
  (`M-IMPL-ASREF`, `M-IMPL-RANGEBOUNDS`, `M-IMPL-IO`).
- Make ownership and lifetimes legible: borrow for read-only access, consume
  `self` for owning conversions/builders, and avoid needless caller-side
  `clone`, `String`, `Vec`, smart-pointer or lifetime requirements.
- Prefer `async fn` over a directly returned `impl Future` when both are viable;
  traits and performance-sensitive APIs can justify the explicit future
  (`M-ASYNC-FN`).

### 4. Type design and complexity

- Prefer strong domain and standard-library types over ambiguous strings,
  booleans, tuples, and primitive parameter clusters when the parameter names and
  surrounding family make the distinction clear (`M-STRONG-TYPES`). Do not infer
  invariants that are not rendered.
- Avoid exposing `Arc`, `Rc`, `Box`, `Pin`, lock/borrow wrappers, or deeply nested
  parameterized types as incidental API plumbing. They are acceptable when
  fundamental to the abstraction or justified by a clear consumer need
  (`M-AVOID-WRAPPERS`, `M-SIMPLE-ABSTRACTIONS`).
- Prefer concrete types over generics and generics over `dyn Trait` for service
  dependencies, escalating only when the flexibility is needed
  (`M-DI-HIERARCHY`). Keep bounds minimal and comprehensible.
- Essential behavior should be inherent rather than available only through an
  extension trait (`M-ESSENTIAL-FN-INHERENT`).
- Assess trait object usability only when the API clearly presents a trait as a
  dynamic extension point. Otherwise object safety is not automatically a goal.

### 5. Construction, errors, and evolution

- Simple types should have a discoverable constructor/default path; signatures
  should communicate fallibility/absence with `Result`/`Option`. For many
  independent configuration permutations, prefer `Type::builder()` and
  `TypeBuilder`, chainable setters named for their fields, and final `build()`;
  do not add public `TypeBuilder::new()` (`M-INIT-BUILDER`).
- Builder setters should not return `Result`; put cross-field validation in a
  fallible final `build()` when validation is needed (`M-BUILD-RESULT`). Output
  alone cannot establish whether validation is actually required.
- Prefer coherent, situation-specific error structs. Check that error types
  implement `Debug`, `Display`, and `std::error::Error`, use standard `From`
  conversion where visible, and expose focused classification/accessor methods
  rather than a global public catch-all enum
  (`M-ERRORS-CANONICAL-STRUCTS`, `M-FROM-ERROR`).
- Public struct fields commit representation and constrain evolution; prefer
  private fields with constructors/accessors unless literal construction is the
  intended contract. For enums and structs likely to grow, assess rendered
  `#[non_exhaustive]` markers and the resulting construction/matching ergonomics.
- Treat public traits as an evolution commitment when downstream implementation
  is visibly supported. Do not claim that a trait is unsealed merely because
  private sealing machinery is absent from the emitted public output.
- Flag dependency types, complex generic bounds, associated types, and concrete
  return types that unnecessarily lock future implementation choices. Balance
  evolution freedom against callers' need to name, store, and compose values.

### 6. Feature and baseline views

- Compare only configurations actually emitted by separate tool runs. A default
  run says nothing about optional feature surfaces, which is why this skill uses
  `--all-features` by default. An all-features run says nothing about whether
  every individual feature combination is additive.
- Assess additions, removals and changed signatures within the review set chosen
  in the procedure. Never infer semver impact from a current-surface listing alone.
- Macro names and signatures visible in the output may be inventoried, but macro
  syntax, expansion hygiene, generated bounds, and behavior are out of scope.

## Evidence and severity

Within the shared finding shape, **Why this matters** must carry the exact public
path, smallest decisive `cargo public-api` excerpt, and concrete usability,
interoperability, type-identity or compatibility cost. Include related emitted
lines when an absence would otherwise be ambiguous. **Suggested fix** names a
specific better public shape, not an implementation patch; cite the applicable
`M-*` ID or Rust convention briefly where useful.

Most cleanliness concerns are `Non-blocking` or `Nit`; only a demonstrated,
substantial consumer or compatibility problem merits blocking severity. Put
context-dependent alternatives under **Design questions**, not disguised
findings. Do not infer certainty from a missing line in simplified output.

## Output

Choose the report role before drafting: delegated area results use the shared
findings-and-coverage contract; standalone reports use the outline below. Omit
`Verdict:` on the requester's own PR. Pass that role through post-processing and
return only its filtered result.

```text
**Posted by an AI agent**

# Public API review: <package>

Scope: <tool version, package, features, target, and optional baseline>
Verdict: <approve | approve with non-blocking comments | changes requested | blocked>

## Findings
<Shared two-section finding blocks, with exact public paths and API excerpts.>

## Design questions
<context-dependent choices, each with exact API evidence>

## What is already clean
<brief, specific strengths; omit generic praise>

## Coverage and limitations
<item families/configurations reviewed, rustdoc JSON verification coverage, and
what this review cannot assess>
```

Return finding blocks in impact order, with evidence-based severity.
Keep excerpts distinct from prose with inline code or a fenced block.

If there are no findings, say so explicitly and still report the configurations
and API families covered. If extraction fails, use `blocked`, include the
decisive diagnostic, and do not issue an API verdict.

The coordinator owns combined presentation and delivery under the shared
contract. This skill returns mergeable filtered findings or a standalone
REPORT-ONLY result; it never posts.

[pragmatic-rust]: https://microsoft.github.io/rust-guidelines/
