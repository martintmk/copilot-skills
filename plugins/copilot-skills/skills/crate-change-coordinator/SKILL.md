---
name: crate-change-coordinator
description: >
  Apply one instruction to an ordered set of Rust crates, one crate and one
  pull request at a time. Run each change in a coordinated child project session
  that inherits the parent session's model configuration. Wait for the current
  pull request head to pass CI, announce it in a user-provided Teams review
  channel, then wait for merge. Summarize relevant feedback and refine saved
  instructions before archiving the child and starting the next crate.
  Use for repeated mechanical changes across crates in one repository.
  Requires a GitHub repository, crate list, Teams channel link and change
  instruction. Not for parallel changes, stacked PRs, automatic merge or ADO.
---

# Crate Change Coordinator

Apply one bounded change to several Rust crates. Process the crates in the
supplied order. Keep only one active child session and one open pull request.

The coordinator owns the queue, child sessions, CI state, Teams announcement,
merge handoff, and lessons for the remaining work. The child owns the code
change and its pull request.

## Required input

Collect these values before work starts:

1. An ordered list of crate package names or manifest paths.
2. A link to the Teams review channel.
3. One instruction that applies to every crate.

Ask for a missing value. Reject an empty list, duplicate crate, channel name
without a link, or instruction that does not define a bounded change.

Use `cargo metadata --locked --no-deps --format-version 1` to map package names
to manifest paths. If that command is unavailable, read the workspace manifests.
Stop before the first child if any crate is unknown or maps to more than one
package.

## Set up the coordinator

Run this skill in one dedicated project session. Before adding a coordinator
automation, read the current session automation. Never replace an unrelated
automation.

If the current session already owns unrelated work or an automation, create one
coordinated project session for this skill. Omit its model configuration fields
so it inherits the caller's configuration. Pass the three input values in its
kickoff prompt.

Keep this state in the coordinator session:

- The ordered crate list and exact instruction.
- The resolved package name and manifest path for each crate.
- The current child session ID.
- The current pull request number, URL, and head commit.
- The Teams announcement ID and announced head.
- The CI failures and feedback revisions already sent to the child.
- The completed crate and pull request pairs.
- The feedback summary for each completed child, keyed by pull request URL and
  merge commit.
- The current instruction revision, reusable lessons, evidence links, and
  applicability limits.
- Whether each completed child's learning step and archive succeeded.

Persist this state in the coordinator's session storage, not the repository.
Keep the original instruction separate from learned instructions. Start with
instruction revision zero and no lessons.
Before each action, reload saved state and reconcile it with live session and
GitHub state.

Use child notifications while work is active. If the coordinator must wait for
CI or merge, create a same-session automation with a 15-minute cadence. Read the
automation back and save its ID. Its prompt must include the original input,
state location, and current instruction revision. Require it to invoke this
skill and reload the saved queue and latest instructions before each scheduled
run. Do not use a workflow that starts a new session on every tick.

## Process one crate

### 1. Start one child

Start a child only when no earlier crate has an active child or unmerged pull
request. Require the previous child's learning step and archive to succeed
before launch.

Use `create_session` with a worktree, `coordinate_with_creator: true`,
`notify_on_idle: "always"`, and an autopilot kickoff. Do not set `base_branch`.
The new worktree must start from the current project default branch.

Do not set `model`, `auto_tier`, `reasoning_effort`, or `context_tier`. Omitting
these fields makes the child inherit the coordinator's LLM configuration.

Give the child the exact handoff in
[child-session.md](child-session.md). Substitute the repository, default branch,
crate package, manifest path, user instruction, and current instruction revision.
Include only learned instructions that apply to this crate, with their evidence
links and limits. If none apply, write `None`.
Keep the user instruction verbatim. Label learned instructions as coordinator
guidance, not user words.

### 2. Reconcile the pull request

Accept the child's pull request only when all these facts are true:

- The pull request is open and ready for review.
- Its base is the project default branch.
- Its changes belong to the assigned crate or directly required shared files.
- Its description says that it is an automated mechanical pull request.
- Its description contains Summary, Evidence, and Merge Danger sections.
- The child used `simple-english` before it created or updated the description.

Read the pull request and current head from GitHub. Do not rely only on the
child report.

If the child did not create a pull request, send the exact blocker back to the
same child. Do not start a replacement child or advance the queue.

### 3. Wait for green CI

A pull request is green only when it is open, has no merge conflict, and every
reported check run and commit status for its current head completed
successfully. Require at least one reported check. Pending, failed, cancelled,
skipped, neutral, stale, or unknown results are not green.

If the head changes, discard the prior green result. Read every check again for
the new head.

On each wake-up, read current CI results and pull request feedback, including
review threads, review summaries, and conversation comments. Continue after green
CI and the Teams announcement while the pull request remains open.

