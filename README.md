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
| [`review-lens`](skills/review-lens/SKILL.md) | Reviews a Rust PR, branch, commit or working-tree diff with a library-maintainer lens covering internal correctness and control flow, public API and semver exposure, abstraction, tests, performance, dependencies, naming and docs. Posts the result to the PR as explicitly AI-attributed inline comments. |

### `review-lens`

Derived empirically from a year of real PR review comments, so the priority
order reflects what actually gets raised in review rather than generic Rust
advice. Its `Repository adaptation` section carries workspace-specific
instincts for `microsoft/oxidizer` and generalises everywhere else.

Findings are verified against a checked-out PR head before being asserted, then
posted as a single GitHub review with each finding anchored to the code it is
about. The review body and every inline comment are explicitly attributed to an
AI agent — none read as if a human maintainer wrote them.

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
