---
status: accepted
date: 2026-05-16
tags: [typescript, tooling, git, pre-commit]
---
# Use Lefthook

## Directive

All TypeScript projects must use Lefthook for Git hooks. A `lefthook.yml` must be present at the project root. At minimum it must run `biome check --write` on staged files via a `pre-commit` hook and enforce commit message format via a `commit-msg` hook (see ADR-GIT-0001).

## Context and Problem Statement

Linting and formatting must be enforced automatically before code enters version control, particularly for agent-written code which may not adhere to project standards without intervention. A pre-commit hook runner provides this guarantee consistently across all contributors.

## Decision Drivers

* Pre-commit hooks must run automatically without developer intervention
* Agent-written code must be validated before it enters version control
* Hook configuration should be version-controlled and consistent across all contributors

## Considered Options

* Lefthook
* Husky

## Decision Outcome

Chosen option: "Lefthook", because it is lightweight, fast, and requires no Node.js dependency to run, making it suitable for any repo regardless of runtime.

### Examples

Add a `lefthook.yml` at the repository root:

```yaml
pre-commit:
  commands:
    biome:
      glob: "*.{ts,tsx,js,jsx}"
      run: npx @biomejs/biome check --write {staged_files}
      stage_fixed: true

commit-msg:
  commands:
    commitlint:
      run: pnpm commitlint --edit {1}
```

`stage_fixed: true` ensures files auto-fixed by Biome are re-staged automatically before the commit completes. The `commit-msg` hook enforces the Conventional Commits format on every commit (see ADR-GIT-0001).

### Consequences

* Good, because linting and formatting are enforced automatically before every commit
* Good, because agent-written code is validated before entering version control
* Good, because hook configuration is version-controlled and consistent across all contributors
* Good, because Lefthook has no Node.js runtime dependency — works in any repo
* Good, because `stage_fixed: true` automatically re-stages files fixed by Biome
* Bad, because developers must install Lefthook locally for hooks to run
* Bad, because hooks can be bypassed with `git commit --no-verify`

## Pros and Cons of the Options

### Lefthook

* Good, because lightweight with no Node.js runtime dependency
* Good, because configuration is a single version-controlled YAML file
* Good, because fast execution — runs hooks in parallel by default
* Good, because `stage_fixed: true` automatically re-stages auto-fixed files
* Bad, because requires local installation per developer
* Bad, because hooks can be bypassed with `--no-verify`

### Husky

* Good, because widely adopted with extensive community documentation
* Good, because easy to set up via npm scripts
* Bad, because requires Node.js and npm to function — not suitable for non-JS repos
* Bad, because heavier configuration overhead compared to Lefthook
* Bad, because slower than Lefthook

## More Information

* Related: [ADR-TS-0001 — Use Biome](ADR-TS-0001-use-biome.md)
* Related: [ADR-GIT-0001 — Use Conventional Commits](../git/ADR-GIT-0001-conventional-commits.md)
