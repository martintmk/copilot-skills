# Review Lens coordinator reference

This reference gives strategies for multi-scope reviews.
It defines evidence outcomes, not one required reasoning process.
Use only the sections that help the current change.

## Evidence-led routing

Start with the complete changed-path inventory and affected contracts.
Consider every Review Lens area, then spend effort according to risk, uncertainty and user intent.
Route connected code, tests and documentation together when they need the same reasoning.
Do not invoke specialists only to fill an area ledger.

### Record coverage

For a small direct review, a short scope note is enough.
For delegated, blocked or multi-stage work, record:

| Field | Content |
| --- | --- |
| Inputs | Exact revisions or local input digest |
| Scope | Changed paths and affected contracts |
| Evidence | Decisive source, tests, generated output or execution |
| Status | Covered, blocked or pending, with material limits |
| Owner | Only when work is delegated or resumed |

Completion requires every changed path and material risk to be covered or named as a gap.
It does not require a separate entry for every specialist area.
Missing execution limits runtime claims.
Missing source or required contract evidence prevents a clean result.

### Best-effort completion

Finish independent work when another scope is blocked.
Preserve valid evidence and try another method when it can add information or resolve the cause.
Do not repeat a command, reviewer handoff or setup attempt when no relevant input changed.

| Blocker | Adaptive response | Required boundary |
| --- | --- | --- |
| Tool or setup failure | Diagnose the cause, use another supported tool or repair the prerequisite within existing permissions | Keep inputs and configuration comparable |
| Reviewer or context limit | Continue directly, narrow the unfinished scope or transfer it with saved evidence | Do not repeat completed work |
| Missing API or test evidence | Finish independent source review and retain proved observations | Do not claim the missing classification |
| Benchmark unavailable | Finish source performance review and report measurement limits | Do not make measured claims |
| Partial publication | Resume through Review Delivery with confirmed write IDs | Do not repeat analysis to repair a write |

Install tools or acquire dependencies only when required and permitted.
Do not change reviewed inputs, lockfiles or requested configurations to make evidence succeed.
Stop recovery when alternatives no longer add evidence or the expected value does not justify their cost.

When generated API evidence is missing, source observations can still describe proved contract concerns.
Label them as source-based and keep the API classification incomplete.
They cannot determine a clean human-review decision.

Deliver supported findings after useful independent work and recovery finish.
Name every material gap and the smallest missing prerequisite.
An incomplete review has no clean verdict or approval.

### Establish applicability

Trace changed definitions, generated inputs, features, dependencies and shared test inputs to their consumers.
Expand scope through a concrete contract connection.
Stop where evidence shows that the selected configuration is unaffected.

| Applicability | Action |
| --- | --- |
| Relevant | Review the connected scope |
| Unknown | Resolve the question or retain a coverage gap |
| Absent with evidence | Record the boundary and skip specialist work |

Filenames, diff size and unchanged test files do not prove absence.

### Choose review owners

Review connected work directly unless delegation provides useful parallelism, isolation or specialist context.
Keep one owner for a continuous trace by default, but split or transfer work when that improves evidence quality.
Combine areas that need the same evidence and return distinct findings only when their conclusions differ.
Do not duplicate completed analysis to seek agreement.

Inherit the session's model, tools, permissions and execution limits.
Newer models can choose different decomposition and tool strategies while preserving the evidence outcomes in this file.

### Combine test review

The test contract must be established independently of the changed implementation and changed expectations.
The reviewer can choose reading order and tools.
Changed code or author rationale cannot be the sole proof of intended behavior.
Use base tests, public contracts, requirements and callers to identify preserved, removed or newly approved behavior.

Share relevant evidence between `review-tests` and `review-integration-tests`.
Return separate classifications when both skills apply.
Do not require a separate context merely because implementation was already read.
If prior context creates uncertainty, state the limit and seek independent evidence rather than restarting by default.

### Protect API contexts

Public API classification must be justified by complete, matching generated surfaces.
Source, discussion and docs can explain impact or disprove an interpretation.
They cannot replace missing generated output or turn an incomplete comparison into no change.

An isolated reviewer context is optional.
Track which claims come from generated output and which come from source.
When using `review-public-api`, follow its evidence rules and keep renderer limits explicit.
Docs can remove or narrow a design concern when they show intended use.

If another source disproves an output-based claim, remove or narrow the delivered claim.
Do not alter the underlying capture or hide a remaining classification gap.

### Bound handoffs and follow-ups

Give delegated work a clear objective, exact inputs, material scope, available evidence and required result.
Include repository rules and execution limits.
Keep large logs and inventories in artifacts.

Let the receiver choose its internal method.
Request follow-up only for a precise gap, contradiction or changed input.
Reuse saved valid work and avoid sending another reviewer to repeat the same scope.

### Schedule evidence by dependency

Start work when its required inputs are available.
Handle ready results and material follow-ups before optional preparation.
Run independent work in parallel when this does not violate measurement isolation or shared-input safety.
Use completion notifications instead of progress polling.

## Shared setup and reuse

Never mutate reviewed inputs or switch a shared checkout to produce evidence.
Collect package ownership and reusable discovery once when practical.
Reuse an artifact when its inputs, scope, configuration and tool behavior match the new use.
If inputs change, rebuild only dependent evidence.

Producers own their detailed freshness rules:

- `review-public-docs` owns rustdoc JSON generation and reuse.
- `review-public-api` and `review-public-api-changes` own public-surface evidence.
- `review-integration-tests` owns integration-test inventory.
- `metabench-compare` owns benchmark setup, isolation and measurement.

## Package presence

Map affected packages and library targets at base and head.
Use available metadata, manifests and history.
Prove absence from a complete inventory, not from a failed build.
Track added, removed, renamed and moved libraries.

Include consumers only when a changed input can affect their public surface.
Unresolved connections remain coverage questions.

## Public API evidence

When a Rust library surface can change, provide complete base and head captures for each affected configuration.
Keep package, features, target, toolchain and rendering behavior comparable.
Use any supported producer that satisfies the specialist skills' evidence requirements.
Record tool identity and material renderer limits.

Do not prescribe one command in the coordinator.
If capture generation fails, continue independent work and apply [best-effort completion](#best-effort-completion).

## Integration-test evidence

When integration-test contracts can change, inventory affected targets and their shared inputs at both revisions.
Include discovery and execution configuration that can add, remove or skip a target.
Use base expectations as independent evidence.
Keep full inventories in artifacts.

## Benchmark evidence

Run benchmarks when the user asks or when measured performance evidence is material and an eligible target exists.
Give `metabench-compare` exact inputs, relevant repository rules and existing discovery.
The helper chooses commands and owns measurement isolation.

Do not delay ready review findings for pending measurements.
Record their owner, inputs and state, then use Review Delivery for an authorized follow-up.

## When the head moves

Revalidate against the new inputs before publication.
Use the delta and affected-contract graph to decide what work must repeat.
Reuse completed work only when its inputs and dependencies still match.

| Change | Minimum response |
| --- | --- |
| Retarget, merge-base change, rebase or force push | Rebuild scope and reassess affected work |
| New commits | Review the delta and every contract it can affect |
| Target-tip-only change | Refresh integration and CI claims |
| Local input digest change | Review the changed local inputs |

For an incremental refresh, preserve the original review revision and record the newly assessed revision.
Revalidate every surviving finding against current code and discussion.
Remove fixed findings, update moved evidence and mark uncertainty as a limit.
Refresh compatibility reports only when their evidence still matches.

If inputs move during publication, preserve confirmed provider IDs and return pending work for revalidation.
Never present historical findings or measurements as current without explicit permission.
