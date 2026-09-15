---
name: minimal-local-validation
description: >
  Keep local Rust validation intentionally narrow when the build pipeline
  already performs comprehensive workspace checks. Use when asked to minimize
  local validation, avoid long workspace-wide checks, run cargo check only for
  changed crates, or rely on CI for exhaustive validation and fix only concrete
  pipeline failures. Not for repositories without comprehensive CI, explicit
  full-validation requests, or diagnosing an already failing pipeline.
---

# Minimal Local Validation

Prefer fast local feedback over duplicating comprehensive CI. Make the requested
change, identify the directly changed Rust crate or crates, and run only:

```text
cargo check -p <changed-package>
```

Combine multiple changed packages in one command with repeated `-p`. Use the
repository's required toolchain and existing feature flags only when the changed
code needs them.

Do not run workspace-wide builds, tests, linting, formatting checks, broad
feature matrices, or unrelated package validation unless the user explicitly
requests them or the narrow check cannot validate the change. Do not add tools,
install dependencies, or clean build artifacts merely to expand validation.

After the narrow check passes, stop local validation and rely on the existing
pipeline for exhaustive coverage. If CI reports a failure, inspect that concrete
failure, reproduce it with the smallest relevant local command when practical,
fix its root cause, and rerun only that command plus `cargo check` for affected
crates. Do not respond to one pipeline failure by running every workspace check.

This policy applies only when comprehensive CI is expected to run for the
change. If that premise is false or uncertain, state the limitation rather than
claiming the change is fully validated.
