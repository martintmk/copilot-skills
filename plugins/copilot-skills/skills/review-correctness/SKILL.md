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

Trace changed runtime implementations, including private code, not signatures.
Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md).

Public/error/panic contracts belong to `review-api-design`; recovery and
middleware composition to `review-resilience`; test changes and preservation to
`review-tests`. Missing regression coverage belongs in the runtime defect's fix,
not a duplicate finding.

## Procedure

1. Trace **every** changed correctness-sensitive path from entry to effect:
   implementation, callers, errors, cleanup, cancellation/drop, concurrency and
   tests. Never sample, stop at the first finding, or skip paths for API priority.
2. Apply the relevant defect questions below, not an indiscriminate checklist.
   Reproduce with faithful focused tests, bounded adversarial inputs and Miri
   where appropriate, following shared baseline/configuration and falsification
   rules.
3. **No adequate reproduction means no correctness finding.** Omit unproven
   suspicions or identify them as questions.

## Defect questions

- **Parsing/version gates:** Off-by-one ranges, accepted decoder versions
  rejected by gates, success-shaped empty results? Patch a fixture to the
  untested value, e.g. valid v2 returns `Ok` with `callers == None`.
- **Representation:** Are sizes/indices checked against actual capacity/index
  models? Can decoding accept an unrepresentable topology then iterate it?
  Use bounded adversarial input.
- **Resources/free lists/UB:** Unreleased slots on unlink, reads consuming
  later-needed state, tables filling during ordinary use? Use Miri for applicable
  focused tests and quote the decisive result.
- **Cancellation/drop:** What commits when dropped at each await? Are abort
  waiters notified? Can dropping a hedge prevent breaker opening? Transports and
  middleware must honor cancellation, not assume completion.
- **Concurrency:** Blocking async calls, locks across await, thread-affine state
  crossing moves? Enforcing public `!Send` types belongs to `review-api-design`.
- **Time:** Test `duration_since` and similar boundaries for every valid system
  time; saturate rather than panic.
- **Invariants/round-trips:** Prove fixed-width or round-trip promises on all
  public paths, including `FromStr` and `TryFrom`, not just constructors.
  Implementation violations stay here. Only when trusted intent supports
  runtime behavior should `review-consistency` correct a wrong documented claim;
  never weaken docs to conceal a runtime violation.

## Proof and coverage

Give the triggering input/sequence, decisive reproduced result, concrete impact
and specific correction, with a focused regression test when useful. Private
wrong output, hangs, leaks, panics or data loss are blocking despite lacking
semver visibility.

Coverage: all paths traced and what remained unverified.
