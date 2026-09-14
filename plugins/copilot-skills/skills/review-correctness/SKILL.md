---
name: review-correctness
description: >
  Review Rust changes for behavioral defects and prove each one before raising
  it: control-flow and version gates, boundary and representation limits,
  resource and free-list models, cancellation and drop safety, time arithmetic,
  and round-trip or invariant claims. Uses reproductions, Miri and bounded
  adversarial inputs under the shared verification discipline. Use for a
  focused defect audit, or when review-lens routes behavioral risk here. Not for
  API design, naming, or formatting.
---

# Review Correctness

Trace the implementation, not the signature. A correctness finding is only worth
posting once it is reproduced. **No adequate reproduction means no correctness
finding** — keep an unproven suspicion out of the review, or phrase it as a
clearly identified question.

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md); reuse supplied context.

Own defects in changed runtime paths, including private implementation.
`review-api-design` owns public contract and error/panic conventions;
`review-resilience` owns recovery classification and middleware composition.
`review-tests` owns test changes and behavior-preservation coverage: a missing
regression test for a runtime defect belongs in that defect's fix, not a second
finding.

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
  just the constructor. A proven implementation violation stays here. If trusted
  intent supports the runtime behavior and only the documented claim is wrong,
  pass the evidence to `review-consistency` to scope or correct that claim.
  Do not resolve a runtime violation by silently weakening its documentation.

A private defect that produces wrong output, a hang, a leak, a panic or data
loss is blocking even though it is not semver-visible.

## Findings

Each finding names the triggering input or sequence and quotes the decisive
reproduced result.

Coverage line: the paths traced and what remained unverified.
