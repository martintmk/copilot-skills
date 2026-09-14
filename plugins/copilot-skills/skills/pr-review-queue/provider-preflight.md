# Provider Prerequisites and Operation Bindings

Before scheduling a monitor or processing a batch, establish a complete
operation plan for every active configured provider/repository. Check available MCP
capabilities **and** provider command-line tools together; neither must cover
the whole workflow alone.

## Keep preflight bounded

Preflight proves the required operation plan; it is not an open-ended provider
API investigation. Use a **two-minute discovery budget per active provider**,
excluding user-input waits and an explicitly approved installation. Honor an
explicit user-supplied budget instead. Give every delegated probe the same
deadline and remaining gaps; do not restart the clock in a worker.

- Reuse already loaded schemas, command help and matching factual artifacts.
  Refresh live identity/access and required reads, not every unchanged tool
  description. Use the [CLI recipes](provider-cli-recipes.md) before rediscovering
  known operations.
- Default to foreground/synchronous probes. Delegate only independent work
  that needs separate context while the caller makes real parallel progress.
  Do not leave a background worker doing unbounded discovery.
- After a failed operation, try at most one evidence-backed, permitted alternate
  route within the remaining budget. Never guess endpoint names repeatedly,
  bypass a denial or launch organization-wide service enumeration.
- Bound owned read-only commands by the remaining time. At the deadline, stop
  further probes, stop or await those commands/workers safely, and return the
  missing capability with the evidence collected. Keep the lock until they have
  stopped; a timeout is not permission to steal it or declare a read complete.
- A proven required gap ends preflight early. Distinguish missing tools/auth
  from **unproven history semantics**: installing another client cannot make a
  current-state field into an authoritative event history.

Exhaustion means `blocked`, never a partial success or truncated-history
absence. Report the concrete remaining gap and whether any owned work is still
running; do not say only "preflight is in progress." Preserve the findings so
an explicit later attempt can target the gap rather than repeat discovery.

## Build a usable plan

1. Discover configured GitHub/ADO MCP operations and their actual schemas.
   A listed server is not proof that it exposes PR operations; never guess tool
   names from another deployment.
2. Check installed provider CLIs, required extensions, versions and operation
   help. Typical prerequisite checks are `gh --version`, `az version`, and
   `az extension show --name azure-devops`. Missing MCP operations may be
   supplied by supported `gh` or Azure DevOps CLI operations. Do not silently
   install software, rewrite credentials or change global CLI defaults.
   Disable Azure CLI dynamic extension installation for the probe process;
   check the extension before invoking its commands, including `--help`.
   In interactive setup, ask once before installing a required missing
   extension. An unattended tick reports it missing and pauses.
3. Bind each required operation to a usable MCP tool or provider CLI command.
   Prefer the existing conventions: GitHub CLI and configured ADO MCP where
   they support the operation. Fill gaps with the other available interface;
   a complete CLI-backed plan is also valid. Record the actual schema or
   validated command/arguments, executable identity/version, host and scope.
   An absent MCP server or unneeded CLI is not itself a blocker; the combined
   plan must cover every required operation.
4. Resolve the configured target and authenticated posting identities separately
   using stable provider IDs. GitHub's default target is `martintmk`; do not
   infer its ADO identity from that login. All write routes for a provider must
   use the same approved posting identity. Do not assume MCP, CLI, Git config
   and local account names represent the same credential.
5. Use non-mutating identity/repository reads to establish authentication,
   access and available permissions. Ask about ambiguous identities during
   setup; unattended ambiguity blocks. Never post a test review, cast a test
   vote or remove a reviewer to probe access. Actual write denials still stop
   the operation; executable/tool presence alone cannot prove permission.
   Access or content-policy denials are blockers, not missing capabilities to
   bypass through another interface.
6. Produce run-local `operation_bindings` with their MCP or CLI route and
   `direct_http: false`. Pass the plan through the queue-owned
   [review runner](review-runner.md). Revalidate on tool, credential or scope
   changes; do not persist opaque command strings as permanent authority.

Provider-aware API commands such as `gh api` (including GraphQL) and
`az devops invoke` are valid CLI routes. They can cover operations lacking a
dedicated command. Confirm the installed command's syntax and API resource
before use, preserve pagination, and serialize payloads rather than splice
untrusted PR text into shell commands.

