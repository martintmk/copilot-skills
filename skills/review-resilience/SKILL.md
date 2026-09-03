---
name: review-resilience
description: >
  Review Rust changes for missing or incorrect recoverable::Recovery
  classification and for retry, timeout, circuit-breaker, hedging, fallback,
  or chaos behavior implemented from scratch instead of with seatbelt. Uses
  the exact recoverable version's _documentation::recipes module as the
  classification source of truth. Use for a PR, branch, commit, working-tree
  diff, or focused crate audit when asked to audit recoverability or seatbelt
  adoption. Not for a general code review or ordinary error changes with no
  recovery concern.
---

# Review Resilience

Review changed code by default. Expand to the whole crate only when requested.
Read the diff, manifests, relevant error definitions and conversions, resilience
call sites, configuration, and focused tests.

## Procedure

1. **Resolve the review scope.** For a PR or branch, diff from the target's
   merge base; for a commit, inspect its parent diff; for a working tree,
   include staged, unstaged, and untracked files. Read unchanged callers for
   context, but report only change-scoped findings unless a crate audit was
   requested.

2. **Load the version-matched guidance.**
   - Use `cargo metadata` and the lockfile, when present, to resolve each
     in-scope package's `recoverable` dependency edge, including aliases and
     multiple versions. Read that resolved version's
     `recoverable::_documentation::recipes` docs or
     `src/_documentation/recipes.rs`.
   - Extract the applicable recipe for flowing inner recovery information,
     permanent errors, or heuristic recovery. Do not classify from memory when
     the resolved recipe is available.
   - Before recommending `seatbelt`, resolve the package's dependency or the
     workspace-approved version, enabled features, and relevant module docs. If
     none exists, recommend the crate and feature without inventing a
     version-specific call.

3. **Inventory potentially recoverable failures.** Trace transient failures,
   service unavailability, timeouts, throttling, connection loss, temporary
   resource pressure, and wrapped versions of those failures from origin to the
   error returned to the caller. Include conversions that erase an inner error.

4. **Check `Recovery` at every boundary.**
   - Implement `Recovery` when at least one state may recover or the
     classification is expected to evolve. An error whose states are all
     permanently non-recoverable does not need the trait.
   - Apply the recipe's `RecoveryInfo` classification. Preserve an inner
     `Recovery` value through conversions (for example,
     `recovery: error.recovery()` in an `ohno` `#[from]` mapping); override it
     only when outer context changes recoverability.
   - For foreign errors without `Recovery`, use the recipe's supported
     heuristic. For `std::io::Error`, prefer the built-in `ErrorKind` conversion
     and walk `Error::source()` when the IO error is buried.
   - Do not infer recovery from formatted error text. Use typed variants, kinds,
     or structured causes.
   - Remember that `Recovery` reports information; it does not perform recovery.
     Preserve useful delay hints such as `Retry-After` for middleware to honor.
   - Verify representative recoverable, unavailable, permanent, wrapped, and
     heuristic paths with focused tests.

5. **Find hand-written resilience.** Search for attempt loops, retry counters,
   sleeps/backoff/jitter, deadline races, timeout cancellation, breaker state,
   parallel hedges, fallback routing, and fault injection. When behavior
   overlaps `seatbelt`, recommend the matching feature and API:

   | Hand-written behavior | Prefer |
   | --- | --- |
   | retry loop/backoff/jitter | `seatbelt::retry` |
   | per-attempt timeout | `seatbelt::timeout` |
   | open/half-open failure state | `seatbelt::breaker` |
   | concurrent duplicate attempts | `seatbelt::hedging` |
   | replacement output or route | `seatbelt::fallback` |
   | injected failures | `seatbelt::chaos::injection` (`chaos-injection`) |
   | injected latency | `seatbelt::chaos::latency` (`chaos-latency`) |

   Enable only required features; `seatbelt` has no default features. Reuse its
   config types and vocabulary instead of mirroring fields under new names.
   Prefer static layer composition over per-call switches. Keep the usual
   outside-to-inside order `fallback -> retry -> breaker -> timeout`, so every
   attempt is timed and observed by the breaker. Account for input cloning or
   restoration, idempotency, delay hints, cancellation/drop safety, and breaker
   partitioning. Do not duplicate the telemetry and retry jitter that
   `seatbelt` already supplies. Keep production fault injection behind the
   repository's test-only feature convention.

6. **Avoid false positives.** Do not call a business workflow loop, a
   protocol-mandated retransmission, or a simple value fallback resilience
   merely because it repeats or uses `unwrap_or`. Recommend `seatbelt` only
   after tracing equivalent failure-handling semantics.

## Findings

Return only actionable findings, ordered by impact. Each finding must include:

- an exact code anchor and triggering failure path;
- the incorrect or lost classification, or duplicated resilience mechanism;
- runtime impact; and
- a specific recipe-based `Recovery` fix or version-correct `seatbelt`
  replacement.

Static API and dependency findings may be argued from code and resolved docs.
Prove executable claims with the smallest existing test or a temporary focused
probe, quote the command and result, then remove the probe. If execution is
unavailable, state the concern as a question or coverage limitation, not a
finding.

End with one coverage line naming the reviewed errors and resilience mechanisms.
If there are no findings, say so without inventing recommendations.
