---
name: review-telemetry
description: >
  Review Rust changes for metrics, logs and spans that break OpenTelemetry
  semantic conventions, destabilize an existing dimension set, risk unbounded
  cardinality, allocate per emission, or duplicate instrumentation a library
  already provides. Use for a focused telemetry audit, or when review-lens routes
  changed telemetry here. Not for general logging style or for telemetry backend
  and exporter configuration review.
---

# Review Telemetry

Telemetry is a consumer contract: dashboards, alerts and queries depend on the
names and dimensions you ship, and changing them later breaks them silently.

## Naming and conventions

- **Follow OpenTelemetry semantic conventions** for metric, span and attribute
  names, and use the conventional value where one exists. Prefer a dot-separated
  namespace (`oxidizer.hyper`) and reuse the namespace its siblings use.
- **Align a new signal to the closest existing one.** If a sibling metric already
  measures a comparable thing, match its name shape, unit and attribute set
  rather than inventing a parallel vocabulary.
- **Do not destabilize a stable dimension set.** Adding, removing or renaming an
  attribute on an existing metric breaks the queries and dashboards built on it;
  call it out and require the same treatment as any other breaking change. Where
  a value can be missing, emit a sentinel rather than dropping the dimension.
- **A service-specific dimension does not belong in a shared component's
  defaults.** If only one consumer would query it, expose the value as an
  extension the consumer emits itself.
- **Telemetry names are required inputs, not optional.** A metrics or logging
  layer should take its name at construction, with a sensible default for the
  standard pipeline, and expose it on the pipeline context.
- **Units belong in the instrument**, not in the metric name, when the convention
  carries them.

## Cardinality and cost

- **Bound every metric attribute and span name.** Reject request IDs, raw URIs,
  and user or tenant identifiers as *metric* attribute values or in span names;
  use a bounded classification such as an error kind or a route template instead.
  Bounded labels look like `connect.hyper.timeout` or
  `request.connect.connection_refused` — a small, enumerable vocabulary. Span and
  log *attributes* may legitimately carry high-cardinality values where the
  convention calls for it (`url.full` on an HTTP client span); judge those on
  sensitivity and backend cost, not cardinality alone.
- **Order composite labels from lower to higher cardinality**, so the prefix
  stays queryable.
- **Do not allocate per emission.** A `String` attribute built on every call is a
  per-request allocation — prefer `&'static str`, `Cow<'static, str>` or a cached
  attribute set. Cache the instrument, not just the value.

## Structure and duplication

- **Do not re-implement instrumentation a library already emits.** When a
  middleware or client crate already publishes retry, timeout, breaker or request
  telemetry, adding a parallel log or metric at the call site duplicates it;
  remove the local copy and reuse the library's signal.
- **Be skeptical of spans.** Do not add spans where the stack does not actually
  consume them — a span that no backend reads carries no telemetry value, and
  regular tracing APIs, a metric, or a log are usually the right signal. Require
  a named consumer before accepting new span instrumentation.
- **Choose the right signal.** A per-request measurement is a metric, not a log
  line per request; use logs for events an operator must read.
- **Prefer a tracing front end for logs** where the crate already uses one, so a
  logger provider does not have to be boxed and stored locally.
- **Redaction is part of telemetry.** Anything carrying user or customer data
  must go through the repository's classification or redaction path before it is
  emitted.

## Evidence

Assert names and attributes from the emitted instrument definitions and any
snapshot or metrics test, not from the surrounding prose. Where the repository
has telemetry tests or exported-signal snapshots, cite the exact emitted name and
attribute set. Documented telemetry tables and dashboards are consumer evidence
that a rename is breaking.

## Findings

Report with the code anchor, the exact signal name and attributes, the convention
or sibling it diverges from, the operator-visible consequence (broken query,
cardinality blow-up, per-request allocation), and the concrete fix. If the change
adds no telemetry where a comparable component instruments its behavior, say so
once rather than demanding instrumentation the change does not need.

When invoked directly rather than through `review-lens`, first read the
repository's own rules from the base revision and treat green CI as the
baseline; `review-lens` carries the workspace-specific adaptation.

Post through the `review-delivery` skill when the review targets a PR.
