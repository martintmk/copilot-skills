# copilot-skills

Personal [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/use-copilot-agents/use-copilot-cli)
skills, packaged as a cross-engine Agency-style plugin so they stay in sync
across machines.

## Install

With Agency:

```
agency plugin install market:copilot-skills@martintmk/copilot-skills
```

For a single Agency session without installing:

```
agency copilot --plugin market:copilot-skills@martintmk/copilot-skills
```

With GitHub Copilot CLI directly:

```
copilot plugin marketplace add martintmk/copilot-skills
copilot plugin install copilot-skills@martintmk-skills
```

Update later with:

```
copilot plugin update copilot-skills@martintmk-skills
```

## Skills

| Skill | What it does |
| ----- | ------------ |
| [`pr-review-radar`](skills/pr-review-radar/SKILL.md) | Scans configured GitHub and Azure DevOps repositories for open PRs the user has not reviewed, selects recent, foundational-API, infrastructure, and mentioned PRs, then sends only newly discovered matches to the user's Teams self-chat. |
| [`review-lens`](skills/review-lens/SKILL.md) | Orchestrates a Rust review of a PR, branch, commit or working-tree diff. Establishes scope, base revision, trust and CI baseline, reviews dependencies/features and docs inline, then routes the areas a change actually risks to the specialized `review-*` skills and delivers one AI-attributed review. |
| [`review-api-design`](skills/review-api-design/SKILL.md) | Reviews a change's public contract from the diff and source: visibility and layering, semver cascade, constructors, builders and defaults, strong types and enums, trait sealing, macros as public API, and error and panic conventions. |
| [`review-correctness`](skills/review-correctness/SKILL.md) | Hunts behavioral defects — version gates, boundaries, resource models, cancellation and drop safety, time arithmetic, round-trip claims — and owns the shared verification discipline that every other review skill follows. |
| [`review-tests`](skills/review-tests/SKILL.md) | Reviews Rust changes for deleted or weakened behavioral tests, unjustified behavior changes, non-idiomatic test code, and missed opportunities to reuse supported test utility features. |
| [`review-resilience`](skills/review-resilience/SKILL.md) | Reviews Rust changes for missing or incorrect `recoverable::Recovery` classification using the selected version's recipes, then finds retry, timeout, breaker, hedging, fallback, or chaos behavior that should use `seatbelt`. |
| [`review-perf`](skills/review-perf/SKILL.md) | Reviews avoidable allocations, hot-path cost, per-call dispatch that could be decided once, and direct clock/sleep calls where an injectable clock exists — with honest benchmark reporting. |
| [`review-naming`](skills/review-naming/SKILL.md) | Reviews names and shapes that diverge from their siblings, padded or unit-duplicating names, and new traits, wrappers or layers that a small change to an existing type would remove. |
| [`review-telemetry`](skills/review-telemetry/SKILL.md) | Reviews metrics, logs and spans against OpenTelemetry semantic conventions, guarding stable dimension sets, unbounded cardinality, per-emission allocation, and instrumentation a library already provides. |
| [`review-public-api`](skills/review-public-api/SKILL.md) | Reviews a Rust library's exported contract from `cargo public-api` output, then uses an isolated rustdoc JSON pass to filter claims refuted by the API docs. It applies common idiomatic Rust API practices first, then API-visible Pragmatic Rust Guidelines; small PRs are scoped to changed public items. |
| [`review-public-docs`](skills/review-public-docs/SKILL.md) | Retrieves a crate's public API documentation from cargo rustdoc JSON, scoped to a change (PR diff, branch, commit, working tree) or an explicit item list. Returns a compact docs bundle instead of raw JSON, so it can be reused as a retrieval primitive by other skills and agents. |
| [`review-delivery`](skills/review-delivery/SKILL.md) | Posts an AI-attributed review to a GitHub PR, Azure DevOps PR, or a local report: comment shape, precise anchoring, severity labels, verdict, and the posting mechanics that bite. Shared by every review skill. |

### `review-lens`

Derived empirically from a year of real PR review comments, so public API design
is the dominant concern rather than one checklist item among many. It is now an
**orchestrator**: it reads the repository rules from the base revision,
establishes intent and the correct base, decides trust and treats green CI as the
baseline instead of re-running it, then selects only the review areas a change
actually risks and delegates them to the specialized `review-*` skills. It keeps
the two short manifest- and prose-shaped lenses inline — dependencies and
features, and tests/examples/docs at a glance — plus the `Repository adaptation`
section carrying workspace-specific instincts for `microsoft/oxidizer` and
`ox-sdk`, generalising everywhere else.

