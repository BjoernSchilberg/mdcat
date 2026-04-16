---
name: pre-commit-checks
description: Run all CI checks locally before committing and pushing to GitHub
license: MIT
compatibility: opencode
---

## What I do

Run all CI checks that also run in GitHub Actions, fix any errors found, and
only commit and push when everything passes cleanly.

## Steps

Run the following checks in order. Stop immediately when a check fails and fix
the error before continuing.

### 1. Format

```
cargo fmt --all -- --check
```

If this fails, run `cargo fmt --all` to fix formatting, then re-check.

### 2. Clippy (with default features)

```
cargo clippy --workspace --all-targets --locked
```

### 3. Clippy (without default features)

```
cargo clippy -p pulldown-cmark-mdcat --all-targets --locked --no-default-features
```

Fix all `error` diagnostics. Clippy errors are denied via `#![deny(warnings)]`
and will break the build. Do not add `#[allow(...)]` unless there is no correct
fix available — prefer fixing the code.

### 4. Tests

```
cargo test --workspace --locked
```

### 5. Documentation

```
cargo doc -p pulldown-cmark-mdcat --locked --no-default-features
cargo doc -p mdcat --locked
```

Intra-doc links must resolve. Crate names with hyphens (e.g.
`pulldown-cmark-mdcat`) cannot be used as Rust intra-doc links — use plain
backtick text instead.

### 6. cargo deny

```
cargo deny check
```

If an advisory has no safe upgrade available and is a transitive dependency,
add it to the `ignore` list in `deny.toml` with a comment explaining why.

### 7. cargo vet

```
cargo vet --locked
```

If this fails with a formatting error, run `cargo vet fmt` first, then
re-run `cargo vet --locked`.

If new dependency versions are unvetted, add exemptions to
`supply-chain/config.toml` for each unvetted package with
`criteria = "safe-to-run"`, then run `cargo vet fmt` to normalize the file,
and verify with `cargo vet --locked` again.

**Version sync:** The locally installed `cargo-vet` version must match the
`CARGO_VET_VERSION` pinned in `.github/workflows/test.yml`. Different versions
format `imports.lock` differently (e.g. quote escaping in TOML strings), which
causes consistency errors in CI. Check the versions with `cargo vet --version`
and keep them in sync.

## When to use me

Use this skill before every `git commit` and `git push`. All checks must pass
locally before pushing to avoid failing GitHub Actions runs.
