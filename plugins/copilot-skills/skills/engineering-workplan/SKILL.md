---
name: engineering-workplan
description: >
  Plan or steer large engineering changes, rewrites and migrations. Define a
  measurable destination, compatibility contract, ordered slices, standalone
  agent prompts and evidence gates. Not for small checklists, code review,
  implementation, launching agents, scheduling, provider writes or merging.
---

# Engineering Workplan

Turn an objective into a verifiable destination and bounded assignments.
Prepare a new instruction pack, or steer an existing one from current evidence.
Reuse the current pack rather than creating a competing plan.

This skill permits read-only investigation, not execution of project code,
implementation or provider writes. Separate proposed actions from authorized
ones. A later execution request needs its own authority and workflow.
Return the pack in chat unless the caller requests a file or output location.

## 1. Establish the baseline

1. Read the requested outcome, existing decisions and relevant repository rules.
   Inspect architecture, production entry points, manifests, tests and CI only
   far enough to ground the plan.
2. Pin repository, revision, configuration and local changes.
   Separate observed facts with references from proposals and assumptions.
3. Identify consequential gaps: scope, compatibility, success conditions,
   release constraints and decision authority. Ask one focused question at a time.
   Do not ask for discoverable facts. If answers are unavailable, mark the pack
   as a draft with blocked decisions.
4. Map consumers, entry points, callbacks, I/O, state, persistence, packaging
   and deployment. Find existing helpers before proposing replacements.
5. For rewrites or dependency replacements, read the
   [risk guide](rewrite-risk-guide.md). Select relevant risks, not language-specific
   checks or targets copied from the case study.

## 2. Define the destination and contract

State what must be true, unchanged, deliberately different and absent at completion.
Name the exact boundary. Removing a production runtime does not necessarily
remove its UI or supported protocol.

Give each requirement an ID, current evidence, affected consumers, observable
pass/fail condition, proof method and acceptance owner.
Metrics need an actual baseline, workload, environment and agreed target.
Do not invent budgets or acceptance thresholds.

Separate required architecture changes from optional redesign.
For behavior-preserving work, keep behavior and algorithms first.
For approved redesign, record each intentional delta and its tests.
Unclear old behavior needs characterization or a decision.

Identify independent correctness evidence: existing consumer/E2E tests, schemas,
fixtures, contracts and operational expectations.
Plan characterization before replacement where that evidence is weak.
New tests copied from new implementation assumptions are not independent proof.

## 3. Order delivery slices

Recommend incremental replacement, dual-running or one cutover from actual
coupling and deployment constraints. Explain tradeoffs and obtain decisions for
consequential choices. Stateful dual-running can duplicate effects.
Distinguish code rollback from recovery of changed persistent data.

Plan build integration, packaging and contract coverage before a small shipping
pilot. The pilot must prove the complete production path.
Then order isolated components before shared state and central orchestration.
Split large components when one assignment cannot own their full change safely.

Each slice needs an owner, dependencies, bounded surface, integration owner,
observable result and link to the destination.
Each temporary adapter, callback bridge, flag or fallback needs a removal
condition and dependent cleanup slice. Permanent compatibility surfaces are separate.
Track incoming feature changes too. Fewer lines are not proof of completion.

## 4. Generate the instruction pack

Use the [templates](instruction-templates.md) for the workplan, coordinator kickoff,
ready-slice prompts and evidence record. Keep later slices brief until prerequisites
make their assignments concrete.

Each ready prompt must stand alone. Include the destination, exact source,
ownership/no-touch boundaries, invariants, granted actions, outputs, proof,
escalation conditions and return format.
Quote the user's assigned instruction separately from coordinator guidance.
Copy applicable binding rules, not the entire plan, transcript or future queue.
Use stable artifact references and small updates to a versioned shared contract.
Never include credentials or raw sensitive logs.

### Binding rules

| Rule | Required instruction |
| --- | --- |
| Preserve independent tests | Existing expectations, E2E tests, snapshots and required gates cannot be weakened without the designated owner's explicit approval. Separate harness changes from implementation. |
| Bound authority | Name allowed edits and operations. Commit, push, integration, publishing, merge and deployment are separate grants. Unknown authority is not permission. |
| Respect ownership | Give shared files one owner. Integrate only handed-off revisions through the designated integrator. Never take a peer's unfinished edits or treat silence as consent. |
| Limit parallel work | Keep small tasks and continuous traces with one agent. Delegate only substantial independent work. Confirm isolation before concurrent writes. Do not prescribe an arbitrary fleet or model. |
| Budget expensive checks | Assign one owner or a capacity/lease policy. Record owner, operation and release condition. Release only after work finishes, not after a timer. Never stop unrelated processes. |
| Keep proof current | Pin evidence to revisions and configuration. Separate local checks from CI/consumer gates. Pending, stale or unavailable evidence cannot close a gate. |
| Fix causes within authority | Compare failures with the contract. Escalate ambiguity, repeated failure or scope changes with diagnostics. Do not weaken the success condition. |

For Rust implementation with comprehensive CI, use
[minimal-local-validation](../minimal-local-validation/SKILL.md).
Otherwise select the smallest relevant repository checks.
For later PR feedback, use [feedback-autonomy](../feedback-autonomy/SKILL.md),
not permission to obey every comment.

## 5. Steer from evidence

Read the current contract, evidence record and affected results.
Compare delivered behavior with requirement IDs.
Distinguish written, integrated, verified and accepted work.
An agent's completion claim is not acceptance evidence.

Issue the smallest corrective prompt: observed mismatch, unchanged goal, owned
scope, required correction, protected constraints, decisive proof and stop condition.
Continue independent planning when one slice blocks.
Return contract, scope and architecture decisions to the decision owner.

After integration or upstream movement, account for added and removed behavior.
Conflict-free integration does not prove compatibility.
Recheck affected contracts and justify reuse of old evidence.

Update the handoff with version, revision, owners, decisions, open requirements,
temporary remnants, evidence links and next action.
Keep enough state to resume without replaying the investigation.
Recurring failures need a focused guard or reusable rule, not a growing transcript.

## 6. Gate readiness and completion

A ready slice has resolved dependencies, concrete authority and an executable
check or named human decision for each acceptance condition.
Assign unknowns to research or characterization, not ready implementation.

Accept a slice only with production wiring, relevant consumer coverage, current
required checks/review and owned cleanup obligations.
Accept the initiative only across all in-scope surfaces, with temporary work
removed, intentional deltas approved and rollout evidence complete.
Prove forbidden remnants absent from artifacts and execution paths.
A compiler pass, merge or agent consensus is not a substitute.

End Prepare with readiness and unresolved decisions.
End Steer with the actual gap, changed instructions and next gate.
The instruction pack's completion does not mean the engineering work is complete.
