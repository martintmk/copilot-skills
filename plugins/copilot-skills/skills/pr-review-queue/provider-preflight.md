# Provider Prerequisites and Operation Bindings

Before scheduling, reviewing or recovery writes, validate a complete operation
plan for every active repository using **MCP and official provider CLIs together**.
Neither interface must cover everything. Missing combined capability blocks;
tool presence alone is not proof of authentication, permissions or semantics.

## Bounded discovery

Use **two minutes per active provider**, or the user's explicit budget, excluding
input waits and approved installation. Share the deadline/gaps with delegated
probes; never restart the clock in a worker.

- Reuse loaded schemas/help and matching facts; refresh live identity/access
  and required reads. Load [CLI recipes](provider-cli-recipes.md) only for the
  applicable CLI route instead of rediscovering known operations.
- Prefer synchronous probes. Delegate only independent, bounded work while
  making parallel progress. After failure, try at most one evidence-backed,
  permitted alternative within the remaining budget.
- Stop early on a proven required gap. At deadline, stop probing, safely stop
  or await owned commands/workers, retain their evidence and report the exact
  gap and any still-running work. Hold the lock until workers stop.
- Never guess endpoints repeatedly, enumerate unrelated organization services,
  bypass a denial or call incomplete history absent. Missing clients/auth and
  unproven history semantics are different blockers.

Exhaustion is `blocked`, not partial success. A later explicit attempt targets
saved gaps rather than repeating the whole investigation.

## Bind operations

1. Discover actual MCP operations/schemas, not presumed tool names. Check
   installed CLI versions, required extensions and operation help. Disable
   process-local Azure dynamic extension installation before even `--help`.
   Ask once in interactive setup before needed installation; unattended missing
   prerequisites block. Never silently install or change credentials/defaults.
2. Prefer GitHub CLI and configured ADO MCP where capable, filling gaps with the
   other interface. Record the actual tool schema or validated command/arguments,
   executable/version, host and repository scope. An entirely CLI-backed plan
   is valid; an unused missing interface is not a blocker.
3. Independently resolve stable target and posting actor IDs. Default GitHub
   target to `martintmk`, never infer ADO identity from it. Every chosen write
   route must use the same approved posting actor; MCP, CLI, Git and local
   names need not identify the same person.
4. Prove live authentication/access and available permissions with non-mutating
   identity/repository reads. Setup ambiguity asks; unattended ambiguity blocks.
   Never test permissions by posting, voting or removing reviewers. Actual
   write denials still stop work; policy/access denials cannot be bypassed
   through another interface.
5. Return run-local `operation_bindings` and `direct_http: false` through the
   [runner](review-runner.md), revalidating tool/credential/scope changes.
   Saved shell commands are not permanent authority.

`gh api`/GraphQL and `az devops invoke` are official CLI routes: verify installed
syntax/resource, preserve pagination and serialize payloads instead of splicing
PR text into commands. `curl`, `Invoke-RestMethod`, SDK/raw HTTP calls are not
fallbacks. Local Git/source inspection and permitted builds remain available;
radar permissions/state do not apply to this queue.

## Required capabilities

| Capability | Required meaning |
| --- | --- |
| Identity/access | Target, creator and posting actor IDs/classification; every active repository. |
| Collection | Fully paginated open PRs; stable repo/PR IDs, author, lifecycle/draft, creation time, target/base/head. |
| Requests | Current individual target requests plus authoritative generation/time/reset history, including same-head re-requests. |
| Prior reviews | Submitted review/vote and code-review-thread history distinguishing `present`, `absent` and `unknown`. |
| Review evidence | Pinned changes/content, commit/iteration, discussion and CI/check facts needed by Review Lens. |
| GitHub delivery | Submitted review, intended inline findings/summary and read-back by actor/head, not drafts or issue comments. |
| GitHub acknowledgment | Target-only request removal and current requests/events sufficient to detect generation races. |
| ADO delivery | Create/read iteration-bound findings/summary and required verdict votes; no vote on target/poster-owned PRs. |
| ADO acknowledgment | Authoritative request/reset/iteration evidence for local completion, without deleting/resetting assignments or votes. |

Generic API access cannot create unavailable history or conditional-write semantics.
A summary thread cannot substitute for a required vote.

## Normalize facts, retain evidence

Map actual response fields and preserve provenance/server timestamps for identities,
lifecycle, exact target/base/head/iteration, request generation/time and reviews.
Normalize REST/GraphQL IDs or ADO descriptors via provider metadata, never names,
so switching interfaces cannot create a new queue key.

- GitHub submitted reviews count regardless of author, outcome or dismissal;
  pending drafts and ordinary issue comments do not.
- ADO recorded votes/review events and code-review threads count. General
  discussion, author updates and system messages alone do not prove review.
  Zero/reset votes or unavailable history cannot prove absence.
- Requests must be individual. Membership/current votes are not generation
  history, and PR creation is not request time. Preserve authoritative
  assignment/re-request/reset/iteration evidence.

Use [cache/recovery rules](state-machine.md) for incomplete facts and
generation races. Read/write/read-back is not atomic removal. Only the runner's
exact verified whole receipt permits acknowledgment.

A gap in any active provider blocks the monitor. Explicit interactive scope
narrowing follows storage rules: retain inactive configuration/history, refresh
only the newly active set and reuse confirmed cadence. Unattended ticks never
drop a provider to bypass its blocker.
