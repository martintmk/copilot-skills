---
name: pr-review-queue-github
description: >
  Review eligible GitHub PRs labeled human-review-required or individually
  requested for the target, sequentially using native GitHub Copilot app
  session automation and linked PR sessions. Use for "monitor and review GitHub
  PRs", "review my PR queue" or recurring GitHub reviews. Delegates selection
  policy to pr-review-eligibility and reviews to review-lens. Not for Azure
  DevOps, discovery digests or feedback/fix loops. Editing or installing this
  skill does not start monitoring.
---

# GitHub PR Review Queue

**Fetch -> [eligibility](../pr-review-eligibility/SKILL.md) -> native PR session
-> [Review Lens](../review-lens/SKILL.md).** One PR runs at a time.

The app owns recurrence, workspaces, session history, status and notifications;
GitHub owns PR and submitted-review facts. Do not build queue cache files,
monitor locks, manual worktrees, provider adapters or an extra review runner.
Review Lens already owns specialist coordination and final delivery.

## Setup

1. Use one dedicated coordinator session. Confirm explicit `github.com`
   repositories (`owner/repo`); never infer scope from the checkout or access.
   Default the target to `martintmk` and resolve its stable identity separately
   from the authenticated posting actor with `gh`. Never impersonate the target.
   Reuse confirmed setup from this session on later wakes.
2. For recurrence, confirm cadence and whether to scan immediately. Use
   `get_session_automation`, then `save_session_automation`, and read it back.
   Its prompt invokes this skill with the confirmed scope/target in **this same
   coordinator session**, preserving context. Do not use `save_workflow`, which
   starts a new session each run, or an external scheduler. Never overwrite an
   unrelated automation or establish a second owner for overlapping queue scope.
   Scheduled wakes reuse setup; they never register or reschedule themselves.
   One-shot requests neither ask for cadence nor change a schedule.
3. Keep setup and concise progress in native session history. An explicitly
   empty scope clears only this coordinator's automation; scope changes retain
   earlier outcomes. Missing setup or required app/`gh` capability blocks;
   report the specific gap, not a speculative preflight or shell replacement.
4. When replacing the old `pr-review-queue`, first stop its schedule and settle
   in-flight work with the user's authorization. Retain its audit files and use
   verified outcomes to seed session history before new reviews. Do not run both,
   discard uncertain delivery or build another state migration system.

## Scan and continue

1. Reconcile the previously dispatched PR first. Use `get_session` for its saved
   session ID; use `get_sessions_status` when live attention/ownership is unclear.
   If it is still working or awaiting input, do not dispatch another PR or send
   another review prompt; surface any human gate. A notification or idle status
   alone is not completion.
   Continue an unfinished finite batch before collecting another.
2. Capture one UTC `scanAt`. Use scoped, paginated `gh` reads to collect the
   **union of open PRs labeled `human-review-required` and open PRs individually
   requesting the target**, including drafts and old target-authored PRs.
   Deduplicate by repository/PR identity. Refresh previously tracked PRs omitted
   from both lists to distinguish loss of admission signals from closure/merge;
   absence from discovery is not retirement.
   Fetch only needed policy facts: current labels, individual requests, their
   applicable `review_requested` timeline event IDs/times, and submitted reviews
   with stable reviewer IDs and provider actor/app types.
   Use stable identities; PR creation, `updatedAt` and polling are not request
   times. Failed or incomplete required reads never mean an empty queue.
3. **Invoke `pr-review-eligibility` and apply it to each candidate**, supplying
   fetched facts and its latest outcome. Do not duplicate or alter its policy
   here. Record retirements, blocked/deferred decisions and one finite due batch
   in the coordinator conversation. Due PRs with current individual requests
   sort first, oldest event first; other work sorts by first observed due time
   retained in this session. Break ties by repository/PR identity.
   Unknown request evidence that could change priority blocks selection.
   New arrivals or changed snapshots wait for the next batch.
