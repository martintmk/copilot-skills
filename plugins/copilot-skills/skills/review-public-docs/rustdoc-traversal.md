# Schema-aware Rustdoc Traversal

JSON parsing reference for `review-public-docs`, not a review. Generation,
matching, baselines and bundles belong to [retrieval](SKILL.md).

## Inspect schema without dumping it

Check `.format_version` first. Routes below describe format 61; adapt to actual
schema or return `blocked` with decisive diagnostics for uninterpretable required
fields. Parser failure never means undocumented.

```text
jq '{format_version, root, crate_version, local_path_count: ([.paths[] | select(.crate_id == 0)] | length)}' <rustdoc-json>
```

- `.index`: local records keyed by ID, with `name`, `docs`, `links`, `attrs`,
  `deprecation`, and kind-keyed `inner`.
- `.paths`: `{path, kind, crate_id}` for local **and foreign** IDs; local
  `crate_id == 0`. Foreign paths without `.index` records are valid.
- Numeric `.root` needs `.index[.root|tostring]`; stringify **every** indexed ID.
  IDs are build-local, never cross-revision identity.
- Fields/methods/trait items lack `.paths`; traverse owners. Enum variants have
  paths and owner links.
- `#[deprecated]` uses `deprecation` with since/note, not `attrs` or a boolean.
  Attributes mix strings (`"non_exhaustive"`) and objects
  (`{"repr":{"kind":"c",...}}`); never blindly join as text.
- `.inner.struct.kind`: `{"plain":{...}}`, `{"tuple":[ids]}` or bare `"unit"`.
  Skip private tuple-field `null` holes **without renumbering** positions,
  including variant tuples.
- Public output omits `#[doc(hidden)]`; absence alone proves no cause. Apply
  retrieval's resolution statuses.

## Resolve exact public associations

Match fully qualified paths exactly, never suffixes or foreign namesakes. Resolve
members from confirmed local owners; check kind/name and governing trait.
Bare `new` cannot silently become foreign `LazyLock::new`; multiple local
owner/trait matches are `ambiguous`.

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

Read member kind/docs/attrs/deprecation/links; associated constants/types are not
methods. Adapt unsupported owner routes to detected schema or report gaps,
never "no members".

**Re-exports:** `.paths` omits some aliases. Resolve module use/re-export records
to confirm alias/target, retaining local re-export and relevant target docs.
The seed below resolves canonical owners, not aliases. Unconfirmed aliases need
known original paths and explicit gaps, not silent substitution or complete
inventory claims. Foreign targets may lack local docs; keep local re-export docs
and external-doc limitations.

Normalize canonical and confirmed alias/member associations within builds, then
compare public identities across builds, resolving link IDs first. Raw IDs/spans
are not changes. Incomparable signatures/aliases remain uncertain candidates,
never excluded by changed-file location.

## Scoped extraction reference

List exact canonical **owner paths**. Apply this `jq` seed with
`--argjson owners '["my_crate::Thing"]'`, `--argjson foreign_traits '[]'` and
`-f <filter-file>` to matching JSON. Keep filters/intermediates outside the
reviewed repository; never dump all items into caller context.

The seed expands owner/member records, not final bundles or re-export
associations. Resolve requested members/aliases/closure above; render only
requested/governing records, not all sibling results. Explicitly select needed
foreign traits (e.g. `core::fmt::Display`) to avoid blanket/derived-impl noise.

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

Account for every request: missing seed/link records are gaps, not absence
proof. Member links may require owner traversal without `.paths`. Foreign
`Send` may resolve to `core::marker::Send` without local docs: return path and
external-doc limitation, not `undocumented`; never fetch online docs.

Retrieval owns closure selection/reporting. Return each selected context's full
text once, including crate docs; neither recursively expand unrelated links nor
trim for guessed findings. The consumer chooses decisive excerpts.
