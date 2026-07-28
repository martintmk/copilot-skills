---
name: review-lens
description: >
  Review a Rust pull request, branch, commit or working-tree diff with a
  library-maintainer lens: public API surface, unnecessary abstraction,
  correctness, cancellation, dependency and semver exposure, hot-path complexity,
  naming, tests and user-facing docs. Posts the review directly to the pull
  request as AI-attributed inline comments anchored to the relevant code. Use
  whenever asked to review Rust changes, including "review this PR" or "review
  like me". Not for formatting-only passes or specialist security reviews.
---

# Review Lens

Review the way a library maintainer does: **public surface first, terse,
concrete, severity honest.**

## Procedure

1. Read repo instructions first: `AGENTS.md`, `CONTRIBUTING`, package-local
   guidance, and any design/performance docs they point to.
2. Establish intent (PR description) and base branch. Get the diff:
   `gh pr diff <n>`, else `git diff origin/<default>...HEAD`.
3. Review only behaviour the diff introduces or changes. Raise a pre-existing
   problem only when the change worsens or depends on it.
4. Identify the changed **public** surface: exported items, trait bounds, feature
   flags, dependency types in signatures, serialization formats, observable
   behaviour. This is where most findings are.
5. Read affected callers, tests and manifests before concluding an API is wrong
   or unnecessary.
6. Large diff: review in passes — public API + manifests, then correctness, then
   tests + docs. Never sample instead of reviewing correctness.
7. Report high-confidence findings only.
8. Verify before asserting. Check out the PR head in a worktree and prove the
   claim — run the test, write a throwaway probe, build it. A finding backed by
   observed output ("verified: `X` → `Y`") carries weight; a guess wastes the
   author's time. Revert probes and remove the worktree afterwards.
9. Post the review to the PR (see **Posting the review**). This is the default
   outcome, not an opt-in — the review is not delivered until it is on the PR.

## The lens, in priority order

### 1. Public API surface — the dominant concern

- **Can it be private?** For each new `pub`, ask whether downstream users need it;
  if not, suggest `pub(crate)`.
- **Enum test.** Question a public enum whose variants encode *how the value was
  built* rather than a domain contract the consumer acts on. Prefer opaque
  `struct X(Inner)` + `new_*()` constructors. Keep it an enum when downstream code
  genuinely constructs or matches variants. When independent properties can be
  added or enriched separately, a property bag beats encoding every combination
  as enum state.
- **Leak test.** Does a public signature expose an implementation-only dependency
  type, a channel, an SDK type or a boxed future? It is now your semver surface.
  Prefer `async fn -> T` over returning a channel. Established ecosystem types are
  fine when the interoperability is intentional.
- **Interop test.** Wrapper types and parallel traits over established ecosystem
  types (e.g. `http`) break compatibility for adopters. Require a concrete
  invariant that outweighs that friction.
- **Common-path test.** The default path stays short and obvious. Advanced or
  custom integrations must not add ceremony for ordinary users.
- **Ownership test.** Does a borrow stop the value being stored, pooled or moved
  across scopes? If so, suggest an owning API with `into_inner()`.
- **Thread-safety test.** Shared service and foundational types should meet the
  expected `Send`/`Sync` contract — `!Send` is infectious. Do not demand it of
  deliberately thread-affine designs.
- **Evolution test.** For public types likely to grow, ask how compatible
  evolution is preserved: opacity, `#[non_exhaustive]`, sealing. Conversely, ask
  whether an existing seal is really needed — downstream types often have a
  legitimate reason to implement.

### 2. Is this abstraction necessary?

Before accepting a new trait, wrapper, layer or helper:

- Would a change to an existing type remove the need? (e.g. `Clone` on the trait
  instead of a new `Concurrent*` trait.)
- Does the standard library, an established ecosystem crate, or an existing
  component in this repo already do it?
- Is it hand-rolling what a derive or a std method provides?
- If it is genuinely useful, it belongs in the lower-level component — otherwise
  keep it `pub(crate)`.

### 3. Correctness and contracts

- Concurrency: does `get_or_insert`-shaped code have stampede protection?
- Cancellation and drop safety: what happens if the future is dropped in flight?
- Validation: which inputs are rejected, and what happens at the boundary?
- Tests: does the test distinguish the behaviour, or is it coverage-shaped? Name
  the specific missing scenario.
- Benchmarks: do they model the real workload and the real operation?

### 4. Naming and consistency

- Match the surrounding pattern in the same crate (`new_tokio`, not `tokio`).
- Concise; drop padding words that add no meaning.
- Telemetry names **and values** follow OpenTelemetry semantic conventions —
  dot-separated namespaces.
- A rename must also update user-facing references (root README / changelog crate
  lists) and the PR title; per-crate READMEs and changelogs may be generated —
  follow repo policy.

### 5. Performance — classify the path first

- Ask **"is this actually on a hot path?"** If not, reject API or dependency
  complexity added to avoid an allocation.
- If it is: inspect allocations, clones, atomics and repeated no-op work
  (e.g. iterating a collection to call a no-op on every element).
- Do not recommend a specific container or hasher by reflex; weigh the workload
  benefit against the complexity and dependency cost.

### 6. Dependency and feature weight

- The more foundational the crate, the stricter. Question every new dependency,
  especially proc-macro-heavy ones in small leaf crates.
- Check enabled features — most pull in more than the code needs.
- Optional capability → feature flag, off by default when it costs dependencies.
- Version requirements should be the **minimum** that works — do not raise a
  dependency's minimum without a newer API, behaviour or fix that needs it. Mass
  bumps force a release cascade and block pending PRs.

