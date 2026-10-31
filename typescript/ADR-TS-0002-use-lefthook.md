---
status: accepted
date: 2026-05-16
tags: [typescript, tooling, git, pre-commit]
---
# Use Lefthook

## Directive

All TypeScript projects must use Lefthook for Git hooks. A Lefthook config must be present at the project root, in any format Lefthook resolves natively (`lefthook.yml`, `.yaml`, `.json`, or `.toml`). At minimum it must:

* `pre-commit`: run `eslint --fix` and `prettier --write` on staged files, and type check the whole project (`pnpm typecheck`)
* `pre-push`: run `pnpm install --frozen-lockfile --offline`, then the tests, in that order (`piped: true`)
* `commit-msg`: enforce the commit message format (see ADR-GIT-0001)

## Context and Problem Statement

Linting and formatting must be enforced automatically before code enters version control, particularly for agent-written code which may not adhere to project standards without intervention. A pre-commit hook runner provides this guarantee consistently across all contributors.

## Decision Drivers

* Pre-commit hooks must run automatically without developer intervention
* Agent-written code must be validated before it enters version control
* Hook configuration should be version-controlled and consistent across all contributors
* A type change can break a file that was not edited, so type checking only staged files is not enough
* Commits must stay fast; slower checks (tests) belong on push, not on every commit
* A push must not carry a `package.json` change without the matching `pnpm-lock.yaml` change, which would fail CI's frozen install

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
    eslint:
      glob: "*.{ts,tsx,js,jsx}"
      run: pnpm exec eslint --fix {staged_files}
      stage_fixed: true
    prettier:
      glob: "*.{ts,tsx,js,jsx,json,md}"
      run: pnpm exec prettier --write {staged_files}
      stage_fixed: true
    typecheck:
      glob: "*.{ts,tsx}"
      run: pnpm typecheck

pre-push:
  piped: true
  jobs:
    - name: lockfile
      run: pnpm install --frozen-lockfile --offline
    - name: test
      run: pnpm test

commit-msg:
  commands:
    commitlint:
      run: pnpm exec commitlint --edit {1}
```

`stage_fixed: true` re-stages files auto-fixed by ESLint or Prettier before the commit completes. `typecheck` runs on the whole project (`tsc --noEmit` or `tsc -b`), but only when a TypeScript file is staged. `piped: true` makes the lockfile check run first and stop the push on a mismatch, and keeps the tests from racing the install. `--offline` keeps the check fast and avoids network access. Projects may add further pre-push checks (e.g. Spectral) after the lockfile step.

### Consequences

* Good, because linting and formatting are enforced automatically before every commit
* Good, because agent-written code is validated before entering version control
* Good, because hook configuration is version-controlled and consistent across all contributors
* Good, because Lefthook has no Node.js runtime dependency — works in any repo
* Good, because `stage_fixed: true` automatically re-stages files fixed by ESLint/Prettier
* Good, because type errors and lockfile mismatches are caught before CI
* Bad, because developers must install Lefthook locally for hooks to run
* Bad, because hooks can be bypassed with `git commit --no-verify`, so CI must still run the same checks
* Bad, because a whole-project type check and the test suite slow down commits and pushes in large projects

### Confirmation

Code review must flag a Lefthook config missing any of the required hooks above.

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

* Related: [ADR-TS-0001 — Use ESLint](ADR-TS-0001-use-linter.md)
* Related: [ADR-TS-0013 — Use Prettier](ADR-TS-0013-use-formatter.md)
* Related: [ADR-GIT-0001 — Use Conventional Commits](../git/ADR-GIT-0001-conventional-commits.md)