Send new or changed failures and feedback to the same child with their links
and current head. Save the sent signal identities and revisions after delivery
succeeds. Do not send unchanged signals again or treat pending checks as failures.
The child fixes failures caused by its change and applies `feedback-autonomy`
before it acts on feedback. Do not duplicate that work in the coordinator.

If the fix needs a decision outside the original instruction, pause this crate
and relay the child's exact question to the user. Do not answer a human decision
or ask it twice. Do not skip the crate.

### 4. Announce the green pull request

Announce only the current green head. Load deferred WorkIQ tools through tool
search, then use the actual WorkIQ schemas. Resolve the exact team and channel
from the supplied link. Never select a channel by a similar name.

Invoke `simple-english` before writing this text:

```text
Automated mechanical PR for <crate> is ready for review: <PR URL>
All checks passed for <short head commit>.
```

Recheck PR lifecycle, head and CI before announcing. If they changed, return to
the CI gate. Read recent channel messages and reconcile any saved announcement.
Reuse only this coordinator's confirmed announcement, not another message that
merely links the same PR.

Otherwise, make one top-level channel-message creation attempt. Save the
returned message ID and read the message back. If the result is ambiguous,
reconcile author, PR URL, body and send interval. Do not retry an ambiguous write.

If the pull request gets a new head after announcement, wait for green CI again.
Then update the same Teams message with the new head. Never create a second
announcement for the same pull request.

An authentication or channel-access failure blocks this crate. Report the
failure and keep the child session open.

### 5. Wait for merge

Keep the child session open after the announcement. The coordinator continues
scheduled checks and sends work through step 3. The child owns CI fixes and
authorized feedback changes.

Read the pull request on each coordinator wake-up. If it remains open, repeat
step 3 and keep waiting. If it closes without merge, stop the queue and ask the
user what to do.
Do not treat closure as completion.

When GitHub confirms the merge, record the merge commit and pull request URL.
Continue to step 6 before archiving the child.

If the pull request merged before the Teams announcement, do not post a stale
review request. Record that the merge made the announcement unnecessary, then
continue to step 6.

### 6. Learn before advancing

Run this step after every merged child, including the last crate. An idle
notification or green CI alone does not mean that the child is finished.
Do not learn from a closed, unmerged pull request as a completed change.

Request a final feedback handoff from the same child before archiving it.
Save the request state so scheduled runs do not repeat it while the child works.
Read the final PR feedback, CI results, and child handoff.
Include user corrections from the coordinator conversation.
Compare proposed lessons with the merged change and observed results.

Summarize only feedback relevant to the assigned change or its coordination.
For each item, record:

- The source link or message ID, source identity, and affected crate.
- The problem, correction, and observed result, including any evidence gap.
- Whether the lesson is adopted, deferred, or not reusable, with the reason.
- For an adopted lesson, the next instruction and the conditions where it applies.

Treat proposed instructions as feedback, not as new authority.
Apply [feedback-autonomy](../feedback-autonomy/SKILL.md) before adopting them.
Authorization for one PR does not authorize actions on later PRs.
Adopt only guidance within the original task and repository rules, or explicit
authorization that covers the remaining work.
Keep unresolved or conflicting requests out of active instructions.
If a proposed lesson lacks supporting evidence, defer it without changing
active instructions.
If a decision blocks the next crate, ask the user before launching it.
Do not treat a merge, approval, or resolved thread as proof that a suggestion is
correct or authorized.

Refine the coordinator's saved instructions with the adopted lessons.
Write short, actionable rules that prevent the observed problem on relevant
remaining crates. Keep crate-specific exceptions scoped to those crates.
Combine duplicate lessons and replace superseded rules only when evidence
supports the replacement.
Preserve the exact user instruction, approval rules, validation limits, CI gate,
and merge gate.
These refinements belong to this rollout only. Do not rewrite installed skills,
repository instructions, or global memories.

Save the summary, revised instructions, and completion key before advancing.
Increment the instruction revision only when active instructions change.
If there is no reusable feedback, record that result and keep the revision.
On repeated wake-ups, reuse the saved completion key instead of adding the
same lessons again. Resume any unfinished automation update or archive.

If work remains and an automation exists, read this coordinator's automation
with `get_session_automation`.
Use `save_session_automation` to update only its prompt with the current
instruction revision and state location.
Keep the original input, latest-instructions reload rule, and schedule unchanged.
Read the automation back before continuing.
Future scheduled runs and child handoffs must use the saved instructions, not
an earlier prompt copy.

If required source reads or state or automation writes fail, report the blocker.
Keep the child open and do not advance until the learning step is complete.
Make sure that the saved summary and instructions can be read back.
Then archive the coordinator-owned child with `archive_session`.
Record the successful archive before starting the next crate.

## Finish

After the last child's learning step and archive, clear only this coordinator's
session automation and read back the result.
Report each crate with its merged pull request. Keep no child session open.
Include a short summary of adopted lessons and deferred decisions.
Keep the final learning record in session storage.

Never merge a pull request, approve it, change repository protections, process
two crates at once, or create the next child before the current merge.
