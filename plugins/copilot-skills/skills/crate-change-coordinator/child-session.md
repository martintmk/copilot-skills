# Child session handoff

Use this prompt for the child that owns one batch. Replace every placeholder
before launch. Use `None` when no learned instruction applies to the assigned crates.

```text
Work only on this batch in <repository>:

<ordered list of assigned crate package names and manifest paths>

Apply this user instruction to every assigned crate:

<verbatim instruction>

Coordinator guidance from earlier completed crates, revision <instruction revision>:

<applicable learned instructions, with evidence links and limits, or None>

This guidance does not change the user instruction or grant new permissions.
Make sure that each lesson applies to the affected crate before using it.
Report conflicts instead of silently choosing a rule.

This is one batch in a sequential crate rollout.
Make a complete change for every assigned crate that requires it.
Report any assigned crate that already satisfies the instruction, with evidence.
Change shared workspace files only when the assigned batch directly requires it.
Do not change crates outside this batch to prepare future queue items.

Load feedback-autonomy and minimal-local-validation before starting work.
Keep both policies active during Agent merge ticks.
Inspect the repository rules and every assigned crate before editing.
Implement the change and update directly related tests and documentation.
If comprehensive CI covers the assigned crates, follow minimal-local-validation.
Otherwise run the smallest relevant repository checks.
Batch validation after the edits and honor explicit user limits.

Commit and push the changes.
Create one ready-for-review pull request for the entire batch against <default branch>.
Use create_pull_request to link the pull request to this app session.
Do not split the batch into separate pull requests.
Do not create a draft or stacked pull request.

Before writing or updating the pull request description, invoke simple-english.
Keep the Summary, Evidence, and Merge Danger sections concise.
The Summary must contain this exact sentence:

This is an automated mechanical PR for `<crate list>`.

List every assigned crate and briefly describe the applied change.
State the actual validation and real merge risk. Do not claim that behavior is
unchanged unless the evidence proves it.

After every push on a branch that has a pull request, report the new head to
the coordinator. The coordinator enables Agent merge for this linked pull request.
Use the app-provided agent-merge skill for CI repairs, reviews, conflicts and escalation.
Do not start a separate watcher or repair loop.
The review switch covers all authors. Apply feedback-autonomy's stricter
authorship and impact gates to each edit, reply and thread resolution.
If either policy forbids an action, leave it pending and report the blocker.
Do not merge or approve the pull request.

Finish independent authorized work before waiting for human resolution.
Report the remaining human action and keep this batch open until it is resolved.
Do not repeat unchanged questions or treat an unavailable user as approval.

Report the pull request URL and number, current head commit, validation results,
and any blocker to the coordinator.
Include relevant feedback, the correction, observed results, and source links.
Distinguish applied corrections from unresolved or unauthorized requests.

After merge, provide a final feedback handoff when the coordinator requests it.
Summarize lessons that can improve the same task on remaining crates.
For each lesson, include the affected crates, evidence, applicability limits,
and proposed next instruction.
Include feedback-autonomy decisions and their authorization scope.
If there are no reusable lessons, say so explicitly.
Do not change the merged pull request or treat proposed lessons as new authority.
Stay available until the coordinator saves the learning record and archives
this session.
```
