---
status: accepted
date: 2026-05-16
tags: [typescript, tooling, package-manager, monorepo]
---
# Use pnpm

## Directive

All TypeScript projects must use pnpm as the package manager. `package.json` must include a `"packageManager"` field pinned to a specific pnpm version. npm and Yarn must not be used.

## Context and Problem Statement

Node.js projects require a package manager to install and manage dependencies. The choice of package manager affects install speed, disk usage, monorepo support, and the reliability of dependency resolution. npm and Yarn have historically been the defaults, but pnpm has emerged as the superior option for performance and correctness.

## Decision Drivers

* Dependency installation must be fast, including in CI environments
* Disk usage should be minimised, particularly in monorepos with shared dependencies
* The package manager must support monorepo workspaces natively
* Phantom dependencies (importing packages not declared in `package.json`) must be prevented

## Considered Options

* pnpm
* npm
* Yarn (classic)
* Yarn Berry (v2+)
* Bun

## Decision Outcome

Chosen option: "pnpm", because it is the fastest option, uses a content-addressable store to minimise disk usage, and prevents phantom dependencies through strict module resolution.

### Examples

Install dependencies:

```bash
pnpm install
```

Add a dependency:

```bash
pnpm add zod
pnpm add -D typescript
```

Workspace `package.json` (monorepo root):

```json
{
  "private": true,
  "workspaces": ["packages/*"]
}
```

`pnpm-workspace.yaml`:

```yaml
packages:
  - 'packages/*'
```

Enforce pnpm as the only permitted package manager via `package.json`:

```json
{
  "packageManager": "pnpm@9.0.0"
}
```

### Consequences

* Good, because pnpm is significantly faster than npm and Yarn classic for installs
* Good, because a global content-addressable store deduplicates packages across projects, saving disk space
* Good, because strict module resolution prevents phantom dependency bugs
* Good, because native workspace support makes monorepo management straightforward
* Bad, because some tooling and CI templates assume npm and require minor adjustments
* Bad, because developers must have pnpm installed — it is not bundled with Node.js by default
* Neutral, because the `node_modules` structure differs from npm, which occasionally causes compatibility issues with poorly written packages

### Confirmation

`package.json` must include a `"packageManager"` field pinned to a specific pnpm version. A CI step runs `pnpm install --frozen-lockfile` to verify the lockfile is up to date.

## Pros and Cons of the Options

### pnpm

* Good, because fastest install times due to parallel downloads and hard-linking
* Good, because global content-addressable store minimises disk usage
* Good, because strict `node_modules` layout prevents phantom dependencies
* Good, because first-class workspace support for monorepos
* Bad, because not bundled with Node.js — requires separate installation
* Bad, because occasional compatibility issues with packages that assume a flat `node_modules`

### npm

* Good, because bundled with Node.js — zero additional setup
* Good, because universal compatibility across all packages and tooling
* Bad, because slowest of the three for installs
* Bad, because flat `node_modules` allows phantom dependencies
* Bad, because workspace support is rudimentary compared to pnpm

### Yarn (classic)

* Good, because fast installs and widespread adoption
* Good, because well-understood by most JavaScript developers
* Bad, because no longer actively developed — superseded by Yarn Berry
* Bad, because flat `node_modules` allows phantom dependencies
* Bad, because lockfile format is less portable than npm or pnpm

### Yarn Berry (v2+)

* Good, because Plug'n'Play eliminates `node_modules` entirely
* Good, because strong monorepo and constraint features
* Bad, because PnP breaks compatibility with many tools and editors without additional configuration
* Bad, because significantly more complex to set up and maintain
* Bad, because ecosystem adoption is low compared to pnpm

### Bun

* Good, because extremely fast installs — faster than pnpm in most benchmarks
* Good, because all-in-one runtime, bundler, and package manager
* Bad, because Node.js compatibility is not complete — some packages and native addons do not work
* Bad, because immature relative to pnpm — ecosystem tooling and CI support assumes Node.js
* Bad, because bundling the runtime with the package manager couples decisions that are better made independently
