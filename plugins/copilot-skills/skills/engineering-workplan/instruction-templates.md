# Instruction templates

Use these output shapes with the procedure and binding rules in
[Engineering Workplan](SKILL.md). Replace angle-bracket fields with observed facts
or explicit pending decisions. A ready prompt cannot retain unresolved fields
that determine scope, permissions, behavior or acceptance. Omit inapplicable
sections with a short reason; do not manufacture busywork to fill a template.

## Workplan

```markdown
# <initiative>

Instruction version: <version>
Readiness: <draft with blocked decisions | ready for the named first slices>
Repository / baseline / configuration: <exact facts, including dirty state>
Decision owner: <human or caller-designated authority>
Coordinator / integration owner: <assigned roles or identities>

## Destination

Why this work matters: <consumer or operational problem>
At completion: <observable delivered state, including the entire target boundary>
In scope: <production surfaces and required supporting work>
Out of scope: <adjacent systems, optional redesign and unrelated fixes>
Must preserve: <external behavior and other invariants>
Approved intentional changes: <decision references, or none>
Must be gone: <legacy paths, temporary bridges, dependencies or other remnants>

## Requirements and baseline

| ID | Current behavior / source | Required outcome / affected consumers | Proof and pass condition | Acceptance owner |
| --- | --- | --- | --- | --- |
| R1 | <revision:path or artifact> | <concrete result> | <command/check/scenario and expected result> | <owner> |

Inventory: <entry points, callbacks, I/O, state, persistence, config, packaging,
platforms and deployment paths; link each to requirements>
Protected oracle: <existing E2E/consumer tests, fixtures, schemas and baselines>
Coverage gaps: <characterization tasks that precede replacement>
Performance/resource targets: <baseline, workload, environment, target, proof;
mark unagreed targets as pending, or explain why no new target is needed>

## Strategy and delivery slices

Chosen/proposed strategy: <atomic incremental replacement, dual-running,
single cutover or another justified approach; include approval state>
Tradeoffs and rejected alternatives: <coupling, side effects, drift and recovery>
Enabling work: <build/CI, packaging, contracts and feedback-loop readiness>
Shipping pilot: <small representative path, what it proves, exit gate>

| Slice | Observable result / requirement IDs | Depends on | Owner / integrator | Exit evidence | State |
| --- | --- | --- | --- | --- | --- |
| S1 | <bounded result> | <none or IDs> | <owners> | <proof> | <planned> |

Dependency order: <acyclic waves; shared-state/orchestration sequencing>
Temporary-work register: <adapter/flag/fallback -> owner -> dependent cleanup
slice -> removal condition and absence proof>
Concurrent product changes: <who reconciles incoming behavior, at what checkpoints>

## Validation, acceptance and rollout

Fast local feedback: <repository commands, scope, configuration and owner>
Required CI/consumer gates: <check names, coverage and result references>
Review: <contract/parity checks, architecture decisions and high-risk owner>
Forbidden shortcuts: <concrete protected tests/gates and applicable binding rules>
Slice acceptance: <integrated revision, wiring, evidence and outstanding obligations>
Initiative acceptance: <all requirements and cleanup closed; authorized acceptance>
Rollout: <small release/exposure units, observation signals/window and owner>
Stop/recovery: <triggers, authority, code rollback and persisted-data constraints>
Not yet verifiable: <checks that need a future environment, release or decision>

## Coordination and authority

Ownership / no-touch map: <slices, shared hubs and external/peer boundaries>
Permitted operations: <edits, workers, commit, push, integrate, publish, deploy;
state granted scope individually and mark the rest not authorized>
Escalation: <overlap, ambiguous semantics, scope/contract change, repeated failure>
Resource budget: <edit concurrency, heavy-check owner/capacity, lease/release policy>
Handoff location and cadence: <where evidence lives, integration/CI/decision checkpoints>

## Decisions and risks

| ID | Unknown or risk | Evidence / options | Owner | Blocks | Resolution |
| --- | --- | --- | --- | --- | --- |
| D1 | <question, not an invented assumption> | <facts and proposed next step> | <owner> | <slice/gate> | <pending> |
```

## Coordinator kickoff

Generate this alongside the workplan. It is an instruction to use **if execution
is authorized**, not a request to start a fleet while preparing the pack.

