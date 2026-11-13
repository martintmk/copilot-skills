---
name: review-public-api-changes
description: >
  Produce the mandatory Review Lens report that classifies a Rust library
  change as breaking public API changes, public API additions, or no API
  changes. Uses matching cargo-public-api captures and shows callable additions as
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
   Apply the [renderer limits](../review-public-api/SKILL.md#renderer-limits) before interpreting omissions as removed contracts.
   Retain both full captures and the complete delta once. Do not filter impls or collapse distinct signatures.
   A successful exact comparison can establish that a capture is unchanged in the selected configuration.
   It does not resolve known renderer gaps.
   Read every changed item and the unchanged context needed to interpret it, not every unchanged declaration twice.
   An unsuccessful or incomplete comparison cannot establish absence.
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

Use the [compact report rules](#report) after classifying the complete delta.

## Human review decision

Set `human-review-required` to **required** when the report contains either
**Breaking Changes** or **Public API Additions**. Intentional and approved API
changes still require the label. Set it to **not required** only for **No API
Changes**.

If required evidence is missing, return `could not review` and name the gap.
Keep any changes already proved. They still require the label.
Otherwise return `undetermined`, never `not required` or **No API Changes**.

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

Preserve every distinct break and meaningful addition or family without copying the complete API listing:

| Change | Visible detail |
| --- | --- |
| Removal or compatibility break | Exact public path, consumer effect and the smallest decisive before/after signature or member. Do not repeat unchanged fields. |
| Added callables | Concise signatures from the capture, grouped by public type or module. |
| Added related data types | Name each type and summarize its fields or variants as a family. Keep exact declarations in the retained delta. |
| Impl changes | Show changed bounds or traits when they affect compatibility or add an operation. Do not enumerate identical derived impls for every type. |
| Unchanged packages | One grouped no-change statement for the reviewed configuration. |

Use prose for abbreviated families. Never present an incomplete declaration as a complete Rust snippet.
Keep distinct contracts separate when grouping would hide a break, bound or consumer choice.
Record classifications against the retained delta, not another copy of its declarations.
Every changed item must have a disposition, even when its visible family is short.
Link details only through an authorized location the reader can access. Do not replace required classification or decisive evidence with an inaccessible link.
Keep inventories, rejected hypotheses and setup logs outside the public block.
