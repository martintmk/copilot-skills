# Child session handoff

Use this prompt for the child that owns one crate. Replace every placeholder
before launch.

```text
Work only on crate <crate> at <manifest path> in <repository>.

Apply this user instruction:

<verbatim instruction>

This is one item in a sequential crate rollout. Make a complete change for this
crate. Change shared workspace files only when this crate directly requires it.
Do not change another crate to prepare future queue items.

Inspect the repository rules and the crate before editing. Implement the change,
update directly related tests and documentation, and run the smallest validation
that proves this crate. Run affected-crate cargo check and cargo test once after
the edits when repository rules permit them.

Commit and push the change. Create one ready-for-review pull request against
<default branch>. Do not create a draft or stacked pull request.

Before writing the pull request description, invoke pr-description. Then invoke
simple-english on the final text. Keep the Summary, Evidence, and Merge Danger
sections concise. The Summary must contain this exact sentence:

This is an automated mechanical PR for `<crate>`.

State the applied change, actual validation, and real merge risk. Do not claim
that behavior is unchanged unless the evidence proves it.

After every push on a branch that has a pull request, invoke track-pr. Fix
change-caused CI failures in this same session. Apply feedback-autonomy before
you act on pull request feedback. Do not merge or approve the pull request.

Report the pull request URL and number, current head commit, validation results,
and any blocker to the coordinator. Stay available until the pull request
merges and the coordinator archives this session.
```

