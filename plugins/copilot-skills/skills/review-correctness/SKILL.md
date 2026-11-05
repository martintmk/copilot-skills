---
name: review-correctness
description: >
  Find behavioral defects in changed Rust code through focused reproductions
  or complete source proofs. Covers boundaries, resources, cancellation,
  concurrency, time and invariants. Use for correctness audits or Review Lens.
  Not for API design, naming or formatting.
---

# Review Correctness

Find runtime defects in changed code, including private code. Trace what the
code does, not just its signatures.

Leave public contract design, retry and recovery policy, and test preservation
to their own reviews. When a defect lacks a regression test, put the test in
this finding's fix instead of a separate finding.

## Procedure

1. **Trace every changed path** that can affect behavior, from entry to effect:
   implementation, callers, errors, cleanup, cancellation, drop, concurrency and
   tests. Do not sample or stop at the first finding.
2. **Ask the relevant questions below**, not every question for every change.
3. **Prove each suspected defect** when you can run code:
   - Write the smallest test that shows the wrong behavior. It should fail
     before a fix and make a good regression test.
   - Run the same test at the base. If it fails there too, the defect is not new.
   - Use Miri for unsafe code and allocator claims. Use bounded adversarial
     inputs; never trigger a real `2^32` blow-up.
   - Try to disprove your own finding. Drop it if the check refutes it.
   - Match the configuration: `--all-features` does not test
     `#[cfg(not(feature = "..."))]` code or another target.
4. Without execution, require a complete source proof: reachable trigger,
   exact-version semantics, causal path and violated contract.
   Compare the base to establish that the change introduced the defect.
   Otherwise drop the suspicion or ask a conditional question.
   Finish source coverage even when builds are unavailable.

## Questions

- **Parsing and version gates:** Off-by-one ranges? A decoder version accepted
  in one place but rejected by a gate? An empty result that looks like success?
  Patch a fixture to the untested value, for example a valid v2 input that
  returns `Ok` with `callers == None`.
- **Representation:** Are sizes and indices checked against the real capacity
  or index model? Can decoding accept a shape it cannot represent and then
  iterate it?
- **Resources, free lists, undefined behavior:** Slots not released on unlink?
  Reads that consume state needed later? Tables that fill up in normal use?
- **Cancellation and drop:** What is committed if the future is dropped at each
  `.await`? Are waiters notified on abort? Can dropping a hedge stop a breaker
  from opening? Transports and middleware must handle cancellation.
- **Concurrency and sync/async bridges:** Locks held across `.await`?
  Thread-affine state moved between threads? Search both synchronous and
  asynchronous code for executor entry points such as `block_on`, including
  methods, free functions and local wrappers. Trace changed `block_on` calls
  before deciding whether their callers and executor topology permit progress.
  Calling it from async code can block the runtime thread that must drive the
  awaited future, and some runtimes panic when an executor is started from
  inside another executor. Calling it from synchronous code can still deadlock
  when the future needs the blocked thread or the same bounded pool to make
  progress. Raise a finding when the call graph and runtime topology prove the
  cycle or panic, or when a focused test reproduces it. Otherwise report the
  unverified risk as a question, not as a proven defect.
- **Time:** Test `duration_since` and similar at every valid system time.
  Saturate instead of panicking.
- **Invariants and round-trips:** Does every public path keep a fixed-width or
  round-trip promise, including `FromStr` and `TryFrom`, not just
  constructors? If docs promise behavior the code breaks, fix the code; never
  weaken the docs to hide it.

## Evidence

Give the trigger, causal proof or reproduction, impact and specific fix.
Include a focused regression test when useful. Show decisive values and
configuration. Never describe reasoning as execution.
Use the shared impact-based severity policy, including for private code.

## Report

Write each finding in the
[findings contract](../review-delivery/findings-contract.md).
Coverage names the paths traced, what you ran, and what remains unverified.
