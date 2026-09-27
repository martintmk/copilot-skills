# Rewrite risks and source

Adapted from Stephen Toub's
[Migrating the GitHub Copilot runtime to Rust using Copilot](https://github.blog/ai-and-ml/generative-ai/migrating-the-github-copilot-runtime-to-rust-using-copilot/).
The strategy, fleet coordination, review, regression and lessons-learned sections
inform this skill. The templates and readiness rules are a synthesis for other
projects, not a reproduced prompt or a guarantee of comparable results.

Load this guide when planning a rewrite, dependency replacement or another
change with comparable compatibility risks. Select relevant rows and translate
them into concrete requirements and evidence for the actual system.

## Carry forward the reasoning, not the incidentals

The case study used a behavior-preserving, in-place migration: small production
replacements, a shipping pilot, dependency-ordered work, temporary interop and
continuous consumer validation. Its human coordinator owned the destination,
ambiguous decisions, review of high-risk changes and final acceptance. The
general lesson is a tighter engineering feedback loop, not a bigger kickoff
prompt followed by unsupervised code generation.

Neither Rust, atomic replacement, a particular agent product/model, a fleet size,
nor the reported time, token cost or performance improvement is a universal
recommendation. Dual-running can be appropriate where state/effects are safely
isolated; it can be dangerous for a coupled stateful subsystem. Choose from
requirements, not from the article's outcome.

The article also describes failure-prone actions such as taking a peer's
unfinished work and bypassing a schema gate. Those are cautionary examples,
not actions to reproduce. Its history-rewriting and automatic PR operations
are not authority for a generated instruction pack.

## Compatibility probes

| Risk | What the plan must establish | Useful proof |
| --- | --- | --- |
| Incomplete translation | Every supported ingress, feature, callback, error path, I/O path and configuration reaches the replacement; not only the hot or pure logic. | Behavior-to-entrypoint coverage map and consumer/E2E scenarios exercising the actual packaged path. |
| Value/serialization differences | Numeric representation and precision, missing/null/empty values, truthiness, defaults, errors, ordering and wire shapes remain compatible. | Existing consumer fixtures plus edge-case round trips through strict clients and persisted-data readers. |
| Dependency differences | Equivalent-looking libraries may disagree on parsing, globs, encodings, malformed input, defaults or error responses. | Run the same representative and adversarial corpus through the old and replacement boundaries; approve any intended difference. |
| Ambient host behavior | Time zone, working directory, environment, credentials/settings and host patches have an explicit owner and capture/refresh moment. | Scenarios varying host context and async ordering, including each supported platform's process/console behavior. |
| Paired effects | A state update still performs its required cancellation, event, persistence, projection or callback counterpart. | Trace a full operation to all observable effects, including failure and cancellation between intermediate states. |
| Lifecycle and concurrency | Handle lifetimes, disposal, cancellation, queue draining, callback completion, generation changes and event order retain their contract. | Deterministic interleavings around creation/start/cancel/dispose; assert no orphaned work, missing result, leak or hang. |
| Interop and hosting | Blocking work stays off sensitive host threads; scheduling, ownership and failure boundaries are explicit in both callback directions. | Host responsiveness and callback/lifetime scenarios across each supported transport and embedding mode. |
| Persistence and recovery | Previously written state remains readable/resumable; deployment rollback respects any newly written format or side effects. | Old-data fixtures, restart/resume tests, and compatibility checks for the actual rollback path. |
| Drift during migration | Incoming behavior, guards and tests survive integration even where legacy files are deleted or conflicts resolve cleanly. | Account for relevant upstream changes against replacement paths; rerun affected consumer checks on the integrated revision. |
| Lost efficiencies | Caching, streaming bounds, async waits and resource limits are not replaced by copies, repeated serialization, polling or unbounded concurrency. | Comparable workload measurements of the changed path, including latency, CPU, memory and retention where relevant. |
| Incorrect oracle | New tests do not simply assert the replacement's mistaken behavior, and existing tests/gates have not been weakened. | Characterization against the baseline, protected expectations, and independent review of any authorized oracle change. |
| Stranded scaffolding | Temporary shims and fallback dependencies disappear when their consumers migrate; permanent external contracts remain. | Owned removal tasks plus artifact/dependency inspection and exercised production paths; if required, run without the old runtime available. |

Static analysis catches many wiring mistakes but cannot establish this whole
contract. Combine checks with different failure modes. New implementations and
their new tests can agree with each other and still violate consumer behavior.

For performance, distinguish the changed subsystem from unrelated latency:
use a controlled workload when appropriate and preserve a realistic consumer
scenario too. Record confounding changes, configurations and measurement limits.
Do not attribute a whole-system improvement solely to a language choice.

## Turn symptoms into bounded steering

| Observed symptom | Next instruction should demand |
| --- | --- |
| "Most logic is ported" but I/O or callbacks still use the old engine | Enumerate the remaining production routes within the assigned boundary, finish their wiring, and close their requirement evidence or explicit blocked decisions. |
| Compilation is green but a consumer contract fails | Preserve the consumer oracle, isolate the semantic difference and restore the agreed contract; do not waive the check. |
| Adjacent workers need the same stateful hub | Have the coordinator assign one integration owner and a handoff boundary; do not seize the peer's unfinished changes. |
| Every worker launches a full build | Route expensive checks through the agreed owner/capacity policy while independent low-resource work continues. |
| A passing slice regresses after upstream changes | Identify the changed contract or lost incoming behavior and re-establish affected evidence on the integrated revision. |
| A rewrite quietly includes an attractive redesign | Separate it from parity work unless already approved; take the scope/behavior decision back to its owner. |
| The same escape hatch appears repeatedly | Repair the instance, then propose a narrowly scoped protected check, reusable rule or evaluation that catches the failure independently. |
