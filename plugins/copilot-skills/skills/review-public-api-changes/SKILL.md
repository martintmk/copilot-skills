---
name: review-public-api-changes
description: >
  Produce the mandatory Review Lens report that classifies a Rust library
  change as breaking public API changes, public API additions, or no API
  changes. Uses matching cargo-public-api captures and shows additions as
  concise Rust signatures. Requires human-review-required for every public API
  change. Use for Review Lens or a focused public API change report. Not for
  API design advice, source review, docs quality, runtime behavior or posting.
---

# Review Public API Changes

Tell the reviewer exactly how the exported Rust surface changed. This is a
classification report, not an API design review. Run it for every Review Lens
review, even when no Rust source changed.

Use the same full `cargo public-api` captures prepared for
[`review-public-api`](../review-public-api/SKILL.md). The coordinator owns
package discovery, revision setup and capture commands. Do not rebuild that
setup inside this skill.

## Evidence boundary

- Compare matching base and head captures for every affected library. Keep the
  package, features, target, toolchain and build flags identical.
- Use the supplied package-presence facts for added, removed, moved or renamed
  libraries. A failed capture never proves an empty API.
- If the coordinator's complete package inventory proves that no library is
  affected, report **No API Changes** without requesting empty captures.
- Read public API output only. Do not inspect source, manifests, tests, source
  diffs or rustdoc to infer changes.
- Report only. Never edit, post, vote or change labels.

## Classify the change

1. Diff the complete public surfaces.
2. Group related lines into consumer-visible items. Do not dump unchanged
   blanket or auto-trait impls.
3. Classify each changed item:

   | Result | Include |
   | --- | --- |
   | **Breaking Changes** | Removed items, reduced visibility, changed signatures, stricter bounds, removed impls, newly required trait items, and additions that break exhaustive construction or matching. Show the old and new signatures when both exist. |
   | **Public API Additions** | New callable, constructible, implementable or matchable surface that is not already listed as breaking. Show concise Rust signatures. |
   | **No API Changes** | Base and head expose the same public surface in every reviewed configuration. |

Breaking changes take priority, but a report may contain both **Breaking
Changes** and **Public API Additions**. Do not describe a changed signature as
an addition merely because the new signature appears as an added line.

For additions, use valid Rust-shaped snippets copied from the public API
output. Group related methods or fields in one small block. Include every
meaningful addition or family, but omit generated impl noise that does not give
consumers a new operation.

## Human review decision

Set `human-review-required` to **required** when the report contains either
**Breaking Changes** or **Public API Additions**. Intentional and approved API
changes still require the label. Set it to **not required** only for **No API
Changes**.

If a required capture or package-presence fact is missing, return `could not
review` and name the missing evidence. Do not claim **No API Changes** and do
not make a label decision from incomplete output.

## Report

Always return one report. Start with the attribution required by the
[findings contract](../review-delivery/findings-contract.md), then use this
shape:

````markdown
**Posted by an AI agent**

### Public API changes

**Public API Additions**

```rust
pub fn parse(input: &str) -> Result<Value, ParseError>;
```

Human review label: required.

Coverage: compared `example` at base and head with all features on the host
target. Status: done.
````

Use **No API Changes** as the complete body result when nothing changed. Keep
the prose short. Exact public paths and signatures are more useful than a
general explanation.
