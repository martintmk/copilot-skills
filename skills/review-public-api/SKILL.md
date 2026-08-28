---
name: review-public-api
description: >
  Review a Rust library crate's exported API first using only cargo-public-api
  output, without reading Rust source, manifests, docs, tests, or diffs. Then use
  a separate agent and cargo-generated rustdoc JSON to remove or narrow claims
  refuted by the real API docs. Use when asked to review, audit, or critique a
  crate's public API for a clean, idiomatic, consumer-friendly, and evolvable
  design, applying common Rust API best practices with API-visible Pragmatic Rust
  Guidelines on top. For small PRs, focuses findings on the changed public items
  shown by an API diff. Not for implementation, correctness, safety,
  documentation quality, performance, or general code review.
---

# Review Public API

Review the public contract of a Rust library from the consumer's perspective.
`cargo public-api` output is the sole source for generating candidate findings.
Judge only what the output makes observable, applying idiomatic Rust API
conventions and the API-relevant [Pragmatic Rust Guidelines][pragmatic-rust].
Before returning the review, a separate agent uses generated rustdoc JSON only
to remove or narrow candidate statements that the API docs refute or answer.

## Hard boundary: output-only review, docs-only filtering

- During the initial review, do not open or search Rust source, `Cargo.toml`,
  `Cargo.lock`, build scripts, docs, tests, examples, diffs, generated rustdoc
  JSON, or repository history.
- During the initial review, do not use `cargo metadata`, rust-analyzer, rustdoc
  pages, code search, or inferred implementation details to supplement the
  review.
- You may use the user's stated scope and intended use cases to understand the
  request, but every candidate finding must be proven by exact
  `cargo public-api` output.
- The only documentation exception is the mandatory post-processing pass. It
  runs in a separate agent after the provisional report exists and may inspect
  only the generated rustdoc JSON fields needed to associate public items with
  their docs. It must not inspect source, manifests, tests, examples, diffs, or
  rendered rustdoc pages.
- Documentation may remove a finding, answer a design question, or narrow
  overbroad wording. It must never create, strengthen, or raise the severity of a
  finding.
- Tool version/help text and build diagnostics may guide tool operation, but are
  not evidence about API quality.
- Do not modify the reviewed repository. Store any captured output outside it
  and remove temporary artifacts after reporting.

This boundary means the review cannot establish implementation correctness,
runtime behavior, validation, panic behavior, soundness, allocation or
performance characteristics, overall documentation quality, or whether
sensitive data is redacted. The post-processor treats docs as evidence of
documented API intent and usage, not proof of implementation behavior. Do not
turn those unknowns into findings.

## Procedure

1. **Fix the scope without inspecting the crate.**
   - Use a package, manifest path, feature set, target, or baseline named by the
     user.
   - Otherwise review the package selected by `cargo public-api` with
     `--all-features` and the host target. This is the default configuration so
     feature-gated public APIs such as builders are included.
   - If the command reports an ambiguous workspace, ask for the package name.
     Do not read the workspace manifest to choose one.
   - State the selected configuration. An all-features review covers feature
     combinations together, not every individual feature combination or other
     targets.

2. **Check trust before execution.** `cargo public-api` builds rustdoc JSON and
   can execute build scripts or procedural macros. Run it only for trusted code,
   or in an isolated, credential-free environment. Do not inspect source to
   decide whether it is safe.

3. **Ensure the tool is available.**

   ```text
   cargo public-api --version
   ```

   If the command is missing, install the official tool with a stable toolchain:

   ```text
   cargo +stable install cargo-public-api --locked
   ```

   If it reports that a compatible nightly toolchain is missing, install one:

   ```text
   rustup toolchain install nightly --profile minimal
   ```

   If installation or API extraction fails, stop and report the exact command
   and decisive diagnostic. Do not troubleshoot by reading the crate.

4. **Collect the complete all-features surface.** Unless the user explicitly
   selected another feature configuration, include `--all-features`. Add only
   other scope arguments established in step 1, such as `-p`,
   `--manifest-path`, `--features`, `--no-default-features`, or `--target`.

   ```text
   cargo public-api --color=never --include function-parameter-names --all-features <scope-args>
   ```

   This full output is authoritative for trait, auto-trait, and derived-trait
   checks. Capture it verbatim if needed; do not rely on a truncated display.

