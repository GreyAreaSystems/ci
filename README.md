# ci

Shared reusable GitHub Actions workflows for the Grey Area repositories.

## Why this repo is public

Cross-organisation reuse requires it. A reusable workflow in a **private** repo
can only be called by repos in the **same** organisation; ours span four
accounts (`GreyAreaSystems`, `KyleEdwardDonaldson`, `quetzalgames`,
`ked-development`), so the shared copy has to be public or it has to be
duplicated four times.

Nothing secret lives here. These files describe *build shape* — crate names,
step order, cache strategy. Secrets stay in each calling repo's own settings and
are never referenced from here.

## Workflows

| Workflow | For |
|---|---|
| `rust-check.yml` | fmt → clippy → test, with `Swatinem/rust-cache`. Optional `cargo package`. |
| `node-check.yml` | typecheck → lint → test → build, with npm caching. |

## Use

```yaml
# .github/workflows/ci.yml in a calling repo
name: CI
on:
  push:
    branches: ["**"]
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  check:
    uses: GreyAreaSystems/ci/.github/workflows/rust-check.yml@main
    with:
      exclude: "myco-desktop myco-ui myco-ui-kit"
```

## Two defaults that are deliberate

**`CARGO_TARGET_DIR` defaults to `target`.** Some repos point `target-dir` at a
path outside the workspace in `.cargo/config.toml` (handy locally, on a fast
scratch disk). `rust-cache` keys off the workspace, so without this override
those repos would rebuild cold on every run and the cache would never hit.

**No `RUSTFLAGS` is set.** As an environment variable it *replaces* any
`target.<triple>.rustflags` block in a repo's `.cargo/config.toml` rather than
extending it — which silently drops flags a GPU or GUI link depends on, such as
`-C link-arg=/STACK:8000000`. Warnings are denied via `clippy -- -D warnings`
instead, which composes correctly.

## Cost notes

- Public repos get **unlimited free** GitHub-hosted minutes. Private repos draw
  on their account's allowance (2,000/month on Free plans).
- Linux is ~60% the per-minute rate of Windows and ~10% of macOS. Run a job on
  Linux unless it produces a platform-specific artifact.
- Since **1 March 2026** self-hosted runners also consume included minutes and
  add a per-minute platform charge, so self-hosting is bought for wall-clock and
  core count, not to avoid the bill.
