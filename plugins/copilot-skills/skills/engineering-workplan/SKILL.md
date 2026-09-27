---
name: engineering-workplan
description: >
  Prepare and maintain outcome-driven instructions for large engineering work,
  rewrites, migrations and multi-agent delivery. Produces a measurable destination,
  compatibility contract, dependency-ordered slices, copy-ready agent prompts,
  evidence gates and course corrections. Use when asked to plan a large change,
  write instructions for engineering agents, break down a rewrite, or steer an
  ongoing initiative toward its intended outcome. Not for small task checklists,
  code review, implementing the change, or automatically launching agents,
  scheduling work, posting to providers or merging pull requests.
---

# Engineering Workplan

Turn intent into **a verifiable destination, bounded work assignments, and a
human-directed feedback loop**. Based on the
[Copilot runtime rewrite case study](rewrite-risk-guide.md), not on an assumption
that every system should be rewritten or use Rust.

## Modes and authority

**Prepare** creates an instruction pack. **Steer** updates an existing pack from
current evidence and produces the next corrective instruction. Infer the mode
from the request; reuse an existing pack rather than creating a competing plan.

This skill prepares instructions, not the implementation. Inspect authorized
repository evidence read-only; do not run project code, change production files,
start workers, create schedules, or perform provider writes merely to prepare
a plan. An execution request needs separate, explicit authority and the applicable
execution workflow. Instructions must distinguish proposed actions from authorized
ones; access to a tool or another worktree is not permission to use it.

Return the pack in chat unless the caller asks for files. Use the requested output
location when supplied; do not scatter planning documents through a repository.
Record unavailable evidence as unknown, never as a successful check.

## 1. Establish intent and a factual baseline

Start with the requested outcome and existing decisions. Inspect repository
instructions, architecture/entry points, manifests, representative production
paths, tests and CI only as needed to ground the plan. Pin the repository,
revision and relevant configuration; describe uncommitted changes when applicable.
Separate observed facts with source references from proposals and assumptions.

Resolve consequential gaps in one focused question batch: why the work matters,
target and excluded surfaces, required compatibility or intentional changes,
measurable success, release constraints, and who can decide or authorize actions.
Use the available question tool. Do not ask for facts already supplied or readily
discoverable. If answers are unavailable, deliver a useful **draft** with blocked
decisions rather than inventing thresholds, commands, permissions or architecture.

Map behavior, not just files: consumers, public entry points, callbacks, I/O,
state ownership, configuration, persistence, packaging and deployment paths.
Find existing helpers and conventions before proposing replacements. For rewrites
or dependency replacements, load [the risk guide](rewrite-risk-guide.md) and
select the applicable risks; do not impose language-specific checks on other work.

## 2. Specify the destination and protect the contract

Define what will be true at completion, what must remain unchanged, what may
deliberately change, and what must no longer exist. Name the exact boundary:
for example, removing a legacy runtime from production execution does not
necessarily remove a UI, test harness or supported external protocol.

Give each requirement an ID and connect it to current evidence, affected
consumers, an observable pass/fail condition, a proof method and an acceptance
owner. Include negative conditions such as no legacy execution path, no temporary
fallback, or no blocking work on a host event loop when relevant. Metrics need a
baseline, workload, environment and agreed target; never borrow the article's
numbers or fabricate a budget.

Separate required architecture changes from incidental redesign. For a
behavior-preserving rewrite, retain behavior and algorithms first; schedule
optional optimizations separately. For a requested redesign or new capability,
name the approved behavior deltas and their tests rather than falsely promising
total equivalence. Ambiguous old behavior is a decision or characterization task.

Identify the independent correctness oracle: existing E2E/consumer tests,
recorded contracts, fixtures, schemas and operational expectations. Where coverage
is weak, plan characterization of the old system **before** replacement. A new
test copied from the new implementation's assumptions is not independent proof.

## 3. Choose a delivery strategy and dependency-ordered slices

Recommend a strategy with tradeoffs; do not silently decide a consequential
cutover choice. Consider incremental atomic replacement, temporary dual-running,
and a single cutover against the actual coupling, data and deployment constraints.
Prefer small, shippable replacements when feasible. Explain when stateful
dual-running would duplicate effects or require unsafe state synchronization.
If a single cutover is necessary, make reconciliation, rehearsal and recovery
explicit. Distinguish rollback of code from recovery of changed persistent data.

Plan the enabling work first: toolchain/build integration, packaging, contract
coverage and fast feedback. Then select a small, representative shipping pilot
that proves the complete path, not just isolated logic. Let the pilot establish
patterns before expanding.

Order remaining work by dependencies: isolated leaves before shared state and
central orchestration. A large component may need several waves through logic,
state ownership, callers, orchestration and cleanup. Each slice must have one
owner, explicit dependencies, a bounded change surface, an integration owner,
its own observable result, and a place in the final outcome.

Separate temporary adapters from permanent compatibility surfaces. Every
temporary shim, callback bridge, flag or fallback needs an owner, dependent
cleanup slice and removal condition. Do not move scaffolding to an unowned
"later" list. Track incoming features and changed requirements as well as
remaining work; shrinking line counts are not a completion measure.

## 4. Generate the instruction pack

