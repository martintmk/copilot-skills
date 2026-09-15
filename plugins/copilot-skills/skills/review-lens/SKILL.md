---
name: review-lens
description: >
  Review a Rust pull request, branch, commit or working-tree diff as an
  autonomous AI reviewing agent applying @martintmk's library-maintainer
  priorities. Establishes facts once, dispatches every review sub-skill to
  a fresh agent context, merges findings and delivers one AI-attributed review.
  Use for "review this PR", "review my changes" or "review like me".
  For a focused area, invoke its review-* skill directly. Not for
  formatting-only passes, output-only API audits or specialist security reviews.
---

# Review Lens

Coordinate, do not perform specialist passes. **Public API dominates** library
review: prioritize what consumers can construct, implement, match, store and
depend on across releases. Risk changes attention within each pass, never the
required roster.

## Procedure

1. **Establish facts once** using [shared context](review-context.md); reuse
   matching caller-supplied facts. Inventory affected packages/configurations
   and scan changed public surface first. Read
   [package comparison](package-comparison.md) to establish comparison scope
   before API/docs extraction.
2. **Dispatch all ten required specialists**, even for small, docs-only,
   naming-only or manifest-only changes. Read [worker isolation](worker-isolation.md)
   before dispatch; give each fresh worker its permitted factual handoff and
   the coverage-record contract below. No inline or combined specialist passes.
3. **Merge by root cause/fix**, including existing discussion. Keep the strongest
   supported evidence; resolve contradictions with the owners and decisive
   evidence, not repeated whole passes. Read the
   [findings contract](../review-delivery/findings-contract.md) when merging;
   preserve it in intermediate and final output. Consolidate public-surface
   coverage and limitations, then enforce the completion gate below.
4. **Refresh target/head immediately before delivery.** A completed review stays
   pinned to its snapshot unless the permitted descendant refresh below applies.
   Other movement requires a fresh review or blocked result, not stale publication.
5. **Deliver once:** after required work completes, dispatch one fresh
   `review-delivery` worker with merged findings, coverage manifest, the
   coordinator's combined verdict and authorized mode. Local/report-only work
   stays in chat. Finish with shared-context cleanup.

## Best-effort finding refresh

Only after the complete roster passes its completion gate may the coordinator
inspect `reviewedHead..currentHead` and current source to re-evaluate existing
merged findings. Never discover new findings or claim full coverage of new commits.
Require unchanged target/base, an ancestor reviewed head, and the complete exact
delta/current source. Retargeting, rewritten/non-descendant history or incomplete
evidence requires a fresh review or blocked result.

Classify **every** merged finding:

| Classification | Evidence and delivery action |
| --- | --- |
| `still-applies` | Evidence unaffected, or current source clearly retains the root cause; retain. |
| `resolved` | Root cause clearly fixed; omit. |
| `updated` | Root cause remains but evidence, wording or anchor changed; update and re-anchor against current head. |
| `uncertain` | Material evidence affected without a confident conclusion; omit and disclose. |

Record `findingRefresh`: reviewed/current heads, target/base, every classification
and inspected delta reference. Keep the original manifest pinned. Force
`COMMENT`-only/no ADO vote regardless of original verdict. State that full Review
Lens coverage ended at the reviewed head; only existing findings were
best-effort re-evaluated through current head. Blocked refresh evidence cannot
authorize publication.

## Required coverage

Every row is mandatory on every invocation, including output-only API and docs
retrieval. Each owns its area, not another full review; assign cross-area root
causes one owner. Direct focused requests retain only their requested workflow.

| Area | Skill | Responsibility on every run |
| --- | --- | --- |
| Public contract and manifests | `review-api-design` | public surface, dependencies/features, error types, conversion/message conventions and panic policy, including internal errors |
| Behavioral defects and proof | `review-correctness` | changed logic, parsing, resources, concurrency, cancellation and time |
| Tests and behavior preservation | `review-tests` | tests/fixtures, expectations, weakened coverage and observable behavior changes |
| Allocations, hot path, clocks | `review-perf` | per-request/item/connection costs, optimization claims and clock/randomness injection |
| Naming and unneeded abstraction | `review-naming` | new names, traits/wrappers and divergence from siblings |
| Metrics, logs and spans | `review-telemetry` | emitted signal contracts and instrumentation changes |
| Recovery and resilience | `review-resilience` | recoverability, retry, timeout, breaker, hedging, fallback and fault-injection behavior |
| Code/docs agreement | `review-consistency` | docs/example coverage, code/docs and related-doc disagreements, changed claims and stale unchanged docs/examples |
| Output-only public contract | `review-public-api` | matching `cargo public-api` current surface/diff and mandatory isolated docs filtering |
| Public API documentation | `review-public-docs` | scoped rustdoc JSON bundle and explicit resolution/coverage; consumers judge it |

`review-public-api` stays output-only/report-only: supply package/configuration,
pinned revisions, execution permission, matching artifacts and factual
`packageComparison`, never source, manifests, source diffs, docs text or other
reviewers' findings. Its isolated docs-filtering stage is mandatory even for
an empty applicable report; return only the filtered area result for merging.

The isolated docs worker supplies data, never findings/verdict. Route its bundle
to API-design/consistency or the isolated API filter, never to the output-only
API worker as candidate evidence. Reuse matching captures/bundles, not workers;
reuse saves builds, not required passes.

## Coverage manifest and completion gate

Keep one returned `coverageManifest` record per required skill: `skill`, actual
`workerId`, exact pinned `snapshot`, `status`, concise `evidence`/artifact reference.
Never invent IDs, substitute coordinator passes or invent findings. No unrelated
probes to fill rows.

- `completed`: worker finished its scoped procedure, including required
  extraction/comparison/filtering, and returned findings/data or explicit
  no-findings. Supported one-sided package comparisons can complete under the
  package-comparison contract.
- `not-applicable`: dispatched worker established no applicable surface from
  stated, permitted evidence. Confirmed absence of Rust library packages can
  qualify API/docs; small/docs-only Rust changes alone cannot.
- `blocked`: required evidence, permission, tools, isolation or dependency is
  unavailable; failed workers and missing output also block, never succeed.

Set `reviewComplete=true` only when all ten records match one reviewed snapshot
and are `completed` or evidence-backed `not-applicable`. Publication requires
that snapshot still current, or the valid finding refresh above. Otherwise
return the decisive limitation, not approval or complete current-head coverage.
