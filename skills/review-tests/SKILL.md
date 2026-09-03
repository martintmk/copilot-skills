---
name: review-tests
description: >
  Review Rust changes for deleted or weakened tests and observable behavior
  changes that lack explicit justification. Protects existing behavioral
  contracts from being rewritten to match an implementation, especially in
  AI-authored changes, and checks that changed tests are idiomatic and reuse
  supported test utility features such as test-util. Use when asked to review
  test preservation, behavior compatibility, or Rust test quality in a PR,
  branch, commit, working-tree diff, or focused audit. Not for coverage
  percentages or test-framework formatting.
---

# Review Tests

Treat baseline tests as contract evidence, not obstacles to make green. Review
only; do not change production code or expectations to make the head pass.

## Procedure

1. **Resolve base and head.** For a PR or branch, use the target merge base; for
   a commit, its parent; for a working tree, include staged, unstaged, and
   untracked files. Read repository rules before judging test placement or
   features.

2. **Inventory test changes before implementation changes.** Diff test files,
   inline `mod tests`, doctests, examples used as tests, snapshots, fixtures,
   property-test cases, and test-only manifest/configuration. Detect deleted
   files and functions, but also weakened coverage:
   - removed cases or assertions;
   - broader assertions (`specific error` to `is_err()`, exact value to
     `contains`);
   - updated snapshots/expected values;
   - new `ignore`, `cfg`, early return, retry, or larger timeout that avoids a
     failure; and
   - reduced property-test ranges or feature/target coverage.

3. **Preserve every distinct baseline behavior.**
   - A rename, move, or equivalent replacement is not deletion; point to the
     surviving assertion. Moving a public integration test to a private unit
     path is not equivalent, nor is widening visibility just to move a test.
   - Require restoration when no head test proves the same contract through the
     same reachable surface.
   - Removing an exact duplicate or auto-derived smoke test is acceptable only
     when it distinguishes no behavior and the rationale is explicit.
   - An obsolete test may be replaced only when the behavior change is
     explicitly authorized and the new contract has focused coverage.
   Never accept “the implementation changed,” a passing head suite, or an
   updated snapshot as justification by itself.

4. **Audit observable behavior.** Derive the old contract from baseline tests,
   public docs/API, and callers. Trace production changes affecting outputs,
   errors, panics, defaults, ordering, serialization, side effects,
   cancellation/timing, or feature/target behavior. For each delta require:
   - an explicit current user instruction or a pre-existing requirement, issue,
     design, or release decision approved by project maintainers; and
   - tests that name and distinguish the new behavior.
   A PR author or agent's new rationale, code comments in the same diff, the
   implementation itself, and changed tests are not independent approval. If
   the change is intentionally breaking, require any migration, versioning, and
   release-note treatment mandated by the repository.

5. **Use existing test utilities.** Before accepting custom clocks, sleeps,
   random sources, mock I/O, fake servers, or safety bypasses:
   - inspect manifests and docs for test utility features on the crate and its
     dependencies; use the actual supported name (`test-util`, `test-utils`, or
     repository equivalent);
   - enable the feature only for development/test consumers and reuse its
     helpers instead of rebuilding them;
   - keep user-facing test APIs behind the test feature; for internal
     cycle-sensitive fixtures, follow an existing `private-test-util` or fixture
     crate pattern rather than expanding public API; and
   - do not demand a feature when no applicable crate provides one or the helper
     does not model the scenario.

6. **Check test quality.** Tests should be behavior-shaped, deterministic, and
   concise. Each test or parameterized/property case should distinguish a named
   outcome with precise assertions. Prefer integration tests for public-only
   behavior and unit tests for private invariants. Reuse existing fixtures and
   parameterization/snapshot facilities when they remove boilerplate. Use the
   simplest synchronization that models the test; an async lock is warranted
   only when its guard must cross an await. Avoid real sleeps, wall clocks,
   external services, brittle formatted-error matching, and tests of
   compiler-derived behavior that distinguish no contract.

## Verification and findings

Static diff, API, or documentation evidence is sufficient for deleted or
weakened assertions and explicit contract deltas. When a behavioral claim
depends on runtime behavior, run the smallest relevant test or temporary probe
on both revisions and compare the exact outcome. Modify only a temporary
worktree, use a trusted checkout or isolated environment, quote the command and
decisive result, and remove probes afterward.

Follow the shared **findings contract** in `review-delivery`. Each finding also
names the baseline contract, the head delta, and the missing justification or
coverage, with the exact restoration or replacement.

Severity in this area is strongly category-linked, though impact still decides a
genuine edge case:

- **Blocking** (unlabelled): unjustified observable behavior change, lost
  distinct test coverage, or a test-only bypass reachable in production.
- **`non-blocking:`**: materially brittle, non-idiomatic, or duplicated test
  infrastructure.

Coverage line: the deleted and changed tests, behavior deltas, and test utility
features reviewed.

When invoked directly rather than through `review-lens`, first read the
repository's own rules from the base revision and treat green CI as the
baseline; `review-lens` carries the workspace-specific adaptation.

Post through the `review-delivery` skill when the review targets a PR.
