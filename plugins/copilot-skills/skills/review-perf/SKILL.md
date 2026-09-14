---
name: review-perf
description: >
  Review Rust changes for avoidable allocations, hot-path cost, and unmockable
  time or randomness. Covers needless String/clone/round-trip allocation,
  per-call dispatch that could be decided once, algorithmic growth, contention,
  hard-wired clocks or entropy where injectable sources exist, and honest
  benchmark reporting. Use for cost or clock/randomness-injection audits, or
  when review-lens routes a hot path or claimed optimization here. Not for
  micro-style preferences or off-path optimizations that add complexity for no
  measured gain.
---

# Review Perf

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md); reuse supplied context.

Own cost classification and measurement, plus injectable clocks and randomness
in runtime code. Test-only utility use belongs to `review-tests`, emitted signal
contracts to `review-telemetry`, and unnecessary abstractions without a cost
claim to `review-naming`. Reuse telemetry and resilience evidence for shared
costs rather than reporting the same allocation or per-call work twice.

## Lenses

- **Classify first.** Is the path per-request, per-item, per-connection, or once
  at startup? Only the first three earn optimization pressure; name the class in
  findings. Off the hot path, added complexity to dodge an allocation is itself
  a finding.
- **No needless allocation for static data.** A `String` field forces an
  allocation when every value is built from static text — prefer
  `Cow<'static, str>`, `HeaderName`, `HeaderValue`, or an enum of common variants
  with an `Other(..)` escape. Drop `str` → `Uri` → `str` round-trips, reflexive
  `.clone()`, and `to_string()` on a value that is already owned.
- **Branch once on values that never change.** A configuration flag read on every
  call should be resolved when the pipeline or service is built, not switched at
  runtime per request.
- **Dispatch.** Prefer static dispatch on the hot path, but type erasure at the
  edge of a stored pipeline is usually an acceptable, measured trade — do not
  demand generics for their own sake.
- **Algorithmic growth and contention.** Watch repeated work, quadratic scans,
  lock contention and per-call synchronization. A cache or fast path that only
  ever sees one entry is worth specializing.
- **Time and randomness must be injectable.** Flag `tokio::time::sleep`,
  `Instant::now()` and `SystemTime::now()` where a clock abstraction exists, and
  ad-hoc entropy where a seedable source exists. This is a testability finding as
  much as a performance one: injected time makes the behavior deterministic.
- **Bounded memory.** Code handling external input should chunk or bound its
  allocation rather than trusting the caller's size.

## Evidence

- Bring a benchmark for any claim that a change is faster or slower, and prefer
  the repository's existing harness over a new one.
- **Report honestly.** Give the point estimate and the range, and say whether the
  difference is significant — "~0.21 ns (~4%), not statistically significant in
  Criterion, so it does not justify carrying the reference".
- An allocation-count or instruction-count assertion is often a better regression
  guard than wall-clock timing; prefer it where the repository supports it.
- Do not claim a regression from reading code alone. Either measure it, or raise
  it as a question and say it is unmeasured.

## Findings

For actionable findings, use the shared attribution and a concise bold diagnosis
title naming the affected path. **Problem** gives its classification, the
avoidable work or hard-wired time/randomness source, and decisive evidence,
including measurements when taken. **Why this matters** states the concrete cost
or testability impact. **Suggested fix** gives the specific correction without
adding unjustified complexity. Keep unmeasured runtime claims conditional.

Mark a finding `Non-blocking` when it is off the hot path or unmeasured. If the
change is performance-neutral, say so rather than inventing micro-optimizations.

Coverage line: the paths classified, what you measured, and what remained
unmeasured.
