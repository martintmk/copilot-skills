# copilot-skills

Personal [GitHub Copilot CLI](https://docs.github.com/copilot/how-tos/use-copilot-agents/use-copilot-cli)
skills, packaged as a cross-engine Agency-style plugin so they stay in sync
across machines.

## Install

With Agency:

```
agency plugin install github:martintmk/copilot-skills:.
```

For a single Agency session without installing:

```
agency copilot --plugin github:martintmk/copilot-skills:.
```

With GitHub Copilot CLI directly:

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
| [`rust-public-docs`](skills/rust-public-docs/SKILL.md) | Retrieves a crate's public API documentation from cargo rustdoc JSON, scoped to a change (PR diff, branch, commit, working tree) or an explicit item list. Returns a compact docs bundle instead of raw JSON, so it can be reused as a retrieval primitive by other skills and agents. |
| [`github-pr-info-fetcher`](skills/github-pr-info-fetcher/SKILL.md) | Maintains a folder of `<pr-number>.json` files for a repository's open, ready-for-review PRs through the GitHub MCP server, then evaluates queued files one by one with the current AI model to produce a brief and interest decision for the caller's query. |

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

### `rust-public-docs`

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

### `github-pr-info-fetcher`

Maintains one JSON file per open, ready-for-review PR in a caller-provided output
folder, without AI, then queues the files that need evaluation and runs one
detached AI pass per PR. The maintenance phase only refreshes deterministic
fields such as title, author, status, review count, and `source_updated_at`, and
drops files for PRs that are merged, closed, or converted to draft. The
evaluation phase fetches the changed files and diff context needed to judge
whether the actual change matches the caller's interest query.

Each file stores a brief, a boolean interest flag, and a short reason, and the
skill re-evaluates a file when the upstream PR changes or the cached AI result
is older than seven days. Downstream tools read `<output_folder>/*.json`
directly; no database is involved.

Trigger it with "cache the interesting open PRs in owner/repo into this folder"
or "refresh PR information for this repository using this interest query".

## Layout

```
.claude-plugin/plugin.json    Shared Copilot and Claude plugin manifest
.claude-plugin/marketplace.json
                              Claude plugin marketplace entry
.github/plugin/marketplace.json
                              Copilot plugin marketplace entry
agency.json                   Agency engine, author, category, and platform metadata
skills/<name>/SKILL.md        one directory per skill
skills/<name>/*.md            supporting procedures referenced by a skill
```

## Adding a skill

Create `skills/<name>/SKILL.md` with YAML frontmatter containing `name` and a
`description` precise enough that the agent knows when to fire it — and when not
to. Bump `version` in `.claude-plugin/plugin.json` and both marketplace entries,
then push.
