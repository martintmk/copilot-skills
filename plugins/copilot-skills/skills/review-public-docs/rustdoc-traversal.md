# Schema-aware Rustdoc Traversal

Reference for the JSON parsing stage of `review-public-docs`, not an independent
review. Generation, configuration matching, baseline semantics and the bundle
contract remain in [the retrieval procedure](SKILL.md).

## Inspect schema without dumping it

Check `.format_version` before traversal. The facts and field routes below were
established for format 61; adapt to the actual schema, and return `blocked` with
a decisive format diagnostic if required fields cannot be interpreted. Do not
silently turn a parser failure into "undocumented".

```text
jq '{format_version, root, crate_version, local_path_count: ([.paths[] | select(.crate_id == 0)] | length)}' <rustdoc-json>
```

- `.index` holds local item records keyed by ID: `name`, `docs`, `links`,
  `attrs`, `deprecation` and `inner` keyed by item kind.
- `.paths` maps IDs to `{path, kind, crate_id}` for local **and foreign** items.
  `crate_id == 0` identifies the inspected crate. Foreign IDs can have paths
  without any `.index` record; this is not an extraction failure.
- `.root` is numeric in this schema: use `.index[.root|tostring]` for crate docs,
  and convert every ID to a string when indexing. IDs are build-local, not
  cross-revision identity.
- Fields, methods and trait items are **absent from `.paths`**; traverse their
  owner. Enum variants **do** have path entries as well as owner links.
- `#[deprecated]` populates `deprecation`, not `attrs`; retain its since/note
  fields, not just a boolean. `attrs` mixes strings such as `"non_exhaustive"`
  with structured objects such as `{"repr":{"kind":"c",...}}`; do not join
  entries blindly as text.
- `.inner.struct.kind` can be `{"plain":{...}}`, `{"tuple":[ids]}`, or the bare
  string `"unit"`. Tuple lists contain `null` holes for private fields; skip
  null IDs without renumbering positional field names. Variant tuple fields
  need the same handling.
- Public output omits `#[doc(hidden)]` items. An absent record alone does not
  identify why it is absent; use the retrieval contract's resolution rules.

## Resolve exact public associations

For fully qualified requests, match the complete public path, not a suffix or
same-named foreign item. Resolve member requests from a confirmed local owner,
then verify member kind/name and any governing trait. A bare `new` must not
silently resolve to a foreign `LazyLock::new`; multiple local owner/trait
matches are `ambiguous`.

| Target | Format-61 route |
| --- | --- |
| Type/trait/module/fn/alias | `.paths[id]` then `.index[id]` |
| Named struct field | `.inner.struct.kind.plain.fields[]` |
| Tuple struct field | `.inner.struct.kind.tuple[]`, skipping null IDs |
| Union field | `.inner.union.fields[]` |
| Inherent associated item | Owner's `.inner.{struct,enum,union}.impls[]`, impl with `.inner.impl.trait == null`, then `.inner.impl.items[]` |
| Trait impl | Same owner impls, with non-null `.inner.impl.trait`; retain governing trait identity |
| Enum variant | `.inner.enum.variants[]` or variant's `.paths` entry |
| Variant field | Variant's `.inner.variant.kind.struct.fields[]` or `.inner.variant.kind.tuple[]` |
| Trait item | `.inner.trait.items[]` |
| Doc link | `.links`: link text to ID; resolve via `.paths` or confirmed local member ownership |
| Crate docs | `.index[.root|tostring].docs` |

Read each member's own kind, docs, attributes, deprecation and links; associated
constants/types are not methods. If impl/member ownership for another item kind
is unsupported by these routes, adapt using the detected schema or report the
gap rather than asserting there are no members.

**Re-export caveat:** `.paths` alone does not enumerate every alias path. Resolve
module use/re-export records in the detected schema to confirm the requested
alias and target, retaining alias/re-export docs as well as relevant target
docs. The seed extractor below emits canonical paths, not alias associations.
If an alias cannot be confirmed, report the known original path and the
unresolved alias; never silently substitute it or claim a complete change
inventory. Foreign re-exports may resolve to a foreign path without local target
docs; retain any local re-export docs and name the external-doc limitation.

