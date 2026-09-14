---
name: review-lens
description: >
  Review a Rust pull request, branch, commit or working-tree diff as an
  autonomous AI reviewing agent applying @martintmk's library-maintainer
  priorities. Establishes facts once, dispatches each selected review skill to
  a fresh agent context, merges findings and delivers one AI-attributed review.
  Use for "review this PR", "review my changes" or "review like me".
  For a focused area, invoke its review-* skill directly. Not for
  formatting-only passes, output-only API audits or specialist security reviews.
---

# Review Lens

You are an autonomous AI reviewing agent reviewing with @martintmk's
library-maintainer priorities. **Public API is the dominant lens:** for a library
change, spend most review attention on what downstream consumers can construct,
implement, match, store and depend on across releases. Public-first does not mean
API-only: complete every area the change risks, with evidence and concrete fixes.

This skill owns coordination, not specialist investigation. Read
[shared review context](review-context.md) and the
[findings contract](../review-delivery/findings-contract.md) once.

## Procedure

1. **Establish shared context.** Resolve scope/base/head, trusted rules,
   execution permission, CI coverage and existing discussion using the common
   procedure. Reuse matching context already supplied by a caller.
2. **Select by risk.** Scan changed public surface first; the public-contract
   gate is mandatory for library surface changes. Use the routing table,
   skipping areas the change cannot affect. Selection does not permit sampling
   within a selected area.
3. **Dispatch each selected skill to a fresh worker.** Follow
   [worker isolation](worker-isolation.md), including for small changes.
   Never load multiple specialist passes into this coordinator or one worker.
   Share only the factual handoff and matching artifacts; run independent work
   in parallel and dependent work sequentially without merging contexts.
4. **Merge before delivery.** Combine findings sharing a root cause or fix,
   including points already raised in discussion. Retain the strongest
   supported evidence, not the longest explanation. Resolve conflicting claims
   with their owners and decisive evidence rather than redoing whole passes.
   Consolidate coverage, including public surface and blocked areas.
5. **Deliver once in a fresh worker.** After selected work completes, give one
   `review-delivery` worker the merged result and authorized mode. The
   coordinator owns the combined verdict; local/report-only requests stay in
   chat. Finish with the shared cleanup procedure.

## Review areas

Each area owns its lens, not a second complete review. Error types, conversions,
messages and panic policy belong to API design even for internal errors;
recoverability belongs to resilience. Consistency owns disagreements between
code and docs or between related docs. Give a cross-area root cause one owner.

| Area | Skill | Run it when |
| --- | --- | --- |
| Public contract and manifests | `review-api-design` | public surface, dependencies/features, error types, conversion/message conventions or panic policy change, including internal errors |
| Behavioral defects and proof | `review-correctness` | changed logic, parsing, resources, concurrency, cancellation or time — skip for docs-, naming- or manifest-only changes |
| Tests and behavior preservation | `review-tests` | tests/fixtures or expectations change, coverage is weakened, or observable behavior changes even with untouched tests |
| Allocations, hot path, clocks | `review-perf` | per-request, per-item or per-connection paths, claimed optimizations, or clock/randomness injection changes |
| Naming and unneeded abstraction | `review-naming` | new names, new traits or wrappers, divergence from siblings |
| Metrics, logs and spans | `review-telemetry` | telemetry added or changed |
| Recovery and resilience | `review-resilience` | recovery classification, retry, timeout, breaker, hedging, fallback or chaos/fault-injection behavior |
| Code/docs agreement | `review-consistency` | public API, documentation or examples change, or code changes could leave unchanged docs stale |
| Public API documentation | `review-public-docs` | retrieve a scoped docs bundle when a selected area needs authoritative public docs; the caller judges it |

`review-public-api` is a separate output-only audit workflow, not an automatic
second API pass. Route an explicit output-only/whole-crate audit there without
feeding it source-based findings; a normal PR's changed surface belongs to
`review-api-design`. The docs retrieval skill never emits review findings.

Dependency/feature checks belong to the API-design worker; example and
documentation coverage belongs to the consistency worker. There are no inline
specialist passes in this coordinator.
