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

You are an autonomous AI reviewing agent reviewing with @martintmk's
library-maintainer priorities. **Public API is the dominant lens:** for a library
change, spend most review attention on what downstream consumers can construct,
implement, match, store and depend on across releases. Public-first does not mean
API-only: run every sub-review on every invocation, with evidence and concrete
fixes. Risk determines attention within a pass, never whether to dispatch it.

This skill owns coordination, not specialist investigation. Read
[shared review context](review-context.md) and the
[findings contract](../review-delivery/findings-contract.md) once.

## Procedure

1. **Establish shared context.** Resolve scope/base/head, trusted rules,
   execution permission, CI coverage and existing discussion using the common
   procedure. Reuse matching context already supplied by a caller.
2. **Plan complete coverage.** Use every entry in the required coverage table
   below, including for small, docs-only, naming-only and manifest-only changes.
   Inventory affected packages/configurations, establish their
   [baseline/head presence](package-comparison.md), and scan changed public
   surface first. Do not replace the full roster with a risk-selected subset.
3. **Dispatch every sub-review to a fresh worker.** Follow
   [worker isolation](worker-isolation.md), including for small changes.
   Never load multiple specialist passes into this coordinator or one worker.
   Share only the factual handoff permitted by each skill's evidence boundary;
   run independent work in parallel and dependent work sequentially without
   merging contexts. Include the coverage-record contract below in each handoff.
4. **Merge before delivery.** Combine findings sharing a root cause or fix,
   including points already raised in discussion. Retain the strongest
   supported evidence, not the longest explanation. Resolve conflicting claims
   with their owners and decisive evidence rather than redoing whole passes.
   Preserve each finding's attribution and bold title, with the issue/evidence
   under **Problem** and impact under **Why this matters**. Actionable findings
   use a diagnosis title and **Suggested fix**; design notes still require
   **Problem** but omit the fix. Clean summaries need no finding sections.
   Consolidate coverage, including public surface and blocked areas. Require a
   returned coverage record from every dispatched skill before calling the
   review complete; a clean partial roster is not a complete review.
5. **Reconcile head movement.** Immediately before delivery, refresh the target
   and head. If only the head advanced and the reviewed head is its ancestor,
   follow the best-effort finding refresh below instead of discarding the
   completed review or rerunning every specialist. Otherwise the completed
   review remains pinned to its original snapshot.
6. **Deliver once in a fresh worker.** After all required work completes, give
   one `review-delivery` worker the merged result, coverage manifest and
   authorized mode. The coordinator owns the combined verdict; local/report-only
   requests stay in chat. Finish with the shared cleanup procedure.

## Best-effort finding refresh

When a PR head advances after all required workers completed, the coordinator
may inspect the exact `reviewedHead..currentHead` diff solely to preserve,
remove or update existing findings. This is a narrow exception to specialist
isolation, not another review pass: do not search the new commits for new
findings or claim that they received the required roster.

The shortcut applies only when the target/base is unchanged, the reviewed head
is an ancestor of the current head, and the complete intervening diff and
current source are available. A changed target, rewritten/non-descendant
history or incomplete diff requires a fresh review or a blocked result.

Classify every merged finding:

- `still-applies`: the delta does not affect its evidence, or current source
  clearly retains the root cause.
- `resolved`: the delta clearly fixes the root cause; omit it from delivery.
- `updated`: the root cause remains but its evidence, wording or anchor changed;
  update and re-anchor it against the current head.
- `uncertain`: the delta affects material evidence and the coordinator cannot
  confidently decide; omit it and disclose the limitation.

Keep a `findingRefresh` record with reviewed/current heads, target/base,
classification for every finding and the inspected delta reference. Delivery
is `COMMENT`-only after a refresh, even when the original verdict supported
approval or changes requested. The summary must state that full Review Lens
coverage ended at the reviewed head and only existing findings were
best-effort re-evaluated through the current head.

## Required coverage