5. **Collect a readable inventory.**

   ```text
   cargo public-api --color=never --include function-parameter-names --all-features -sss <scope-args>
   ```

   The simplified output omits blanket implementations, auto traits, and
   auto-derived implementations. Use it to inspect structure and signatures,
   but return to the full output before claiming that a type lacks `Debug`,
   `Clone`, `Send`, or another omitted implementation.

6. **Optionally collect a change view.** Only when the user requests a semver or
   change review, run the matching tool-supported baseline in addition to the
   full current-surface commands:

   ```text
   cargo public-api --color=never --include function-parameter-names --all-features <scope-args> diff latest
   cargo public-api --color=never --include function-parameter-names --all-features <scope-args> diff <version>
   cargo public-api --color=never --include function-parameter-names --all-features <scope-args> diff <ref1>..<ref2>
   ```

   Never use `--force`: commit diffing performs in-place checkouts, and forcing
   it can discard worktree changes. A diff is evidence about changes, not a
   substitute for reviewing the complete current surface.

7. **Inventory before judging.** Group every emitted item by public path and
   family: modules/re-exports, types and fields, traits and implementations,
   functions and methods, constants/statics, macros, and error/builder/iterator
   families. Finish the inventory even after finding an issue.

8. **Choose the review set.**
   - For a standalone crate audit, review the complete current output.
   - For a PR or change request, use the API diff to identify additions,
     removals, and changed signatures. If the diff is small (a few public items),
     make those changed items the review set and use the unchanged output only to
     understand their immediate surrounding family. Do not generate findings
     about unrelated pre-existing APIs.
   - If the PR changes a broad or unclear portion of the surface, review the
     complete current output and say so in the coverage section.
   - Apply the review lenses below to the selected review set, starting with
     common idiomatic Rust API practices and then adding the Pragmatic Rust
     Guidelines. Honor guideline intent and stated exceptions; do not flag a
     pattern merely because it resembles a discouraged example.

9. **Draft only proof-carrying candidate findings.** Quote the decisive emitted
   line or lines exactly. Separate definite findings from context-dependent
   design questions. If the tool cannot expose the evidence, omit the claim and
   name the limitation once in the coverage section. Do not return this
   provisional report yet.

10. **Post-process the complete draft in a separate agent.** Follow the
    [rustdoc JSON post-processing procedure](rustdoc-post-processing.md). Give a
    fresh agent the provisional report, reviewed crate and working directory,
    exact package/features/target/baseline scope, and toolchain context. That
    agent generates matching cargo rustdoc JSON outside the repository,
    associates each claim with the relevant item docs, and returns a filtered
    report with refuted statements removed or narrowed. Keep doc inspection out
    of this agent's context and publish only the filtered report. If the
    post-processor or JSON generation fails, do not present the provisional
    claims as verified; return `blocked` with the decisive diagnostic.

## Review lenses

### 0. Common idiomatic Rust API baseline

Apply this baseline to every selected item before the Pragmatic Rust overlay.
Use the emitted API shape, not assumptions about implementation:

- Prefer conventional Rust names and signatures: `snake_case` functions and
  modules, `UpperCamelCase` types and traits, `SCREAMING_SNAKE_CASE` constants,
  `new`/`default` constructors, receiver methods for instance behavior, and
  `Result`/`Option` where the signature communicates fallibility or absence.
- Prefer standard-library vocabulary and traits over bespoke equivalents:
  `From`/`TryFrom`, `AsRef`/`AsMut`, `Borrow`, `Iterator`, `IntoIterator`,
  `FromIterator`, `Extend`, `Error`, and formatting traits where their semantics
  fit. Use `as_`, `to_`, and `into_` consistently.
- Make ownership and borrowing legible. Prefer borrowing for read-only access,
  consuming `self` for conversions/builders, and avoid needless `clone`,
  `String`, `Vec`, or smart-pointer requirements at caller-facing boundaries.
