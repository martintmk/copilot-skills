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

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md). Own emitted names,
dimensions, units, cardinality, redaction and duplicate instrumentation, on which
dashboards, alerts and queries depend.

Route symbol names to `review-naming`, cost-only findings to `review-perf`, and
stale/contradictory docs to `review-consistency`. Keep emitted-contract defects
here, combining signal and measured-cost evidence rather than duplicating them.

## Procedure

1. Inventory changed signals from instrument definitions and telemetry tests or
   exported-signal snapshots, not surrounding prose. Locate comparable signals
   and consumers.
2. Apply the contract and emission questions below.
3. Return exact emitted evidence and impact; apply `review-perf` measurement
   requirements to per-emission cost claims.

## Signal-contract questions

- Follow OpenTelemetry semantic names/values for metrics, spans and attributes.
  Reuse sibling dot-separated namespaces (`oxidizer.hyper`), name shapes, units
  and attribute sets rather than inventing parallel vocabulary.
- Treat additions/removals/renames in stable metric dimensions as breaking
  changes to dependent queries/dashboards. Emit a sentinel for missing values,
  never drop the dimension.
- Keep service-specific dimensions out of shared defaults; expose an extension
  for the consumer to emit.
- Require metrics/logging layer names at construction, with sensible standard
  pipeline defaults, and expose names on pipeline context.
- Put convention-carried units in instruments, not metric names.

## Emission questions

- Bound **metric attributes and span names**: no request IDs, raw URIs, user or
  tenant identifiers. Use enumerable error kinds or route templates, e.g.
  `connect.hyper.timeout` or `request.connect.connection_refused`.
  Convention-appropriate span/log **attributes** may be high-cardinality
  (`url.full` on HTTP client spans); assess sensitivity/backend cost, not
  cardinality alone. Order composite labels low-to-high cardinality so prefixes
  remain queryable.
- Avoid per-emission `String` allocation: prefer `&'static str`,
  `Cow<'static, str>` or cached attributes. Cache instruments too.
- Does middleware/client instrumentation already emit retry, timeout, breaker
  or request signals? Remove duplicate call-site metrics/logs and reuse it.
- Require a named consumer for new spans. If no backend consumes them, prefer
  regular tracing APIs, metrics or logs. Per-request measurements are metrics;
  logs describe events an operator must read.
- Reuse an existing tracing log front end rather than boxing/storing a local
  logger provider.
- Route all user/customer data through repository classification/redaction
  **before** emission.

## Proof and coverage

Name the exact signal/attributes and violated convention or sibling, with
decisive emitted evidence, operator impact and corrected instrumentation.
Telemetry tables/dashboards can prove consumer dependence for a breaking
rename, not the emitted definition.

If comparable components instrument behavior but this change adds none, note
that once; do not demand unnecessary instrumentation.

Coverage: signals reviewed and names/attribute sets unconfirmed by definitions
or telemetry tests.