Load [instruction templates](instruction-templates.md). Produce a workplan, a
coordinator kickoff, copy-ready prompts for dependency-ready slices,
and an initial evidence/handoff record. Later slices can remain brief until
their prerequisites establish enough facts to issue precise instructions.

Each ready prompt must stand alone: include the destination, its contribution,
source snapshot, concrete ownership/no-touch boundaries, relevant invariants,
permitted actions, deliverables, proof, escalation conditions and return format.
Restate applicable binding rules below inside the prompt; a link alone is
insufficient for a worker without this skill in context. Refer to stable artifact
paths rather than past conversation or "do the same as the other agent."

Specify outcomes and constraints, not a guessed line-by-line implementation.
Keep the shared contract stable and versioned; send small task-specific deltas.
Do not copy the whole repository, session transcript or every future task into
each worker prompt. Never include credentials or raw sensitive session logs.

### Binding rules for generated instructions

- **Preserve the oracle.** Existing E2E tests, expected results, snapshots,
  compatibility baselines and required gates are protected. Changing/removing
  them, skipping checks, adding suppressions or applying waiver labels needs
  explicit approval from the designated decision owner. Even necessary harness
  changes must be identified separately from implementation changes. Adding
  independent coverage is encouraged; redefining success to make CI green is not.
- **Bound autonomy.** Name permitted edits and operations, no-touch areas,
  shared-file owners and the coordinator who resolves overlap. Unknown authority
  means not authorized. No grabbing a peer's uncommitted work, crossing its
  ownership boundary, or treating its refusal as consent. Integrate only an
  explicitly handed-off revision through the designated integrator. Commit,
  push, history rewriting, publishing, merge and deployment authority are
  separate; a work assignment does not silently grant them.
- **Parallelize only independent work.** Keep small tasks and continuous traces
  with one agent. Use workers for substantial independent research or bounded
  implementation slices. Verify actual workspace isolation before concurrent
  mutation; do not assume a subagent has its own worktree. Serialize shared hubs
  or give them one owner. No arbitrary fleet size or model pinning.
- **Budget validation resources.** Separate edit concurrency from build
  concurrency. Give expensive local work one designated owner or an explicit
  capacity/lease policy; workers do not each launch full suites. A lease records
  owner, operation and release condition. Reassign only after the previous work
  is confirmed finished, not merely after a timer. Never stop unrelated processes.
- **Keep evidence independent and current.** Plan fast local checks and required
  CI/consumer checks separately. For changed Rust crates with comprehensive CI,
  use [minimal-local-validation](../minimal-local-validation/SKILL.md);
  otherwise choose the smallest applicable repository checks. Do not weaken
  acceptance to fit the local budget. Pin results to revision/configuration;
  pending, unavailable or stale evidence cannot close a gate.
- **Fix causes within authority.** Check suspected defects against the contract,
  then fix and recheck the affected paths. Use applicable review workflows, with
  human attention on architecture, contracts and high-risk decisions. If execution
  later includes PR feedback, follow
  [feedback-autonomy](../feedback-autonomy/SKILL.md), not a blanket instruction to
  obey every comment. Repeated failure without new evidence requires escalation
  with diagnostics, not endless retries or an escape hatch.

## 5. Steer from evidence, not progress claims

Read the existing contract, ledger and relevant current evidence. At each
checkpoint compare delivered behavior with requirement IDs. Distinguish code
written, integrated, verified and accepted; an agent's "done" is a claim to assess.

Find the smallest actionable gap and produce a corrective prompt using the
template: observed mismatch, unchanged goal, owned scope, required correction,
protected constraints, decisive proof and stop/escalation condition. Continue
planning independent slices when one slice blocks. Scope, contract or architecture
changes go back to the decision owner; they are not unilateral course corrections.

After upstream movement or integration, account for newly added/removed behavior
and translate incoming changes into the replacement where needed. Conflict-free
integration is not evidence of semantic preservation. Recheck affected contracts
on the integrated revision; reuse old evidence only with a stated relevance basis.

Update the handoff with instruction version, exact snapshot, owners, decisions,
closed and open requirements, temporary remnants, evidence references and the
next bounded action. Keep enough durable context to resume after compaction
without replaying the investigation. Use observed regressions to propose a
targeted test or guardrail; recurring failures warrant a reusable instruction or
evaluation, not an ever-growing dump of transcripts.

## 6. Gate readiness and completion

Before issuing a ready slice, check that its dependencies and required decisions
are resolved, its boundary/authority is concrete, and every acceptance condition
has an executable check or a named human decision. Unknowns can be assigned to
bounded research/pilot tasks, not hidden in supposedly ready implementation work.

Before accepting a slice, require production wiring, applicable consumer
coverage, current required checks/review, and assigned cleanup obligations.
Before accepting the initiative, require the destination across **all** in-scope
surfaces, closed temporary-work obligations, approved intentional deltas, and
the agreed rollout/operational evidence. Demonstrate forbidden remnants are
absent from relevant artifacts and execution paths, not merely absent from one
source search. A compiler pass, PR merge, line count or agent consensus is not
substitute evidence.

End Prepare with the pack's readiness and unresolved decisions. End Steer with
the actual gap, changed instructions and next gate. Never claim the engineering
work itself is complete because its instruction pack is ready.
