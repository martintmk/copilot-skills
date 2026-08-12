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
| [`review-lens`](skills/review-lens/SKILL.md) | Reviews a Rust PR, branch, commit or working-tree diff as an autonomous AI reviewing agent covering public API surface, constructors and defaults, dependencies, consistency, forensic correctness, performance, naming, telemetry, tests and docs. Proves findings by building and running them, then posts a structured, AI-attributed review to the PR (GitHub review or Azure DevOps threads). |

### `review-lens`

Derived empirically from a year of real PR review comments, so the priority
order — public API surface, dependencies, consistency, forensic correctness —
reflects what actually gets raised in review rather than generic Rust advice. It
works as an autonomous AI reviewing agent: it checks out the PR head and proves
findings by building and running them (tests, Miri, adversarial inputs,
benchmarks), quoting the exact output, before asserting them. Its `Repository
adaptation` section carries workspace-specific instincts for `microsoft/oxidizer`
and `ox-sdk`, and generalises everywhere else.

Findings are posted as a single structured review, explicitly attributed to an AI
agent so none read as if a human maintainer wrote them, with each one anchored to
the code it is about — as a GitHub review or as Azure DevOps threads.

Trigger it with "review this PR", "review my changes", or "review like me".

## Layout

```
.github/plugin/plugin.json    Copilot plugin manifest
.claude-plugin/plugin.json    Claude Code plugin manifest (same contents)
.github/plugin/marketplace.json
                              Copilot plugin marketplace entry
agency.json                   engine + category metadata
skills/<name>/SKILL.md        one directory per skill
```

## Adding a skill

Create `skills/<name>/SKILL.md` with YAML frontmatter containing `name` and a
`description` precise enough that the agent knows when to fire it — and when not
to. Bump `version` in both plugin manifests and the marketplace entry, then
push.
