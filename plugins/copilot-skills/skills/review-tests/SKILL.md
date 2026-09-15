---
name: review-tests
description: >
  Review Rust changes for deleted or weakened tests and observable behavior
  deltas, even when test files are untouched. Protects existing behavioral
  contracts from being rewritten to match an implementation, especially in
  AI-authored changes, and checks that changed tests are idiomatic and reuse
  supported test utility features such as test-util. Use when asked to review
  test preservation, behavior compatibility, or Rust test quality in a PR,
  branch, commit, working-tree diff, or focused audit. Not for coverage
  percentages or test-framework formatting.
---

# Review Tests

Baseline tests are contract evidence, not obstacles to green. Review only;
never change production code or expectations to make head pass.

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md). Own test changes,
preservation coverage and behavioral-delta authorization. Share contract deltas
with `review-api-design`, production defects with `review-correctness`; missing
coverage belongs in that root cause's fix. Own test utilities, not runtime
clock/randomness injection (`review-perf`). Use docs as preservation evidence;
docs-only disagreements belong to `review-consistency`.

## Procedure

1. **Inventory tests before implementation.** Diff test files/functions, inline
   `mod tests`, doctests, test examples, snapshots, fixtures, property cases and
   test-only manifests/configuration. Detect deletions and weakening: removed
   cases/assertions; `specific error -> is_err()` or exact value -> `contains`;
   changed expectations/snapshots; failure-avoiding `ignore`, `cfg`, early return,
   retry or larger timeout; narrower property ranges or feature/target coverage.
   **Even an empty test diff must continue to step 3.**
2. **Preserve each distinct baseline behavior.** Renames/moves/replacements need
   a surviving equivalent assertion through the same reachable surface. Public
   integration -> private unit coverage, or widening visibility to move tests,
   is not equivalent. Otherwise require restoration. Remove duplicate or
   auto-derived smoke tests only with explicit rationale and no distinct
   behavior lost. Replace obsolete tests only for explicitly authorized behavior
   with focused new coverage. Implementation changes, green head suites and
   updated snapshots alone never justify loss.
3. **Audit observable deltas.** Derive baseline contracts from tests, public
   docs/API and callers. Trace outputs, errors, panics, defaults, ordering,
   serialization, side effects, cancellation/timing and feature/target behavior.
   Each delta needs both:
   - explicit current user instruction or pre-existing maintainer-approved
     requirement, issue, design or release decision; and
   - tests naming and distinguishing the new behavior.

   New author/agent rationale, same-diff comments, implementation and changed
   tests are not independent authorization. Intentional breaks require
   repository-mandated migration, versioning and release notes.
4. **Reuse supported test utilities.** Before accepting custom clocks, sleeps,
   randomness, mock I/O, fake servers or safety bypasses, inspect crate/dependency
   manifests and docs for the actual feature (`test-util`, `test-utils` or
   equivalent). Enable only for development/test consumers; reuse helpers.
   Gate user-facing test APIs behind the test feature; use existing
   `private-test-util`/fixture-crate patterns for cycle-sensitive internal
   fixtures instead of expanding public API. Do not demand nonexistent features
   or helpers unsuited to the scenario.
5. **Check quality.** Require concise deterministic behavior-shaped tests and
   precise named outcomes per test/parameter/property case. Prefer integration
   tests for public-only behavior, unit tests for private invariants. Reuse
   fixtures, parameterization and snapshots to remove boilerplate. Use simplest
   faithful synchronization; async locks need guards crossing await. Avoid real
   sleeps/clocks, external services, brittle formatted-error matching and
   compiler-derived tests distinguishing no contract.

## Proof and coverage

Static diff/API/docs evidence suffices for lost/weakened assertions and explicit
contract deltas. Runtime claims require focused commands and exact base/head
outcomes under shared verification rules.

Identify baseline contract, head delta, missing authorization/coverage, decisive
evidence and consumer/regression impact; specify restoration, replacement or
coverage of authorized behavior.

Normally blocking (unlabelled): unjustified observable changes, lost distinct
coverage, or production-reachable test bypasses. Materially brittle,
non-idiomatic or duplicate test infrastructure is `Non-blocking`; concrete
impact decides edge cases.

Coverage: deleted/changed tests, behavior deltas and test utility features.
