---
name: minimal-local-validation
description: >
  Keep Rust implementation validation narrow when comprehensive CI covers the
  workspace. Batch edits, check affected packages once, and continue queued
  work without waiting for CI. Not for explicit full-validation requests or
  repositories without comprehensive CI.
---

# Minimal Local Validation

Use fast local feedback instead of repeating comprehensive CI.
If that CI coverage is absent or uncertain, state the limit and use the smallest
relevant repository check. Do not claim full validation.

After all edits, run:

```text
cargo check -p <changed-package>
```

Combine affected packages with repeated `-p`. Use the required toolchain.
Add existing feature flags only when the change needs them.
Do not run workspace suites, broad feature matrices or unrelated checks unless
explicitly requested or necessary to validate the change.
Do not install tools or clear build artifacts merely to widen validation.

After the narrow check passes, rely on CI for exhaustive coverage.
Continue queued work instead of starting a blocking watcher or polling checks.
Report CI as pending until its result is known.
This does not waive CI-dependent merge or release gates.

For a concrete pipeline failure, inspect its cause and reproduce it with the
smallest relevant command. Fix it, then rerun that command and the affected
package check. Targeted diagnostic reproductions can precede the final check.

This skill limits implementation validation. It does not replace specifically
requested review evidence or benchmark measurements. Those tasks still inherit
the caller's execution limits.