Each area owns its lens, not a second complete review. Error types, conversions,
messages and panic policy belong to API design even for internal errors;
recoverability belongs to resilience. Consistency owns disagreements between
code and docs or between related docs. Give a cross-area root cause one owner.

Every row is mandatory on every Review Lens invocation. This includes the
output-only API audit and docs retrieval, not just source-based reviewers.
Directly invoking a focused skill still runs only that requested workflow.

| Area | Skill | Responsibility on every run |
| --- | --- | --- |
| Public contract and manifests | `review-api-design` | public surface, dependencies/features, error types, conversion/message conventions and panic policy, including internal errors |
| Behavioral defects and proof | `review-correctness` | changed logic, parsing, resources, concurrency, cancellation and time |
| Tests and behavior preservation | `review-tests` | tests/fixtures, expectations, weakened coverage and observable behavior changes |
| Allocations, hot path, clocks | `review-perf` | per-request/item/connection costs, optimization claims and clock/randomness injection |
| Naming and unneeded abstraction | `review-naming` | new names, traits/wrappers and divergence from siblings |
| Metrics, logs and spans | `review-telemetry` | emitted signal contracts and instrumentation changes |
| Recovery and resilience | `review-resilience` | retry, timeout, breaker, hedging, fallback and fault-injection behavior |
| Code/docs agreement | `review-consistency` | changed claims and unchanged docs/examples the change could make stale |
| Output-only public contract | `review-public-api` | matching `cargo public-api` current surface/diff and mandatory isolated docs filtering |
| Public API documentation | `review-public-docs` | scoped rustdoc JSON bundle and explicit resolution/coverage; consumers judge it |

`review-public-api` remains output-only and report-only while participating in
the combined review. Give it package/configuration, pinned revisions, execution
permission and matching artifact paths, **not** source, manifests, source diffs,
docs text or other reviewers' findings. Its own isolated filtering stage remains
mandatory, even for a clean applicable API audit. Return its filtered area
result to the coordinator; only the final delivery worker posts.
Include the factual `packageComparison` record: a genuinely new crate uses a
logical empty baseline and real head artifacts, not a doomed baseline build.

Run `review-public-docs` in its own context. Share its matching bundle with the
API-design/consistency consumers or the isolated API filter, never as candidate
evidence to the output-only API worker. Reuse matching captures and bundles to
avoid duplicate builds, not to omit a required worker. The docs skill supplies
data, never findings or a verdict.

Dependency/feature checks belong to the API-design worker; example and
documentation coverage belongs to the consistency worker. There are no inline
specialist passes in this coordinator.

## Coverage manifest and completion gate

Keep an internal `coverageManifest` with one record per required skill:
`skill`, actual `workerId`, pinned `snapshot`, `status`, and a concise
`evidence`/artifact reference. Preserve the workers' returned coverage; do not
invent worker IDs or fill missing records with a coordinator-written pass.

- `completed`: the worker finished its scoped procedure, including required
  extraction/comparison/filtering, and returned findings/data or an explicit
  no-findings result. A proven added/removed package's supported one-sided
  comparison can complete; no artifact is required for its absent side.
- `not-applicable`: the worker ran and established that its lens has no
  applicable surface, stating the inspected, permitted evidence. This is not a
  pre-dispatch skip. A confirmed absence of Rust library packages can yield this
  result for API/docs workers; a small or docs-only Rust change cannot by itself.
- `blocked`: required evidence, execution permission, tools, isolation or a
  dependent stage is unavailable. Missing output and failed workers also block;
  they are never `not-applicable` or successful coverage.

All ten records must match one reviewed snapshot and be `completed` or
evidence-backed `not-applicable` before `reviewComplete=true`. Normal completed
review publication requires that snapshot to remain current. If the head later
advances, publication may instead use the best-effort finding refresh above:
the original manifest remains pinned, every existing finding gets a refresh
classification, and delivery is `COMMENT`-only with the partial-delta
limitation. Do not manufacture findings or run unrelated probes to fill a row.
If either the original roster or required refresh evidence is blocked, return
the decisive limitation without claiming complete current-head coverage.
