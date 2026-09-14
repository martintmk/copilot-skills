---
name: review-consistency
description: >
  Review changed code and documentation for conflicting contracts, stale
  claims, examples or defaults, documentation gaps and conflicting documents.
  Use for "check code and docs agree", "review documentation consistency",
  or when review-lens routes a change to documented behavior here.
  Not for prose styling, naming preferences, runtime defect hunting or
  output-only API audits.
---

# Review Consistency

Check that implementation, public docs, examples and guides describe the same
contract. Review changed claims and their immediate counterparts, not the whole
repository unless asked.

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md); reuse supplied context.

## Procedure

1. **Map the affected claims.** Pair changed behavior with its existing docs,
   and changed docs with the relevant implementation and related documents.
   Include unchanged counterparts that the change could make stale.
2. **Compare like with like.** Match revision, version, features, target and
   audience. Do not flag a historical changelog or an explicitly scoped
   exception as a current contradiction. Reuse available public-docs bundles;
   request a fresh `review-public-docs` worker only when authoritative
   reachable-public-item documentation is needed.
3. **Check concrete agreement.** Compare signatures and examples, defaults and
   allowed values, units and limits, feature gates, lifecycle/ordering, and
   error or panic guarantees. Follow re-exports, wrappers and configuration
   sources far enough to establish the actual reachable contract. Compare
   related docs for incompatible instructions or descriptions of that contract.
4. **Resolve intent before choosing the fix.** Neither code nor prose is
   automatically correct. Use trusted requirements and baseline evidence to
   decide which is stale. If intent cannot be established, ask a focused
   question rather than silently changing the documented contract to fit code.
5. **Prove and scope the mismatch.** Quote the two conflicting claims or a
   documented claim and its decisive code/probe evidence. Use the common
   reproduction rules for runtime assertions. Recommend the smallest coherent
   correction across the affected surfaces, not just the first stale sentence.

## Boundaries

- Own semantic agreement, not general API redesign (`review-api-design`),
  runtime defects (`review-correctness`), test adequacy (`review-tests`) or
  naming preferences (`review-naming`). Pass a shared root cause to its owner
  with the evidence; do not emit a second finding for the same fix.
- Docs retrieval supplies evidence, not a judgment. Do not feed source findings
  into `review-public-api` or bypass its output-only boundary.
- For generated docs, identify the authoritative source or generation step;
  do not request hand-edits to derived files or flag generated README
  wording/casing. Describe current behavior, not speculative future APIs.

## Documentation coverage

- New public items need minimal docs on their reachable public surface.
- A new feature deserves a short, readable example (roughly 100 lines); use
  integration tests for exhaustive scenarios and keep crate-doc example or
  extension lists aligned with what exists.
- Keep design docs focused on tenets, constraints and an API sketch, without
  turning this into a prose-styling pass.

## Findings

Name both locations and the concrete consumer consequence under **Why this
matters**; give the correction and affected surfaces under **Suggested fix**.
Static contradictions need precise excerpts, not a test run for its own sake.
Do not invent certainty when the behavior or intended contract is unknown.

Coverage: claims/documents compared, public-doc/example gaps, applicable
configuration, and unresolved intent or unverified behavior.
