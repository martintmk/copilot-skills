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

Compare changed code/docs claims with their immediate counterparts, including
unchanged ones made stale. Do not audit the whole repository unless asked.

Follow [shared context](../review-lens/review-context.md) and the
[findings contract](../review-delivery/findings-contract.md).

## Procedure

1. **Map claims:** pair changed behavior with docs, and changed docs with
   implementation and related documents.
2. **Match scope:** revision, version, features, target and audience must agree.
   Historical changelogs and explicitly scoped exceptions are not current
   contradictions. Reuse matching public-docs bundles; request fresh
   `review-public-docs` retrieval when authoritative reachable-public-item docs
   are needed.
3. **Compare contracts:** signatures/examples, defaults/allowed values,
   units/limits, feature gates, lifecycle/ordering and error/panic guarantees.
   Follow re-exports, wrappers and configuration sources to the reachable
   contract; compare related documents' instructions too.
4. **Resolve intent:** neither code nor prose wins automatically. Use trusted
   requirements and baseline evidence. Unknown intent warrants a focused
   question, never weakening docs to fit implementation.
5. **Prove and correct:** quote both conflicting claims, or the documented claim
   and decisive code/probe evidence. Apply shared reproduction rules to runtime
   assertions. Recommend the smallest coherent correction across all affected
   surfaces.

## Specialist checks and boundaries

- New public items need minimal reachable docs. New features need readable
  examples (roughly 100 lines); exhaustive scenarios belong in integration tests.
  Keep crate-doc example/extension lists current, and design docs focused on
  tenets, constraints and an API sketch, not prose styling.
- Identify generated docs' authoritative source/generation step. Never request
  derived-file hand-edits or flag generated README wording/casing. Assess current
  behavior, not speculative APIs.
- Route API redesign, runtime defects, test adequacy and naming preferences to
  `review-api-design`, `review-correctness`, `review-tests` and `review-naming`.
  Supply shared-root-cause evidence without duplicating findings.
- Retrieval provides evidence, not judgment. Never feed source findings into
  `review-public-api` or bypass its output-only boundary.

## Proof and coverage

Give both locations/excerpts, concrete consumer impact and the correction's
affected surfaces. Static contradictions need no gratuitous execution; retain
uncertainty about behavior or intent.

Coverage: claims/documents compared, public-doc/example gaps, configuration,
unresolved intent and unverified behavior.