For changes, compare canonical and confirmed alias/member associations within
each build, then compare normalized public identities across builds. Resolve
link IDs to those identities before comparison; raw IDs and source spans can
change without any API/docs change. If a signature or alias cannot be compared,
retain it as an uncertain candidate rather than excluding it by changed-file
location.

## Scoped extraction reference

Build an exact list of canonical **owner paths** before expanding records. The
following `jq` seed filter uses `--argjson owners '["my_crate::Thing"]'` and
`--argjson foreign_traits '[]'`; apply it with `-f <filter-file>` to the matching
JSON. Keep the filter and intermediate records outside the reviewed repository.
Do not run an all-items extraction and dump it into the caller's context.

This extracts owner/member records for subsequent selection, not a finished
bundle or a re-export resolver. Resolve requested members, aliases and closure
using the rules above; render only requested/governing records, not every
sibling emitted by the seed. Include foreign trait paths explicitly in
`foreign_traits` when needed (for example `core::fmt::Display`); otherwise
blanket and derived standard-library impls can swamp the bundle.

```jq
def idx($c; $id): $c.index[$id|tostring];
def record($c; $id):
  idx($c; $id) | select(. != null) | . as $item
  | {
      id: $id, name: .name, kind: (.inner | keys[0]),
      docs: .docs, attrs: (.attrs // []), deprecation: .deprecation,
      links: [
        ($item.links // {}) | to_entries[]
        | {text: .key, target_id: .value,
           target: ($c.paths[.value|tostring].path
                    | if . == null then null else join("::") end)}
      ]
    };
def fields($item):
  ( ($item.inner.struct?.kind?.plain?.fields)
  // ($item.inner.struct?.kind?.tuple? | if . then map(select(. != null)) else null end)
  // ($item.inner.union?.fields)
  // [] );
def vfields($item):
  ( ($item.inner.variant?.kind?.struct?.fields)
  // ($item.inner.variant?.kind?.tuple? | if . then map(select(. != null)) else null end)
  // [] );
def impls($item):
  ( ($item.inner.struct?.impls) // ($item.inner.enum?.impls)
  // ($item.inner.union?.impls) // [] );
. as $c
| [
    $c.paths | to_entries[]
    | select(.value.crate_id == 0)
    | . as $p
    | (.value.path | join("::")) as $path
    | select($owners | index($path))
    | idx($c; $p.key) as $item
    | select($item != null)
    | record($c; $p.key) + {
        canonical_path: $path,
        members: (
            [fields($item)[] | record($c; .)]
          + [($item.inner.enum?.variants // [])[] | . as $vid
             | record($c; $vid) + {
                 members: [vfields(idx($c; $vid))[] | record($c; .)]
               }]
          + [vfields($item)[] | record($c; .)]
          + [($item.inner.trait?.items // [])[] | record($c; .)]
          + [impls($item)[] | idx($c; .) | select(. != null)
             | select(.inner.impl.trait == null)
             | .inner.impl.items[] | record($c; .)]
        ),
        trait_impls: [
          impls($item)[] | . as $iid | idx($c; $iid)
          | select(. != null and .inner.impl.trait != null)
          | . as $impl
          | ($c.paths[.inner.impl.trait.id|tostring]) as $trait
          | (($trait.path // []) | join("::")) as $trait_path
          | select($trait.crate_id == 0 or ($foreign_traits | index($trait_path)))
          | record($c; $iid) + {
              trait: $trait_path,
              items: [$impl.inner.impl.items[] | record($c; .)]
            }
        ]
      }
  ]
```

Account for every request after extraction: a missing seed record or unresolved
link target is a resolution gap, not proof of absence. Member doc links often
need owner traversal because their targets lack `.paths` entries. A foreign
link such as `Send` can resolve to `core::marker::Send` with no local doc text;
return that path and an external-doc limitation, not `undocumented`, and do not
fetch online docs.

The retrieval procedure owns documentation-closure selection and reporting.
Return full text for each selected contextual record once, including crate docs;
do not expand unrelated linked items recursively or trim text based on a guessed
finding. Only the docs consumer decides which excerpts answer its question.
