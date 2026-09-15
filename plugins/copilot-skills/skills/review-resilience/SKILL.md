---
name: review-resilience
description: >
  Review Rust changes for recovery classification and retry, timeout,
  circuit-breaker, hedging, fallback or chaos behavior that should use approved
  middleware. In recoverable/seatbelt repositories, checks Recovery against the
  exact recoverable version's _documentation::recipes and audits seatbelt
  adoption. Use for a PR, branch, commit, working-tree diff, or focused crate
  audit. Not for a general code review or ordinary error changes with no
  recovery concern.
---

# Review Resilience

Review changed code; expand crate-wide only on request. Read diff, manifests,
errors/conversions, resilience call sites including unchanged callers,
configuration and focused tests.

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md). Own recovery
propagation/classification and middleware selection/composition. Route
error/panic conventions to `review-api-design`, other runtime defects to
`review-correctness`, and emitted contracts to `review-telemetry`.

Use `recoverable`/`seatbelt` only where shared repository adaptation selects them;
else use repository equivalents, not new Oxidizer dependencies.

## Procedure

1. **Resolve exact recipes.** Use `cargo metadata` and any lockfile to resolve
   each package's `recoverable` edge, including aliases/multiple versions. Read
   that version's `recoverable::_documentation::recipes` or
   `src/_documentation/recipes.rs`; extract applicable inner-propagation,
   permanent-error or heuristic recipes, not remembered classifications. Before
   recommending `seatbelt`, resolve dependency/workspace-approved version,
   enabled features and module docs. If approved without an established version,
   recommend only crate/feature, never an invented version-specific call.
2. **Inventory failure flows.** Trace transient failures, unavailability,
   timeouts, throttling, connection loss, temporary resource pressure and
   wrappers from origin to caller, including conversions erasing inner errors.
3. **Check every recovery boundary.**
   - Require `Recovery` when some state may recover or classification may evolve;
     permanently non-recoverable-only errors need no trait.
   - Apply recipe `RecoveryInfo`; preserve inner recovery through conversions,
     e.g. `recovery: error.recovery()` in `ohno` `#[from]`. Override only for
     outer context that changes recoverability.
   - Use supported heuristics for foreign errors without `Recovery`; for
     `std::io::Error`, prefer built-in `ErrorKind` conversion and traverse
     `Error::source()` when buried. Use typed variants/kinds/causes, not text.
   - `Recovery` informs, not performs, recovery. Preserve `Retry-After` and
     other useful delay hints. Test recoverable, unavailable, permanent,
     wrapped and heuristic paths.
4. **Find equivalent middleware.** Inspect attempt loops/counters, sleeps,
   backoff/jitter, deadline races, timeout cancellation, breaker state, parallel
   hedges, fallback routing and fault injection:

   | Behavior | Prefer |
   | --- | --- |
   | retry/backoff/jitter | `seatbelt::retry` |
   | per-attempt timeout | `seatbelt::timeout` |
   | open/half-open state | `seatbelt::breaker` |
   | duplicate concurrent attempts | `seatbelt::hedging` |
   | replacement output/route | `seatbelt::fallback` |
   | injected failures | `seatbelt::chaos::injection` (`chaos-injection`) |
   | injected latency | `seatbelt::chaos::latency` (`chaos-latency`) |

   Enable only required features; there are no defaults. Reuse config types and
   vocabulary. Prefer static composition, usually outside-to-inside
   `fallback -> retry -> breaker -> timeout`, so every attempt is timed and
   observed by the breaker. Check input cloning/restoration, idempotency, delay
   hints, cancellation/drop and breaker partitioning. Do not duplicate supplied
   telemetry/jitter. Gate production fault injection with the repository's
   test-only feature convention.
5. **Exclude lookalikes.** Business workflow loops, protocol-mandated
   retransmission and simple value fallback (`unwrap_or`) are not automatically
   resilience. Trace equivalent failure-handling semantics before recommending
   middleware.

## Proof and coverage

Identify triggering flow, lost/incorrect classification or duplicate mechanism,
decisive evidence, reliability impact and recipe-based/version-correct fix.
Static API/dependency findings can use code and resolved docs. Executable claims
need shared-rule reproduction with focused command/result; unconfirmed behavior
is a question or limitation.

Coverage: errors/mechanisms and resolved `recoverable`/`seatbelt` versions or
repository equivalents.