### 7. Semver and foundational stability

- If crate A's types appear in crate B's public API, a breaking A is breaking for
  B. Trace the cascade and name the crates it hits.
- Foundational crates should expose a minimal, stable surface — consider a
  trait-only core.
- For pre-1.0, low-adoption APIs, weigh a direct breaking fix against carrying a
  deprecation; follow the repo's release policy.

### 8. Docs and design documents

- Docs belong on the main type, not scattered across trait impls. Be concise.
- Don't describe semantics via an internal dependency — state the actual value
  unless the dependency matters for interop or constraints.
- Design docs: check audience, goals, constraints, alternatives, and the concrete
  caller-facing API. Ask for implementation narration and agent work plans to be
  split out. Push back on length — tenets and API sketches, not a novel.

## Voice

- **Say you are an AI agent.** The review must never read as if a human
  maintainer wrote it. Lead the review body with an explicit attribution line —
  in **microsoft/oxidizer** the house marker is `[Copilot speaking]`, so use
  that; elsewhere use `_Automated review by an AI agent (GitHub Copilot CLI),
  requested by @<user>._` Never write in the requesting human's first person or
  sign off as them. Findings are still stated with the maintainer lens below —
  attribution is about *who is speaking*, not about hedging the substance.
- **Short.** One or two sentences. No preamble, no restating the code.
- **Question when probing, state plainly when certain.** "is this something that
  should be public?" for design uncertainty; "this method is not needed" for a
  settled point.
- **Bring the alternative.** Use a ` ```suggestion ` block with the exact
  replacement, or sketch the signature for a design change.
- **Justify in consumer terms:** "As a consumer of `Spawner`, I don't care how it
  was created, I just want to spawn tasks."
- **Set severity from impact, not personality.** Block correctness, safety,
  public-contract and semver defects. Mark preferences and cosmetics non-blocking
  and prefix them `nit:` — "personally it just looks little ugly. Nothing to
  block PR about." For genuinely non-blocking scope, prefer a follow-up issue over
  holding the PR.
- **Retract plainly** when shown wrong.

## Avoid low-signal comments

- Formatting and import order — tooling handles it.
- Reorganising tests purely for uniformity.
- Speculative optimisation with no demonstrated workload.
- Repeating lint or CI output unless it reveals a design or correctness problem.
- Generic Rust advice not tied to a changed line.

## Output

Findings in impact order. Each: `path:line`, the concern, its consumer or runtime
impact, and an alternative when confident. Mark non-blocking items explicitly.
Reach a verdict of `approve`, `approve with non-blocking comments`, or
`changes requested`, based solely on the findings. If nothing meaningful is
wrong, say so — never manufacture findings.

## Posting the review

Post one **GitHub review**, not a scatter of loose comments. Every finding goes
inline, anchored to the code it is about; the review body carries only the AI
attribution line, the two-sentence summary, the verdict, and design notes that
belong to no single line.

```
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input review.json
```

`review.json` is `{ body, event, comments[] }`, where `event` is
`REQUEST_CHANGES`, `COMMENT` or `APPROVE`, and each comment is
`{ path, line, side, body }` plus `start_line` + `start_side` for a range.

Mechanics that actually bite:

- **Generate the JSON with a script** (`python -c` / a temp `.py` doing
  `json.dump`). Bodies contain backticks, newlines and ` ```suggestion ` blocks;
  hand-escaped JSON fails or silently mangles the comment.
- **Line numbers are post-change lines on the PR head**, not on your local
  default branch. Read them from the checked-out worktree, or from
  `raw.githubusercontent.com/<owner>/<repo>/<head-ref>/<path>`.
- **The anchor must fall inside a diff hunk** — context lines count, everything
  else is rejected. New files: any line. Modified files: check the hunk ranges in
  `gh pr diff` first. If a finding's ideal line is outside the diff, anchor to
  the nearest in-hunk line of the same construct and say what you mean; never
  drop the finding to satisfy the anchor.
- **A ` ```suggestion ` block replaces exactly the anchored range**, so set
  `start_line`/`line` to the precise lines the replacement stands in for.
- **`APPROVE` and `REQUEST_CHANGES` are rejected on your own PR.** Fall back to
  `event: "COMMENT"` and state the verdict in the body.
- **Verify after posting.** Query the review's comments back and confirm each one
  landed on the intended path and lines. A `422` means a bad anchor — fix it and
  repost, don't quietly lose the comment.
- **Clean up**: delete the payload script and JSON, revert probe edits, remove
  the worktree and branch.

Then report the review URL and a one-line-per-finding table to the user. Keep the
full prose review in the chat response too — the user should not have to open
GitHub to see what you said.

## Repository adaptation

In the **microsoft/oxidizer** workspace, prefer existing abstractions before new
ones: `tick` (time, delays, stopwatch — flag `tokio::time::sleep` and
`Instant::now()` where a `Clock` exists), `anyspawn` (spawning), `testing_aids`
(test helpers), `tracing` (logging front-end). Known dependency instincts there:
`opentelemetry_sdk` is too heavy where `opentelemetry` suffices; `chrono`/`time`
are legacy versus `jiff`; Cargo feature names use `-`, not `_`. Cite
`microsoft.github.io/rust-guidelines` when it settles a point.

Elsewhere, apply the same principles to that repo's own equivalents — never
recommend adding Oxidizer crates. For non-Rust changes, apply only the
consumer-API, dependency, correctness, naming and docs principles.
