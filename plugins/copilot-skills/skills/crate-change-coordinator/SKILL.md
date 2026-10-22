---
name: crate-change-coordinator
description: >
  Apply one instruction to an ordered set of Rust crates, one crate and one
  pull request at a time. Run each change in a coordinated child project session
  that inherits the parent session's model configuration. Wait for the current
  pull request head to pass CI, announce it in a user-provided Teams review
  channel, then wait for merge before archiving the child and starting the next
  crate. Use for repeated mechanical changes across crates in one repository.
  Requires the crate list, Teams channel link, and change instruction. Not for
  parallel changes, stacked pull requests, or automatic merge.
---

# Crate Change Coordinator

Apply one bounded change to several Rust crates. Process the crates in the
supplied order. Keep only one active child session and one open pull request.

The coordinator owns the queue, child sessions, CI state, Teams announcement,
and merge handoff. The child owns the code change and its pull request.

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
- The completed crate and pull request pairs.

Do not store coordinator state in the repository. Before each action, reconcile
saved state with the live session and GitHub state.

Use child notifications while work is active. If the coordinator must wait for
CI or merge, create a same-session automation with a 15-minute cadence. Read the
automation back and save its ID. Its prompt must invoke this skill with the
original input and continue the saved queue. Do not use a workflow that starts
a new session on every tick.

## Process one crate

### 1. Start one child

Start a child only when no earlier crate has an active child or unmerged pull
request.

Use `create_session` with a worktree, `coordinate_with_creator: true`,
`notify_on_idle: "always"`, and an autopilot kickoff. Do not set `base_branch`.
The new worktree must start from the current project default branch.

Do not set `model`, `auto_tier`, `reasoning_effort`, or `context_tier`. Omitting
these fields makes the child inherit the coordinator's LLM configuration.

Give the child the exact handoff in
[child-session.md](child-session.md). Substitute the repository, default branch,
crate package, manifest path, and user instruction. Keep the user instruction
verbatim.

### 2. Reconcile the pull request

Accept the child's pull request only when all these facts are true:

- The pull request is open and ready for review.
- Its base is the project default branch.
- Its changes belong to the assigned crate or directly required shared files.
- Its description says that it is an automated mechanical pull request.
- The child used `pr-description`, then `simple-english`, before it created or
  updated the description.

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

Send current CI failures to the same child. The child fixes failures caused by
its change and invokes `track-pr` after every push. The child applies
`feedback-autonomy` before it acts on pull request feedback. Do not duplicate
that work in the coordinator.

If the fix needs a decision outside the original instruction, pause this crate
and ask the user. Do not skip the crate.

### 4. Announce the green pull request

Announce only the current green head. Load deferred WorkIQ tools through tool
search, then use the actual WorkIQ schemas. Resolve the exact team and channel
from the supplied link. Never select a channel by a similar name.

Invoke `simple-english` before writing this text:

```text
Automated mechanical PR for <crate> is ready for review: <PR URL>
All checks passed for <short head commit>.
```

Read recent channel messages first. If the current user already posted a message
for the same pull request, save its message ID instead of posting another one.

Otherwise, make one top-level channel-message creation attempt. Save the
returned message ID and read the message back. If the result is ambiguous, read
the channel and reconcile by pull request URL. Do not retry an ambiguous write.

If the pull request gets a new head after announcement, wait for green CI again.
Then update the same Teams message with the new head. Never create a second
announcement for the same pull request.

An authentication or channel-access failure blocks this crate. Report the
failure and keep the child session open.

### 5. Wait for merge

Keep the child session open after the announcement. It continues to own CI and
authorized feedback changes through `track-pr`.

Read the pull request on each coordinator wake-up. If it remains open, keep
waiting. If it closes without merge, stop the queue and ask the user what to do.
Do not treat closure as completion.

When GitHub confirms the merge, record the merge commit and pull request URL.
Archive the coordinator-owned child with `archive_session`. Start the next
crate only after the archive succeeds.

If the pull request merged before the Teams announcement, do not post a stale
review request. Record that the merge made the announcement unnecessary, then
archive the child.

## Finish

After the last merge, clear only this coordinator's session automation and read
back the result. Report each crate with its merged pull request. Keep no child
session open.

Never merge a pull request, approve it, change repository protections, process
two crates at once, or create the next child before the current merge.

