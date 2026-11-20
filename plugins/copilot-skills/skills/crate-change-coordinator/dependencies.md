# Dependencies and standalone packaging

This skill is a Markdown procedure, not an executable service.
Moving its files does not move the tools, accounts or running coordinator sessions.
The autonomous procedure requires the GitHub Copilot app's session and Agent merge capabilities.
Installing it in a plain CLI without those capabilities is not sufficient.

## Skill dependencies

| Dependency | Used by | Source and move requirement |
| --- | --- | --- |
| [feedback-autonomy](../feedback-autonomy/SKILL.md) | Child repairs and coordinator learning | Bundled sibling skill. Preserve the sibling path or update links and skill resolution. It has no further skill or reference-file dependencies. |
| [minimal-local-validation](../minimal-local-validation/SKILL.md) | Child implementation and CI repairs | Bundled sibling skill. Preserve its name or update child prompts. It has no further skill or reference-file dependencies. |
| `simple-english` | PR descriptions and Teams announcements | External skill, not included in this repository. Install it separately, or package it with permission and its reference files. |
| `agent-merge` | Native maintenance ticks in the child | App-provided skill. Keep it supplied by the app so instructions match the runtime. Do not vendor a stale copy. |

The coordinator's local files are `SKILL.md`, `child-session.md`,
`github-maintenance.md` and this file.
Move the whole directory so these relative links remain valid.
The bundled policy skills have no executable scripts or package dependencies.

`simple-english` also uses `references/use-cases.md` for PR prose and agent instructions.
Preserve its other referenced files if packaging that external skill.
Do not assume that installing this plugin also installs external skills.
Resolve installed names from the live skill list because plugin prefixes can change.

## Runtime and service dependencies

| Capability | Tools or access | Requirement |
| --- | --- | --- |
| Child lifecycle | `create_session`, `get_session`, `send_session_message`, `archive_session`, child notifications | Coordinated worktree children, inherited model configuration, autopilot kickoff and durable session IDs. |
| Human decisions | `ask_user`, `relay_session_input` | Relay pending child questions without answering them or bypassing approval. |
| Coordinator schedule | `get_session_automation`, `save_session_automation` | A 15-minute automation that wakes the same session, detects human resolution during waits, and supports read-back and cleanup. |
| PR maintenance | `manage_agent_merge` and app-native ticks | Enable CI repairs, review handling and conflict resolution on the linked child PR. Keep `merge_pr: false`. |
| GitHub delivery | `create_pull_request`, `update_pull_request` | Link the child PR and maintain its description. Agent merge owns the tools for authorized feedback replies and thread resolution. |
| GitHub reads and Git writes | Authenticated `gh` and `git`, or available provider tools | Read lifecycle, head, complete checks, statuses and feedback provenance. Commit and push only to the batch branch. |
| CI evidence | Access to each check's actual provider and logs | GitHub Actions uses `gh`. External CI requires its configured integration. Unreadable failures cannot count as passing. |
| Teams delivery | Deferred WorkIQ tools and their discovered schemas | Resolve the exact channel link, read recent messages, create one message, read it back and update it. |
| Rust workspace | Cargo and the repository's toolchain | Resolve packages with `cargo metadata` or manifests, then run narrow package checks and targeted reproductions. |
| Durable state | Session storage and deferred-tool discovery | Save the queue, setup result, human blockers, announcements and learning outside the target repository. Native maintenance owns repair history. |

GitHub access must permit PR reads, check and log reads, branch pushes and authorized PR writes.
WorkIQ access must permit reads and message writes in the supplied Teams channel.
Discover capabilities and use the actual schemas rather than assuming a fixed WorkIQ API.
Report authentication or permission failures without retrying uncertain writes.

`teams-self-message`, `pr-review-teams-channel`, `pr-auto-approve`, `review-lens`
and `track-pr` are not dependencies.
Another reviewer or process can use the Teams announcement, but this skill does not require their implementation.
ADO PRs are unsupported. An external CI provider for a GitHub PR is a separate dependency.

## Move to a standalone plugin

1. Copy the complete `crate-change-coordinator` directory into the new plugin's `skills` directory.
2. Bundle `feedback-autonomy` and `minimal-local-validation` as sibling skill directories, or arrange their separate installation.
3. If policy skills are separate, replace sibling file links with working references and make sure that invocation resolves.
4. Document the required external `simple-english` skill and app-provided `agent-merge` skill.
5. Add a plugin manifest and only the marketplace entries needed by the chosen installation method.
6. If Agency remains supported, include the relevant engine and platform metadata from `agency.json`.
7. Update plugin names, source paths, descriptions, install examples and matching plugin versions.
8. Make sure that all local links and installed skill names resolve before running a batch.

The current packaging uses `plugins/copilot-skills/.claude-plugin/plugin.json`
and `plugins/copilot-skills/agency.json`.
The repository root contains `.claude-plugin/marketplace.json` and
`.github/plugin/marketplace.json`.
These files distribute the skills. They do not provide runtime tools or credentials.

For the smallest self-contained bundle of repository-owned skills, include three directories:
`crate-change-coordinator`, `feedback-autonomy` and `minimal-local-validation`.
Keep external skills and service access explicit in the new README.
Moving files alone does not migrate running sessions, saved child IDs or scheduled automations.