Splitting the areas out keeps a small PR from loading the whole review corpus:
the orchestrator is ~150 lines instead of 526, and a docs-only change never pays
for the correctness, perf and telemetry lenses.

Findings are posted as a single structured review through `review-delivery`,
explicitly attributed to an AI agent so none read as if a human maintainer wrote
them, with each one anchored to the code it is about.

Trigger it with "review this PR", "review my changes", or "review like me".

### `pr-review-radar`

Tracks open PRs across a saved list of GitHub and Azure DevOps repositories. It
filters out drafts, the user's own PRs, and PRs the user has already reviewed,
then selects PRs opened in the last seven days plus older PRs involving
foundational APIs, infrastructure fixes, or an explicit mention of the user.

Reports are sent through `teams-self-message` as structured Teams HTML with
category headings, compact PR entries, clickable links, and a specific
`Why review` explanation for every item. A persistent local state file records
PR URLs only after successful delivery, so later scans send only PRs that have
never appeared in an earlier report. Trigger it with "find PRs I should review",
"track open PRs in these repositories", or schedule that prompt for a recurring
digest.

### `review-api-design`

The largest cluster in the review corpus — 178 comments across 83 PRs — covering
visibility and layering ("does it need to be public?"), dependency-in-signature,
macros as public API and `#[doc(hidden)]` plumbing, strong types, the enum test,
builders and constructor families, coherent defaults, the panic test for
configuration constructors, privacy escape hatches, `Send`/`Sync`/`Clone`, opaque
return contracts, trait sealing, and the semver cascade across crates. Error type
and panic conventions are folded in, since they are the same contract decision.

It reads the diff, source, manifests and callers, which is what separates it from
`review-public-api` — that skill reviews an existing surface from `cargo
public-api` output and never opens source.

Trigger it with "review the public API of this change" or "what does this commit
downstream consumers to?".

### `review-correctness`

The forensic pass, and the home of the shared **verification discipline** every
other review skill follows: attribute against the base rather than a green head,
reproduce with the smallest faithful test, falsify before raising, check the real
configuration matrix, quote exact values, and state what you could not check. Its
defect classes are drawn from real findings — version gates on decoding, boundary
and representation limits, resource and free-list models, cancellation and drop
safety, thread affinity, time arithmetic, and round-trip claims.

No adequate reproduction means no correctness finding.

Trigger it with "find bugs in this change" or "is this logic correct?".

### `review-perf`

Classifies the path before spending complexity on it, then looks for needless
allocation for static data, `to_string()`/`clone()` round-trips, per-call dynamic
switching on values that never change, contention, and unbounded input handling.
It also flags `Instant::now()`, `SystemTime::now()` and raw sleeps where an
injectable clock exists — a testability finding as much as a performance one.
Claims need numbers, reported honestly with the range and significance.

Trigger it with "review this for allocations" or "is this hot path efficient?".

### `review-naming`

Treats divergence from siblings as a finding in its own right, quoting the family
convention as the evidence. Covers cross-crate alignment, concise non-padded
names, units already carried by a type, capability-versus-action naming,
terminology precision, and the rename's full blast radius. Its second half asks
whether an abstraction earns anything: could adding `Clone` or a method to an
existing type remove the need for a new trait, wrapper or layer?

Trigger it with "review the naming" or "is this abstraction necessary?".

### `review-telemetry`

Treats telemetry as a consumer contract, since dashboards and alerts break
silently. Checks OpenTelemetry semantic conventions and namespace reuse, guards
stable dimension sets from silent renames, rejects unbounded attribute values
such as request IDs and raw URIs, flags per-emission allocation, and removes
instrumentation duplicating what a middleware crate already emits.

Trigger it with "review the metrics" or "check this telemetry".

### `review-delivery`

The shared posting layer. It owns the `[AI AGENT]: ` attribution rule, the
concern/verification/fix comment shape, precise anchoring, severity labels
(`nit:`, `non-blocking:`, design notes), the verdict, the anti-low-signal list,
and the mechanics that actually bite: generating the review JSON from a script on
disk rather than inline, never using `-f body=@file`, pinning `commit_id` to a
re-checked `headRefOid`, keeping anchors inside diff hunks, and the ADO
1-based-offset thread API.

Extracting it means every review skill posts identically instead of each carrying
its own drifting copy.

