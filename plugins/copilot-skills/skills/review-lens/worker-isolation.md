# Review Worker Isolation

Every invoked review sub-skill gets a **fresh agent session/context**, including
standalone reviews, docs retrieval and final delivery. `review-lens` coordinates;
it does not perform specialist passes itself. Supporting documents are not
additional skills, except the explicitly isolated API-filtering stage.

## Entry and dispatch

1. Before investigation, retrieval or delivery, the caller starts a new
   subagent using the fresh-agent facility (for example, `task`). Assign exactly
   one skill/stage, scope and revision snapshot in its prompt. Loading a skill
   into the current chat, switching worktrees, or compacting a conversation
   does not create isolation.
2. If the trusted caller already launched this fresh worker for this exact
   assignment, execute here; **do not delegate the same skill to itself**.
   Otherwise dispatch first, including for a directly requested specialist.
   The standalone caller remains the coordinator without invoking `review-lens`.
3. Never combine multiple review skills in one worker or reuse a worker from a
   different skill, review pass or revision. Same-assignment clarifications may
   return to its worker. Another skill needs a fresh child or coordinator
   dispatch, never an inline invocation.
4. Isolation is mandatory even for small changes. Run independent workers in
   parallel when safe; run dependent or overlapping work sequentially in
   separate contexts. Fresh sessions share the filesystem, not a security
   sandbox: coordinate checkouts, serialize shared-worktree mutations and
   assign artifact ownership.
5. If fresh-worker execution is unavailable, report the affected stage as
   blocked. Do not silently run it in the caller's context or claim coverage.

## Minimal factual handoff

Pass the assigned skill/stage and result role, requested scope, repository and
revision/dirty-state identity, relevant trusted rules, configuration/toolchain,
execution permission, CI facts and owned artifact paths. Keep existing-comment
identities/anchors for deduplication; the coordinator retains their narratives.

Do not fork or paste the caller's conversation, previous reasoning, another
area's draft findings, or full logs into a new reviewer. Reuse matching factual
artifacts with provenance, not other reviewers' conclusions. Each skill's
evidence restrictions still apply: an output-only API worker gets no source,
diff or documentation evidence from a source-review coordinator.
For change reviews, the compact [package-presence record](package-comparison.md)
is permitted scope metadata. Retain its provenance without exposing source,
manifest content or raw documentation to an output-only worker.

Stage-specific inputs are deliberate exceptions, not shared conversation:

- The API filter receives the provisional report it must narrow or remove.
- Docs retrieval receives item paths/configuration, not candidate rationales;
  it returns a scoped bundle to the docs consumer, never raw JSON.
- Delivery receives the final merged findings, coverage, verdict, anchors and
  permitted mode, not the investigation history.

## Return, then deliver once

Area workers return findings and coverage to the coordinator using the
[findings contract](../review-delivery/findings-contract.md). Every finding
already includes attribution, a bold title, **Problem** and **Why this matters**
before handoff. Actionable findings also include **Suggested fix**; design notes
omit only that section. Retrieval returns data instead; filtering preserves
the shared format for surviving findings. Workers do not switch into another
review area or post findings independently.

The coordinator owns merging and the combined verdict, then starts **one fresh
`review-delivery` worker** with the selected PR/report-only mode. That is the
only worker authorized to deliver the combined review. Output-only API reports
and docs-only requests can return their results directly without a delivery
stage; their no-posting boundaries remain unchanged.