```text
Coordinate <initiative> against instruction version <version> at <plan location>.
Destination: <standalone finish line and forbidden remnants>.
Contract: <preserved behavior, approved deltas and protected oracle>.
Baseline and evidence: <repository/ref/configuration and compact source map>.

Your authority is <explicit operations and boundaries>; everything else is
not authorized. The decision owner is <owner>. You own <integration surface>.
Shared/peer boundaries and applicable binding rules: <spell them out>.

Start with <dependency-ready slices and pilot/enabling gates>. Dispatch only
substantial independent work whose tools, permissions and isolation are confirmed.
Use <single owner or explicit capacity/lease policy> for expensive local checks;
require <fast local feedback> and rely on <named CI gates> for broader coverage.

At each <integration, upstream change, CI result or decision checkpoint>, compare
evidence with requirement IDs. Produce a bounded corrective instruction for a
verified gap; escalate <named decisions> rather than changing the goal or oracle.
Integrate only <handoff protocol and authorized revisions>. Require <rechecks>.
Do not equate code generation, a merge or green compilation with acceptance.

Keep <ledger location> current with snapshots, owners, evidence, decisions,
remaining transitional work and next actions. Apply <review/feedback policy>.
Stop at <authorized completion boundary>; acceptance requires <gates and owner>.
Return <completed requirements, unclosed gates, exact artifacts and next decision>.
```

## Slice kickoff

Produce a complete packet for each ready slice. A research or characterization
task can use this shape too, with read-only authority and an evidence deliverable.

```text
Task: <slice ID and instruction version>
Owner / integrator: <identities or assigned roles>
Objective: <observable slice result and contribution to the overall destination>
Requirements: <IDs plus their actual pass/fail conditions>

Context: <observed behavior, source locations, relevant callers and conventions>
Baseline: <repository/ref/configuration; branch/worktree if assigned>
Prerequisites: <dependency IDs, approved handoff revisions and decisions>
Own: <allowed files/components and exact boundary>
Do not touch: <shared hubs, neighboring slices, peers and protected artifacts>
Inputs/outputs at the boundary: <contracts, callbacks, state and errors>

Preserve: <behavior, algorithms for parity work, and relevant operational limits>
Change deliberately: <only approved deltas, or none>
Applicable binding rules: <standalone oracle, autonomy, isolation, resource,
evidence and feedback constraints selected from SKILL.md>
Permitted operations: <specific grants; distinguish edits from publishing,
integration, history changes and deployment>

Deliver: <implementation or research result, production wiring, applicable tests,
docs/config/packaging, and cleanup required by this slice>
Temporary obligations: <only planned bridges, owners and removal dependencies>
Proof: <requirement -> exact check/scenario -> expected observation; name local,
CI and human ownership separately>
Resource policy: <allowed local checks and heavy-work owner/lease>

Escalate instead of guessing: <missing prerequisite, ambiguous behavior,
ownership collision, scope change, failed oracle or repeated failure trigger>
While blocked: <independent authorized work, or return diagnostics and stop>

Return: <snapshot; changed/handed-off artifacts; observed results by requirement;
pending/stale/failed checks; residual obligations; next gate>
Completion boundary: <this slice's acceptance, not "most logic ported";
list broader initiative work explicitly outside this slice>
```

## Evidence ledger and resumable handoff

Keep the record small and link artifacts instead of pasting logs. A result is
`verified` only when its stated proof passes for the relevant snapshot and
configuration; `accepted` additionally requires the designated acceptance gate.
A blocked item stays blocked even if other slices advance. Use these same state
meanings in the workplan; code-written/integrated milestones alone are not verified.

```markdown
Instruction version / checked snapshot: <facts>
Current destination and authority: <compact unchanged contract or approved delta>
Active owners and resource leases: <task -> owner; heavy operation -> owner/status>

| Slice | State: planned / active / blocked / verified / accepted | Snapshot | Evidence by requirement | Missing gate / next action |
| --- | --- | --- | --- | --- |
| <ID> | <state> | <revision/config> | <command/check, observed result, artifact> | <specific gap or none> |

Decisions: <approved deltas and unresolved questions, owner and affected IDs>
Residuals: <unported paths, bridges, fallbacks, cleanup and assigned slices>
Incoming changes: <baseline movement, affected requirements and reconciliation>
Evidence limits: <not run, pending, unavailable or stale; never label these passed>
Next bounded action: <owner, instruction and decisive next gate>
```

## Corrective instruction

Use a factual mismatch rather than "try harder", "finish everything" or a fresh
open-ended rewrite request. If the cause is unknown, the next bounded action
is investigation, not a speculative fix.

```text
Keep destination <goal> and contract <version/requirement IDs> unchanged.
At <snapshot/configuration>, <observed evidence> does not satisfy <pass condition>.
The gap is <specific missing behavior, wiring, evidence or decision>.

Within <owned boundary>, <investigate or correct this bounded gap>.
Preserve <oracle, neighboring work and other applicable binding rules>.
Do not <tempting shortcut or unauthorized scope expansion>.
Use <decisive check> to demonstrate <expected result>; obtain <CI/human gate>
where it cannot be established locally.

Return the evidence record and remaining obligations. If <ambiguity, missing
authority or repeated-failure condition>, stop this slice with <diagnostics and
decision owner>; continue only <independent authorized work>.
```
