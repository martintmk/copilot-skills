# Review Worker Isolation

Read before entering review sub-skills/stages. Every invocation needs a **fresh
agent session/context**, including standalone specialists, docs retrieval and
final delivery. Supporting documents are not extra skills; API filtering is an
explicitly isolated stage.

## Entry and dispatch

1. Before investigation/retrieval/delivery, the caller launches a fresh agent
   (for example, `task`) for exactly one skill/stage, scope and revision snapshot.
   If a trusted caller already launched this fresh worker for that exact
   assignment, execute here; **never dispatch yourself again**. A standalone
   caller coordinates without invoking `review-lens`.
2. No multi-skill workers or reuse across skills, passes or revisions.
   Same-assignment clarifications may return to their worker. Loading a skill
   inline, switching worktrees or compacting chat does not provide isolation.
   Another skill needs a fresh child or coordinator dispatch, never inline.
3. Even small changes require isolation. Parallelize independent work safely;
   sequence dependencies/overlaps in separate contexts. Sessions share a filesystem,
   not a sandbox: coordinate checkouts, serialize shared-worktree mutations and
   assign artifact ownership.
4. Unavailable fresh-worker execution means `blocked`, never an inline substitute
   or claimed coverage.

## Minimal factual handoff

Pass assignment/result role, scope, repository/revision/dirty-state identity,
trusted rules, configuration/toolchain, execution permission, CI facts and owned
artifact paths. Supply existing-comment IDs/anchors for deduplication; keep
narratives with the coordinator. Never fork conversations, prior reasoning,
other areas' drafts or full logs. Reuse only matching factual artifacts with
provenance, within each stage's evidence boundary.

For change reviews, the compact [package-presence record](package-comparison.md)
is permitted scope metadata, not source/manifest/doc content. Read that contract
when preparing or consuming a comparison.

Stage-specific exceptions, not shared conversations:

- Output-only API receives no source, manifests, source diffs, documentation
  evidence or other reviewers' findings.
- API filtering receives the provisional report to narrow/remove, not
  source-based reasoning.
- Docs retrieval receives paths/configuration, not candidate rationales, and
  returns a scoped bundle, never raw JSON.
- Delivery receives final merged findings, coverage, verdict, anchors and mode,
  not investigation history; a refresh also supplies reviewed/current snapshots
  and complete `findingRefresh`.

The coordinator's sole specialist-investigation exception is Lens's post-roster
[finding refresh](SKILL.md#best-effort-finding-refresh). Do not replay area
workers for it unless starting a fresh full review.

## Return, then deliver once

Read the [findings contract](../review-delivery/findings-contract.md) when
returning area findings/coverage or filtered findings. Retrieval returns data.
Workers never switch areas or post independently.

The coordinator merges/owns the combined verdict, then launches **one fresh
`review-delivery` worker** in the selected PR/report-only mode, the sole combined
delivery owner. Output-only API and docs-only requests may return directly
without delivery; they still never post.
