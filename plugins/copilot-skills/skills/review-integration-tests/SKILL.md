---
name: review-integration-tests
description: >
  Produce the mandatory Review Lens report for Rust integration tests in each
  package's tests directory. Detects removed, weakened or changed expectations
  that introduce a breaking behavioral contract, distinguishes additive-only
  coverage, and requires human-review-required for confirmed behavioral
  breaks. Use for Review Lens or a focused integration-test change report. Not
  for inline unit tests, general coverage review, production correctness or
  posting.
---

# Review Integration Tests

Report how package integration tests changed and whether their established
behavioral contract was broken. Run this area for every Review Lens review,
including changes that do not touch integration tests.

A Rust integration test is a target below a package's `tests/` directory.
Include fixtures, snapshots and test-only configuration owned by those targets.
Leave inline unit tests, doctests and whole-diff behavior review to
[`review-tests`](../review-tests/SKILL.md).

## Procedure

1. Inventory base-to-head changes below every affected package's `tests/`
   directory. Include added, removed, renamed and modified test cases,
   assertions, snapshots, fixtures and discovery configuration.
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
   | **Breaking Behavioral Changes** | An established integration-test expectation was removed, weakened or changed. This classification applies even when the change is intentional or approved. |
   | **Integration Test Additions Only** | New tests, cases, assertions or fixtures were added and no existing expectation or execution path changed. |
   | **No Breaking Behavioral Changes** | Existing integration tests changed, but their observable expectations and execution coverage were preserved. |
   | **No Integration Test Changes** | No affected package changed files owned by its `tests/` targets. |

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

If base content, head content or test ownership cannot be established, return
`could not review` and name the gap. Do not make a clean claim or a label
decision from an incomplete inventory.

## Report

Always return one concise report. Start with the attribution required by the
[findings contract](../review-delivery/findings-contract.md), then use this
shape:

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