4. For the next due PR, refresh decisive facts and UTC time; reapply eligibility.
   Find its saved PR session or discover it with `list_sessions_and_chats`;
   confirm the repository/PR and ownership with `get_session`. Never take over
   an unrelated or busy session. Record the selected snapshot/request in
   coordinator history before sending work.
5. Before dispatching the review, create or update one top-level GitHub issue
   comment owned by this queue using the stable marker
   `<!-- pr-review-queue:preparing -->`. The visible text must state that review
   preparation has started, include the repository/PR identity and pinned head,
   and say that the submitted review will contain the result. Reuse the marked
   comment on retries or newer preparations instead of creating duplicate
   status comments. Read the comment back and verify its ID, author and head
   before continuing; an uncertain comment write blocks dispatch until it is
   reconciled.
6. Use `open_pr_session` for a new linked workspace, with
   `kickoff: { prompt: <handoff below>, mode: "autopilot" }` and top-level
   `notify_on_idle: "always"`, `coordinate_with_creator: true`.
   Save its returned session ID. Reuse an idle queue-owned session with
   `send_session_message` (`delivery_mode: "immediate"`, `mode: "autopilot"`).
   Only one review handoff may be outstanding. Return control and resume from
   native child notifications or the next scheduled wake; do not busy-poll.
7. Accept a reported publication only after confirming the actual GitHub submitted
   review, its actor, commit, intended event and coverage result. Native pending
   review drafts, local reports and prose success are **not publication**.
   Save the child ID, reviewed head/target, handled request event, review ID/URL,
   coverage/blockers and any newer deferred work in coordinator history before
   advancing. No-write skips/deferrals are not completion; incomplete coverage
   stays debt under the eligibility policy.
8. Let GitHub handle review-request state as part of normal review submission.
   **Never remove reviewers manually.** Record only the request event actually
   processed; a newer request/head is not satisfied by the old result. Re-read
   current facts after delivery and retain changed work for the next batch.
   Report reviewed PRs, deferred work and blockers after the finite batch ends.

## PR session handoff

Supply confirmed scope, PR URL/identity, scoped target and posting actor, current
labels and individual requests, exact head and base repository/ref, selected
request event/reason, prior outcome/debt and any matching factual artifacts.
Use this bounded instruction:

> Invoke `pr-review-eligibility` with these facts and current metadata. If due,
> invoke `review-lens` for one full, fresh review in authorized posting mode.
> Let Review Lens supply specialist context and dedicated high-reasoning agents.
> Preserve inherited session permissions, execution limits and delivery rules.
> Verify checkout/evidence match the pinned head; reused workspaces are not
> automatically current. Preserve existing edits; never reset or clean to force
> a match. Recheck eligibility with fresh UTC time, labels, individual requests
> and the selected head/target immediately before publication; moved inputs are
> deferred, not a best-effort descendant finding refresh.
> Target-authored and poster-authored PRs remain
> COMMENT-only. Carry these publication constraints to the sole `review-delivery`
> worker through Review Lens, not a second queue-specific posting stage.
> Return the submitted review ID/URL, covered snapshot, complete/incomplete/failed
> outcome, handled request event and any deferred newer work to the coordinator;
> for a no-write skip/deferral, return its reason instead. Finish or stop all owned
> review/delivery workers before reporting the terminal outcome.
> Do not edit code or labels, push, reply to discussion, resolve threads, remove
> reviewers, enable Agent Merge, invoke feedback-autonomy or create automation.

## Recovery

Recover missing context from the owning sessions' history with
`session_store_sql` and GitHub read-back, not a second cache. Reuse the original
child for read-only clarification of an incomplete outcome. An uncertain dispatch or write must be
reconciled before retrying; never rerun a review merely because its reply was lost.
A proven PR-local, zero-write failure may be recorded and skipped for this batch,
without marking it reviewed; auth, collection and uncertain-write failures stop
the batch. For an unrecoverable recurring run, clear only its own automation,
retain confirmed setup/history and report what needs user intervention.