- Prefer the least surprising receiver, generic bounds, lifetime surface, and
  return type. Avoid needless `dyn Trait`, opaque nesting, boolean/tuple
  parameter ambiguity, and public implementation-detail types.
- Make public types composable: provide appropriate `Debug`, `Display`,
  equality/hash/order, `Clone`/`Copy`/`Default`, iterator, conversion, and
  `Send`/`Sync` support when the type's meaning and role warrant them.
- Preserve forward compatibility: avoid exposing fields or concrete details
  unnecessarily, use `#[non_exhaustive]` where a growing enum/struct requires
  it, keep public traits implementable or deliberately sealed, and avoid
  redundant export paths.
- Keep APIs discoverable and consistent with their sibling items. Constructors
  should be easy to find, fallible construction should be explicit, errors
  should implement `std::error::Error`, and public behavior should not be split
  across surprising extension-only entry points.

These are review heuristics, not automatic defects. Report only a concrete
consumer or evolution cost demonstrated by the output. The Pragmatic Rust
Guidelines below add stricter, repository-independent preferences where they
provide additional safety, maintainability, or UX value.

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

- Check Rust casing and established vocabulary. Prefer short, precise names over
  fillers such as `Manager`, `Helper`, `Util`, `Common`, or `Data` when those
  words add no distinction (`M-WEASEL-WORDS`, `M-SHORT-NAMES`).
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

- Simple types should have a clear constructor/default path. When a type exposes
  many independent configuration permutations, prefer `Type::builder()` and
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
- For a requested diff, classify additions, removals, and changed signatures,
  then assess consumer impact. For a small PR, findings must be limited to those
  changed public items and directly affected signatures; do not use the baseline
  to opportunistically review unrelated APIs. Do not infer semver impact from a
  current-surface listing alone.
- Macro names and signatures visible in the output may be inventoried, but macro
  syntax, expansion hygiene, generated bounds, and behavior are out of scope.

## Evidence and severity

A finding must contain:

1. **Severity and item** — `high`, `medium`, or `low`, plus the exact public path.
2. **Evidence** — the smallest exact `cargo public-api` excerpt proving the
   shape. Include related lines when absence would otherwise be ambiguous.
3. **Consumer impact** — the concrete usability, interoperability, type-identity,
   or evolution cost.
4. **Recommendation** — a specific better public shape, not an implementation
   patch.
5. **Guideline** — the applicable Pragmatic Rust ID or idiomatic Rust convention.

Use `high` only for a concrete, substantial consumer or compatibility problem;
most API cleanliness findings are `medium` or `low`. Present a
context-dependent alternative under **Design questions**, not as a defect.
Never manufacture certainty from a missing line in simplified output.

## Post-processing

Treat the complete report from the output-only review as provisional. Before
returning it, launch a fresh agent with the handoff defined in the
[rustdoc JSON post-processing procedure](rustdoc-post-processing.md). The
post-processor, not the parent review agent, generates and reads matching cargo
rustdoc JSON, associates docs with each claim, and filters the report at
statement level.

This pass is mandatory even when the API evidence appears decisive: docs can
establish an intended constraint, explain that a documented guideline exception
applies, or answer a design question. The post-processor may remove or narrow
such statements, but it may not add findings or use docs as proof of runtime
behavior. Publish its filtered report without loading the docs into the parent
agent's context.

## Output

Return only the report produced by the rustdoc JSON post-processor:

```text
# Public API review: <package>

Scope: <tool version, package, features, target, and optional baseline>
Verdict: <clean | clean with questions | needs changes | blocked>

## Findings
### [severity] <public path>: <consumer-facing headline>
Evidence:
<exact cargo public-api line(s)>
Impact: <specific consequence>
Recommendation: <specific API shape>
Guideline: <ID or convention>

## Design questions
<context-dependent choices, each with exact API evidence>

## What is already clean
<brief, specific strengths; omit generic praise>

## Coverage and limitations
<item families/configurations reviewed, rustdoc JSON verification coverage, and
what this review cannot assess>
```

If there are no findings, say so explicitly and still report the configurations
and API families covered. If extraction fails, use `blocked`, include the
decisive diagnostic, and do not issue an API verdict.

[pragmatic-rust]: https://microsoft.github.io/rust-guidelines/
