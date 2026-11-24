---
name: review-integration-tests
description: >
  Classify affected Rust integration-test contracts in package tests
  directories, including shared inputs and discovery configuration.
  Detects removed, weakened or changed expectations, distinguishes additive-only
  coverage, and requires human-review-required for confirmed behavioral
  breaks. Use when Review Lens identifies possible integration-test impact or
  for a focused report. Not for inline unit tests, general coverage review,
  production correctness or posting.
---

# Review Integration Tests

Report how affected package integration tests changed and whether their
established behavioral contract was broken.

This area covers targets below a package's `tests/` directory at either revision.
Include their fixtures, snapshots, helpers and test-only configuration, even when those inputs live outside `tests/`.
Leave inline unit tests, doctests and whole-diff behavior review to
[`review-tests`](../review-tests/SKILL.md).

## Procedure

1. Inventory the covered targets and their owned inputs at base and head for every affected package.
   Include added, removed, renamed and modified paths.
   Trace changed shared inputs to their consuming tests, including unchanged test files.
   Inspect target discovery and execution configuration, including `autotests`, `[[test]]`, target paths, `required-features` and `cfg`.
   Compare both revisions before treating a deleted or moved target as absent.
   Establish baseline expectations independently under the [test evidence rules](../review-tests/SKILL.md#baseline-scope-and-reading-order).
   The reviewer can choose reading order, but changed implementation and expectations cannot be the sole baseline proof.
   An unchanged-input classification needs complete ownership and discovery evidence, not a whole-consumer test baseline.
   If only product dependency versions change, inspect test-specific dependencies and execution selectors for effects on this inventory.
   Leave unresolved product behavior to the relevant source areas unless it changes a covered expectation, shared input or execution path.
2. Compare existing expectations before counting additions. Look for:
   - deleted cases or assertions without equivalent coverage;
   - exact values or errors replaced by broader matches;
   - changed expected output, errors, defaults, ordering, serialization, side
     effects or timing;
   - new ignores, conditions, retries, longer timeouts or narrower matrices;
   - fixtures or snapshots rewritten to accept behavior that the base rejected.
3. Classify the result:

   | Result | Meaning |
   | --- | --- |
   | **Breaking Behavioral Changes** | An established expectation was removed, weakened or changed, or its execution coverage was reduced. Intent or approval does not change this classification. |
   | **Integration Test Additions Only** | New tests, cases, assertions or fixtures were added and no existing expectation or execution path changed. |
   | **No Breaking Behavioral Changes** | Existing integration tests changed, but their observable expectations and execution coverage were preserved. |
   | **No Integration Test Changes** | Complete inventory proves that no covered test inputs or discovery/execution configuration changed. |

When additions and breaking changes coexist, report the break first and
summarize the additions separately. A rename or fixture refactor is not
additive-only unless existing discovery and assertions are unchanged.

The diff is enough to prove an explicit test-contract change. Claims about
actual runtime behavior require the same focused integration test at base and
head. If execution is unavailable, describe only what the expectations changed;
do not turn that limitation into **No Breaking Behavioral Changes**.

## Human review decision

Set `human-review-required` to **required** for **Breaking Behavioral Changes**.
Set it to **not required** for additions-only, behavior-preserving changes and
no integration-test changes.

If inventory, content or ownership is incomplete, return `could not review` and name the gap.
A proved behavioral break still requires the label.
Otherwise use `undetermined`, not a clean classification.

## Report

Return one concise report when this classification applies.
Start with the attribution required by the
[findings contract](../review-delivery/findings-contract.md).
The following shape is an example:

```markdown
**Posted by an AI agent**

### Integration tests

**Integration Test Additions Only**

- Added `tests/reconnect.rs` coverage for reconnecting after a transient error.
- Existing integration-test expectations and execution paths are unchanged.

Human review label: not required.

Coverage: reviewed integration-test targets for `example`. Status: done.
```

For a behavioral break, name the old expectation and the replacement. For no
changes, write **No Integration Test Changes** without filler.

Keep complete file inventories and unchanged assertions in artifacts.
Report only the meaningful contract delta and additions. A required human-review
signal is not itself a defect finding or grounds for a blocking vote.
