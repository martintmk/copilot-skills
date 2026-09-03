---
name: review-perf
description: >
  Review Rust changes for avoidable allocations, hot-path cost, and unmockable
  time. Covers needless String/clone/round-trip allocation, per-call dynamic
  dispatch that could be decided once, algorithmic growth and contention, direct
  clock and sleep calls where an injectable clock exists, and honest benchmark
  reporting. Use for a focused performance audit, or when review-lens routes a
  hot path or claimed optimization here. Not for micro-style preferences or
  optimizations off the hot path that add complexity for no measured gain.
---

# Review Perf

Classify the path before spending any complexity on it. Off the hot path, do not
add complexity to dodge an allocation — that is itself a finding.

## Lenses

- **Classify first.** Is this code per-request, per-item, per-connection, or
  once at startup? Only the first three earn optimization pressure. Say which
  when raising a finding.
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

Performance claims need numbers, not adjectives.

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

Follow the shared **findings contract** in `review-delivery`. Each finding also
names the path classification and the cost — allocation per call, extra dispatch,
contention — plus the measurement if you took one.

Mark a finding `non-blocking:` when it is off the hot path or unmeasured. If the
change is performance-neutral, say so rather than inventing micro-optimizations.

Coverage line: the paths classified, what you measured, and what remained
unmeasured.

When invoked directly rather than through `review-lens`, first read the
repository's own rules from the base revision and treat green CI as the
baseline; `review-lens` carries the workspace-specific adaptation.

Post through the `review-delivery` skill when the review targets a PR.
