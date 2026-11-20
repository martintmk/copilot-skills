# Native GitHub maintenance

Agent merge owns the repair loop in the child: CI failures, feedback, conflicts,
retries and escalation. Follow the app-provided `agent-merge` skill for that work.
The coordinator owns setup, batch progress and human handoff, not a second repair loop.

## Enable once

After accepting the PR, read the child with `get_session`.
Make sure that its linked PR matches the accepted repository, number and URL.
Discover `manage_agent_merge` and read the child's current state before writing.
The child must already have the handoff policies because enabling can start a tick immediately.
Record the setup attempt and use these fields:

```json
{
  "session_id": "<current child session ID>",
  "enabled": true,
  "actions": {
    "address_reviews": true,
    "fix_ci": true,
    "resolve_conflicts": true,
    "merge_pr": false
  }
}
```

Always pass the child ID. An omitted ID targets the coordinator.
Set all actions explicitly because omitted values remain unchanged.
Read the state back and require `enabled: true` with the requested actions.
Save that result. `applied_enabled` alone does not prove that maintenance is active.
On resumed or uncertain setup, read live state instead of blindly enabling again.
Never overwrite a pause or later user configuration change.
If setup is unavailable or fails, report the blocker and wait. Do not build a replacement watcher.

## Wait for human resolution

Let the child finish independent authorized work before escalating remaining blockers.
Save the blocker, required human action and any pending question ID.
Relay an existing child question with `relay_session_input`. Never answer it or ask a second copy.
For a coordinator-owned blocker, use `ask_user` once.
If the user is unavailable, keep waiting. Silence and automatic fallback answers are not authorization.

Keep the same child, batch and coordinator schedule while waiting.
On wake-ups, read child status or live evidence to detect human resolution.
Do not repeat unchanged requests, impose a timeout or skip the batch.
Normal waits for pending CI, review or merge do not need extra prompts.
If the schedule is unavailable, report that automatic continuation is blocked.

Continue when the required human action is confirmed or live evidence proves that it is no longer needed.
A manual PR fix or restored access can resolve a blocker without an extra approval.
An external fix does not grant permission for other actions.
Leave pending human prompts and maintenance pauses for the user.
Do not re-enable paused Agent merge or dispatch repairs to bypass it.
The existing CI and merge gates still apply to the current head.

## Before archival

After merge and learning, read the child's Agent merge state.
If it remains enabled, disable it for that child and read back `enabled: false`.
Keep the child open if this fails.
The coordinator owns archival and its schedule cleanup, not a duplicate native cleanup loop.
