---
name: dependency-cooldown-skill
description: Adds dependency cooldowns to projects to mitigate the risk of supply chain attacks.
---

# dependency-cooldown-skill

## Overview

A **dependency cooldown** delays installing a newly-published package version (typically 7 days). The window gives security vendors time to detect and report supply chain attacks before the malicious version reaches builds. Recent data: 8 of 10 supply chain attacks had a window of opportunity under 7 days, so a 7-day cooldown blocks the majority. Cooldowns complement — they don't replace — pinning, lockfiles, and trusted publishing.

## When to use

- User asks to add, configure, set up, or audit dependency cooldowns
- User asks about supply chain security mitigations for dependencies

**Do NOT use when:**
- The project already has appropriate cooldowns for every ecosystem in use — verify first and report current state
- User wants reproducible pinning to a specific date — that's a different feature (absolute timestamps, not a rolling cooldown)
- Security updates need to be delayed — they should *bypass* the cooldown (most tools do this by default)

## Instructions

### Step 1: Survey the project

Before changing anything, identify:

1. **Package managers in use** — lockfiles and manifests (`pyproject.toml`/`uv.lock`, `package.json`/`pnpm-lock.yaml`/`yarn.lock`/`bun.lockb`, `Cargo.toml`, `Gemfile`, `go.mod`, `.github/workflows/*.yml`, etc.)
2. **Existing update bots** — `.github/dependabot.yml` (Dependabot) or `renovate.json`/`.renovaterc` (Renovate)
3. **Existing cooldown config** — grep for: `cooldown`, `minimumReleaseAge`, `min-release-age`, `npmMinimalAgeGate`, `exclude-newer`, `uploaded-prior-to`, `stabilityDays`, `minimum-dependency-age`

Report findings to the user. If cooldowns are already set up for every ecosystem in use, stop.

### Step 2: Choose the layer(s)

Two layers — apply both for defense in depth:

- **Package manager (primary)** — protects every install: manual CLI runs, CI, and transitive deps. Native support in uv, pip, pnpm, npm, yarn, bun, deno, and (per-bump) cargo.
- **Update bot (secondary)** — protects automated PRs. Applies when the project uses or wants Dependabot/Renovate.

**Default:** start with the package-manager layer. Add a bot layer if the project already uses one or the user wants automated PRs gated too.

### Step 3: Configure the package-manager layer

**Recommended cooldown: 7 days** (14 days for higher-risk projects — prod infra, payments, auth).

For each package manager in the project, **look up the current syntax** — feature names and config locations are young and have shifted across versions. Approach, in order:

1. **Run `<tool> --help`** to find the relevant flag (e.g. `uv add --help`, `pip install --help`, `pnpm install --help`, `deno update --help`).
2. **Fetch the official docs** if `--help` is insufficient. Confirm the minimum version required.
3. **Persist the config** to the project's config file (`pyproject.toml`, `.npmrc`, `.yarnrc.yml`, `bunfig.toml`, etc.) — not just as a one-off CLI flag — so every install respects it.

The feature has different names per tool: `exclude-newer` (uv), `--uploaded-prior-to` (pip), `minimumReleaseAge` (pnpm/bun), `min-release-age` (npm), `npmMinimalAgeGate` (yarn), `--minimum-dependency-age` (deno). pnpm/npm/yarn/bun take values in **minutes** (7 days = 10080); the rest take human-readable strings. Verify against current docs before writing.

For ecosystems without native cooldown support (Ruby/Bundler, Go modules, Composer, Maven, Gradle, Swift PM, Hex, Dart pub), skip to the bot layer.

### Step 4: Configure the update-bot layer (when used)

Security updates bypass cooldowns automatically in both Dependabot and similar tools — keep that default.

#### Dependabot (`.github/dependabot.yml`)

Add one `updates:` entry per ecosystem:

```yaml
version: 2
updates:
  - package-ecosystem: pip           # change per ecosystem
    directory: /
    schedule:
      interval: weekly
    cooldown:
      default-days: 7
      # Optional per-semver-level overrides:
      # semver-major-days: 14
      # semver-minor-days: 7
      # semver-patch-days: 3
```

Valid `package-ecosystem` values include `npm`, `pip`, `bundler`, `cargo`, `gomod`, `nuget`, `composer`, `docker`, `github-actions`, `gradle`, `maven`, `mix`, `pub`, `swift`, `terraform`. Consider adding `github-actions` if the repo has workflows — it's a real attack target (`tj-actions`, `nx`).

### Step 5: Verify

1. Show the diff and the chosen cooldown duration
2. Validate the config — for Dependabot, run `zizmor .github/dependabot.yml` (its `dependabot-cooldown` rule also flags missing cooldowns); for Renovate, run its config validator; for package managers, run the tool's normal install/check
3. Note that cooldowns take effect on the *next* dependency operation — existing lockfile entries aren't retroactively delayed

## Attribution

- [We should all be using dependency cooldowns](https://blog.yossarian.net/2025/11/21/We-should-all-be-using-dependency-cooldowns)
- [Package Managers Need to Cool Down](https://nesbitt.io/2026/03/04/package-managers-need-to-cool-down.html)
- [We should all be using dependency cooldowns - Simon Willison](https://simonwillison.net/2025/Nov/21/dependency-cooldowns/)
