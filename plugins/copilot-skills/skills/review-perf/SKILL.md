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
[findings contract](../review-delivery/findings-contract.md). Own cost
classification/measurement and runtime clock/randomness injection. Test-only
utilities belong to `review-tests`, signal contracts to `review-telemetry`, and
abstractions without cost claims to `review-naming`. Reuse telemetry/resilience
evidence without duplicating the same allocation or per-call finding.

## Procedure

1. **Classify frequency:** per-request, per-item, per-connection or startup-only.
   Only the first three earn optimization pressure. Off-path complexity to
   avoid allocations can itself be a finding.
2. Apply the cost/injection questions below.
3. Benchmark faster/slower claims, preferring the repository's existing harness.
   Never infer a regression by reading code; unmeasured runtime suspicions are
   conditional questions.

## Specialist questions

- **Allocation:** For static text prefer `Cow<'static, str>`, `HeaderName`,
  `HeaderValue`, or common enum variants with `Other(..)` over `String`.
  Remove needless `str -> Uri -> str`, reflexive `.clone()` and `to_string()`
  on owned values.
- **Frequency/dispatch:** Resolve invariant flags when building the pipeline or
  service, not per request. Prefer hot-path static dispatch; measured type
  erasure at a stored pipeline's edge is legitimate, not grounds for generics
  for their own sake.
- **Growth/contention:** Check repeated work, quadratic scans, locks and
  per-call synchronization. Consider specializing single-entry caches/fast
  paths. Chunk or bound external-input allocations; never trust caller sizes.
- **Determinism:** Flag `tokio::time::sleep`, `Instant::now()` and
  `SystemTime::now()` where clock abstractions exist, and ad-hoc entropy where a
  seedable source exists. Injection is also a testability requirement.

## Proof and coverage

Report benchmark point estimate, range and statistical significance: e.g.
"~0.21 ns (~4%), not significant in Criterion, so insufficient to justify the
reference." Prefer allocation/instruction-count regression guards over timing
where supported.

Identify path frequency, avoidable work or hard-wired source, decisive evidence
and measurements, concrete cost/testability impact, and a correction without
unjustified complexity. Keep unmeasured runtime claims conditional; off-path or
unmeasured findings are `Non-blocking`. Report performance neutrality rather
than inventing micro-optimizations.

Coverage: paths classified, measurements and unmeasured areas.
