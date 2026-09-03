---
name: review-correctness
description: >
  Review Rust changes for behavioral defects and prove each one before raising
  it: control-flow and version gates, boundary and representation limits,
  resource and free-list models, cancellation and drop safety, time arithmetic,
  and round-trip or invariant claims. Owns the verification discipline the other
  review-* skills reuse, including base-versus-head attribution, Miri, bounded
  adversarial inputs, and honest reporting of what was not checked. Use for a
  focused defect audit, or when review-lens routes behavioral risk here. Not for
  API design, naming, or formatting.
---

# Review Correctness

Trace the implementation, not the signature. A correctness finding is only worth
posting once it is reproduced. **No adequate reproduction means no correctness
finding** — keep an unproven suspicion out of the review, or phrase it as a
clearly identified question.

## Coverage: trace every changed correctness-sensitive path

Trace the risky changed behaviour from entry point to effect — implementation,
callers, error paths, cleanup, cancellation and drop, concurrency, and the tests.
**Do not sample the implementation:** review *all* changed correctness-sensitive
paths, not a representative few, and keep going after the first finding rather
than stopping there.
Public API priority elsewhere in the review never licenses skipping one here.

## Defect classes

These are the *kinds* of defect worth hunting, drawn from real reviews, not a
checklist to force onto every PR. Apply the ones the change actually risks.

- **Control-flow and version gates on parsing/decoding.** Off-by-one accept
  ranges, a version the reader parses but the gate rejects, a valid input that
  returns a success-shaped empty. Prove it by patching a fixture to the untested
  value — "a valid v2 snapshot returns `Ok` with `callers == None`".
- **Boundary and representation.** Sizes/indices not validated against the real
  capacity or index model; an unrepresentable topology accepted then iterated.
  Prove it by decoding a *bounded* adversarial input.
- **Resource, free-list and UB models.** Slots not released on unlink, reads that
  consume state a later read needs, tables that fill under ordinary operation —
  the class where running the specific test under Miri turns a hunch into a
  quoted panic.
- **Cancellation and drop safety.** What stays committed if a future is dropped
  at an await point; waiters not notified on abort; a dropped hedged attempt that
  prevents a circuit breaker from opening. Transports and middleware must honor
  cancellation rather than assume completion.
- **Concurrency and thread affinity.** Blocking calls on an async path, a lock
  held across an await, and thread-affine state relied on across a move. The
  *contract* side of `!Send` — enforcing it in the returned public type rather
  than in documentation — belongs to `review-api-design`.
- **Time arithmetic.** `duration_since` and friends must not fail for any valid
  system time; saturate rather than panic; test the boundary.
- **Round-trip and invariant claims.** If a type advertises a fixed width or a
  round-trip, prove it holds on *all* public paths (`FromStr`, `TryFrom`), not
  just the constructor — and if it does not, scope the claim or enforce it.

A private defect that produces wrong output, a hang, a leak, a panic or data
loss is blocking even though it is not semver-visible.

## Verification discipline

This is what makes a finding a proof rather than an opinion. Other `review-*`
skills reuse these rules.

- **Attribute against the base, not just a green head.** When claiming the change
  introduced a regression, run the same test on the merge base too, so "passes on
  base, fails on head" is measured. If it also fails on the base, do not
  attribute it to the PR.
- **Reproduce with the smallest faithful test.** It must assert the intended
  behavior so it fails before the fix and becomes useful regression coverage
  after it. Patch an encoded fixture to hit an untested branch; run under Miri
  for unsafe/allocator code; parse a *bounded* adversarial input — never actually
  trigger a `2^32`-iteration blow-up, reason from the decoded size.
- **Falsify, don't just confirm.** Run the check that would refute the finding.
  On a fresh review, if it turns out fine, simply do not raise it; a retracted
  false alarm is worth more than a wrong block.
- **Check the real configuration matrix.** A defect can hide in a
  `#[cfg(not(feature = "…"))]` build that `--all-features` never compiles.
- **Quantify sweeping claims.** If you assert something about a whole change,
  count it.
- **Quote exact values, not paraphrases.** Prefix a reproduced finding with
  `Verified:`; argue non-behavioral findings tightly; never dress a guess as a
  verification. **State what you could not check** rather than guessing.
- **Do not re-run CI.** The PR's own CI owns lint, format and the full suite;
  read its status and build or run only the narrowest thing a specific finding
  needs. When CI is red, open the failing job rather than re-deriving it.
- **Revert every probe** and remove any temporary worktree afterwards.

**Safety:** a worktree is not a sandbox. Checking out and building a PR runs its
`build.rs`, proc-macros and tests as arbitrary code with your credentials and
network. Gate execution on provenance and environment, not a familiar author
name: build and run an untrusted or fork PR only in an isolated, credential-free
environment. If no safe environment is available, raise correctness concerns only
as questions.

## Findings

Report defects in impact order, each with the code anchor, the triggering input
or sequence, the runtime consequence, the decisive verification, and the fix.
Include a complete focused failing test when it directly demonstrates the defect
and should become permanent regression coverage; otherwise cite the result.
End with a line naming the paths traced and what remained unverified.

When invoked directly rather than through `review-lens`, first read the
repository's own rules from the base revision and treat green CI as the
baseline; `review-lens` carries the workspace-specific adaptation.

Post through the `review-delivery` skill when the review targets a PR.
