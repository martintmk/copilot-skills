# copilot-skills

Personal [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/use-copilot-agents/use-copilot-cli)
skills, packaged as a plugin so they stay in sync across machines.

## Install

```
/plugins install martintmk/copilot-skills
```

Update later with:

```
/plugins update copilot-skills
```

## Skills

| Skill | What it does |
| ----- | ------------ |
| [`review-lens`](skills/review-lens/SKILL.md) | Reviews a Rust PR, branch, commit or working-tree diff as an autonomous AI reviewing agent. Public API is the dominant lens: it inventories and evaluates the complete changed contract before performing exhaustive correctness and risk-based dependency, performance, naming, telemetry, test and documentation passes. Focused failing tests are included when they are useful permanent regression coverage, and every posted comment starts with `[AI AGENT]: `. |
| [`review-public-api`](skills/review-public-api/SKILL.md) | Reviews a Rust library's exported contract from `cargo public-api` output, then uses an isolated rustdoc JSON pass to filter claims refuted by the API docs. It applies common idiomatic Rust API practices first, then API-visible Pragmatic Rust Guidelines; small PRs are scoped to changed public items. |

### `review-lens`

Derived empirically from a year of real PR review comments, so public API design
is the dominant concern rather than one checklist item among many. It evaluates
the complete changed contract—exports, re-exports, implementability, representation,
features and semver commitments—without neglecting exhaustive correctness and
risk-based dependency, performance, naming, telemetry, test and documentation
review. It checks out the PR head and proves correctness or behavioral findings
with targeted tests, Miri, adversarial inputs and benchmarks. Inline comments
lead with consumer impact and cite the decisive result; when a focused failing
test demonstrates the defect and belongs in the permanent suite, the complete
test and recommendation are included after the finding. Its
`Repository adaptation` section carries
workspace-specific instincts for `microsoft/oxidizer` and `ox-sdk`, and
generalises everywhere else.

Findings are posted as a single structured review, explicitly attributed to an AI
agent so none read as if a human maintainer wrote them, with each one anchored to
the code it is about — as a GitHub review or as Azure DevOps threads.

Trigger it with "review this PR", "review my changes", or "review like me".

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

## Layout

```
.github/plugin/plugin.json    Copilot plugin manifest
.claude-plugin/plugin.json    Claude Code plugin manifest (same contents)
.github/plugin/marketplace.json
                              Copilot plugin marketplace entry
agency.json                   engine + category metadata
skills/<name>/SKILL.md        one directory per skill
skills/<name>/*.md            supporting procedures referenced by a skill
```

## Adding a skill

Create `skills/<name>/SKILL.md` with YAML frontmatter containing `name` and a
`description` precise enough that the agent knows when to fire it — and when not
to. Bump `version` in both plugin manifests and the marketplace entry, then
push.