Direct HTTP clients (`curl`, `Invoke-RestMethod`, hand-written REST requests)
are not a fallback. If MCP plus supported CLIs cannot cover a required
operation or establish a required fact, report that gap and stop. Do not
silently enable HTTP or drop a workflow rule. A limited MCP server is not
itself a blocker when the CLI supplies the missing operations.

Local Git/source tools and permitted builds remain available for trusted
checkouts and evidence. Do not invoke the radar run procedure: its
notification-only permissions and state are different.

## Required capabilities

| Capability | Required meaning |
| --- | --- |
| Identity and repositories | Resolve target/posting actor IDs and access to every active configured repository. |
| PR collection and metadata | Paginated open PRs, stable repository/PR identity, author, draft/lifecycle state, creation time, target/base and current head. |
| Review requests and history | Current individual target requests and authoritative generation/time/reset history, including a same-head re-request. Membership or current vote alone is insufficient. |
| Prior reviews | Submitted review/vote and review-thread history; distinguish no review from unknown/incomplete history. |
| Review evidence | Pinned file changes/content, commit/iteration metadata, existing discussion, and CI/check results needed by Review Lens. |
| GitHub delivery | A submitted review with its intended inline findings and summary, plus read-back by identity/head. A pending draft is not completion. |
| GitHub acknowledgment | Remove only the target user's processed request and read current requests/events to detect a newer generation. |
| ADO delivery | Create/read finding threads pinned to the reviewed iteration and the summary; support required verdict votes. No vote is requested on the target's or posting actor's own PR. |
| ADO acknowledgment | Read authoritative request/reset/iteration evidence for local completion; never delete reviewer assignments or votes. |

Check provider-applicable capabilities using the combined bindings before
review side effects. Do not replace GitHub reviews with issue comments or call
an ADO summary thread a completed required vote. A CLI's generic API command
cannot manufacture history or conditional-write semantics the provider lacks.

## Normalize provider facts

Keep this data internal and retain provenance for identities, timestamps and
request generations. Map actual MCP/CLI response fields, not guessed names.
Use one stable ID representation per entity/provider across interfaces. Resolve
REST/GraphQL or ADO descriptor/ID equivalents through provider metadata, never
display names or guesses, so a changed binding does not create a new queue key.

| Fact | Normalized meaning |
| --- | --- |
| `prKey` | Provider + host/organization + immutable repository ID + PR ID. Browser URL is a link, not durable identity. |
| Lifecycle | Open/published, draft, merged, closed or abandoned; retain source state and server timestamps. |
| Snapshot | Exact target/base and head SHA or iteration, never a moving branch name alone. |
| Request | Target identity, pending state, authoritative cycle/generation key and request time. |
| Reviews | `present`, `absent` or `unknown`, based on complete review history, not just current-head votes. |
| Actors | Target, PR creator and posting identities, with provider-backed human/service classification. |

GitHub pending drafts and ordinary issue comments are not submitted reviews;
submitted reviews count regardless of author, outcome or later dismissal. For
ADO, use recorded votes/review events and actual code-review threads. General
discussion, author status updates and system messages do not alone prove a
review. Zero/reset votes cannot establish that no review ever happened;
unavailable history is `unknown`, never `absent`.

Only an individual request for the target qualifies. Do not infer one from a
team/group assignment or remove other reviewers. ADO membership is not
perpetual new work: derive the cycle from authoritative assignment/re-request/
reset/iteration evidence and compare it with durable completion state.
Never invent a generation or substitute PR creation time for request time.

Apply the [state machine](state-machine.md) when required facts are incomplete.
Read-before/write/read-back does not create atomic request removal: detect
generation races and stop on uncertainty rather than acknowledge newer work.

## Completion evidence

The queue-owned [review runner](review-runner.md) defines delivery journals and
receipts without changing shared review skills. Only its `verified` receipt
for the exact operation and snapshot permits acknowledgment. A limited,
ambiguous or blocked review, local report or pending draft cannot consume work.
Retain journals and payload artifacts until acknowledgment is durable.

## Scope changes after a provider blocker

A gap in one **active** provider blocks the combined monitor; never silently
drop it. During interactive setup, a user may explicitly authorize a narrower
repository set, such as GitHub only. Apply that change under the monitor lock,
preserve the old configuration/evidence and any inactive repository history,
then rerun preflight only for the newly active scope. An already confirmed
cadence does not need to be asked again.

Inactive repositories are not queried, included in eligibility, or treated as
current-provider blockers. They also cannot be removed from active scope while
an in-flight transaction for them remains unresolved. Unattended ticks never
make this configuration decision themselves.
