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
| [`review-lens`](skills/review-lens/SKILL.md) | Reviews a Rust PR, branch, commit or working-tree diff with a library-maintainer lens: public API surface first, then unnecessary abstraction, correctness, dependency and semver exposure, hot-path complexity, naming, tests and docs. |

### `review-lens`

Derived empirically from a year of real PR review comments, so the priority
order reflects what actually gets raised in review rather than generic Rust
advice. Its `Repository adaptation` section carries workspace-specific
instincts for `microsoft/oxidizer` and generalises everywhere else.

Trigger it with "review this PR", "review my changes", or "review like me".

## Layout

```
.github/plugin/plugin.json    Copilot plugin manifest
.claude-plugin/plugin.json    Claude Code plugin manifest (same contents)
agency.json                   engine + category metadata
skills/<name>/SKILL.md        one directory per skill
```

## Adding a skill

Create `skills/<name>/SKILL.md` with YAML frontmatter containing `name` and a
`description` precise enough that the agent knows when to fire it — and when not
to. Bump `version` in both manifests, then push.