Trigger it with "post this review" — or let another `review-*` skill call it.

### `review-public-api`

Runs full and simplified `cargo public-api` listings, then inventories the
selected public items before reviewing common idiomatic Rust API practices,
module shape, naming, traits, signatures, construction, errors, and evolution
hazards. It layers the API-relevant Pragmatic Rust Guidelines on top and uses
exact output excerpts as evidence.

The main review deliberately does not inspect Rust source, manifests, docs,
tests, diffs, or rustdoc JSON, so it does not speculate about behavior,
correctness, safety, documentation quality, or performance. After drafting its
report, it delegates a matching cargo rustdoc JSON build to a separate agent,
which associates claims with API docs and removes or narrows statements those
docs refute. It runs `cargo public-api --all-features` by default so
feature-gated public APIs are included; explicit package, feature, target, and
semver-baseline scopes override that default. For a PR with only a few public
additions or changes, its findings are limited to those changed items and their
immediate API family rather than unrelated existing surface.

Trigger it with "review this crate's public API" or "audit the Rust API using
cargo public-api".

### `review-public-docs`

A retrieval primitive rather than a review skill. It resolves a change — PR
diff, branch, commit, working tree, or an explicit item list — to the public
items it touches, generates rustdoc JSON with `cargo +nightly rustdoc
--output-format json`, and returns a compact docs bundle: public path, item
kind, doc text, documented attributes, member docs, and resolved doc links.

Raw JSON never leaves the skill, so callers get authoritative documentation
without paying the context cost of parsing it. `review-public-api` uses it for
the post-processing pass that filters claims the docs refute, and any other
skill or agent needing real API docs can invoke it the same way. It reports
undocumented items as a fact, and deliberately makes no design or correctness
judgement.

Trigger it with "what do the changed public APIs document?" or "get the public
docs for these items".

### `review-resilience`

Uses the exact `recoverable` dependency's `_documentation::recipes` module to
review recovery classification and propagation through error conversions. It
also identifies hand-written resilience and recommends the matching
feature-gated `seatbelt` middleware, checking layer order, cancellation,
configuration, telemetry, and focused tests.

Trigger it with "review resilience", "check these errors implement Recovery",
or "find retries that should use seatbelt".

### `review-tests`

Compares tests and observable behavior against the baseline so an implementation
cannot silently delete coverage or rewrite expectations around a regression.
Intentional changes need an explicit trusted rationale and focused replacement
coverage. It also checks changed Rust tests for deterministic, behavior-focused
style and reuses existing `test-util`, `test-utils`, or repository-specific
test helpers instead of custom scaffolding.

Trigger it with "review the tests", "check for deleted tests", or "verify this
change preserves behavior".

## How the review skills compose

```
review-lens ......... orchestrator: scope, base, trust, CI baseline, routing
  ├── review-api-design ...... public contract from the diff and source
  ├── review-correctness ..... defects + the shared verification discipline
  ├── review-tests ........... deleted/weakened tests, unjustified behavior change
  ├── review-resilience ...... Recovery classification, seatbelt adoption
  ├── review-perf ............ allocations, hot path, injectable clocks
  ├── review-naming .......... sibling conventions, unnecessary abstraction
  ├── review-telemetry ....... OTel conventions, cardinality, duplication
  └── review-delivery ........ posts the single AI-attributed review

review-public-api ... standalone: existing surface from cargo public-api output
  └── review-public-docs ..... rustdoc JSON retrieval primitive
```

Every skill also runs standalone — ask for "review the naming" and only that
skill loads. `review-lens` exists so a single "review this PR" still gets the
full treatment without any one skill carrying all of it.

## Layout

```
.claude-plugin/marketplace.json
                                      Claude plugin marketplace entry
.github/plugin/marketplace.json      Copilot plugin marketplace entry
plugins/copilot-skills/
  .claude-plugin/plugin.json          Shared Copilot and Claude plugin manifest
  agency.json                         Agency engine, author, category, and platform metadata
  README.md                           Plugin documentation
  skills/<name>/SKILL.md              one directory per skill
  skills/<name>/*.md                  supporting procedures referenced by a skill
```

## Adding a skill

Create `plugins/copilot-skills/skills/<name>/SKILL.md` with YAML frontmatter
containing `name` and a `description` precise enough that the agent knows when
to fire it — and when not to. Bump `version` in
`plugins/copilot-skills/.claude-plugin/plugin.json` and both marketplace
entries, then push.
