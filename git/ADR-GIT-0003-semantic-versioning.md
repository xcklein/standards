---
status: proposed
date: 2026-10-31
tags: [git, versioning, releases, semver]
---
# Semantic Versioning

## Directive

All versions must follow Semantic Versioning 2.0.0 and are recorded as git tags in the form `vX.Y.Z`. Development starts at `0.1.0`. While the major version is `0`, no change of any kind or size increments the major version: breaking changes and features increment the minor version, and everything else increments the patch version. The version is set to `1.0.0` by hand when the project is ready for a true release; from then on, standard semver applies.

## Context and Problem Statement

Semantic Versioning reserves `0.y.z` for initial development, when anything may change and the public API is not yet stable. Starting at `1.0.0` promises a stable API before one exists. Letting a tool bump the major version on the first breaking change has the opposite problem: the project reaches `1.0.0`, `2.0.0`, and beyond during early development, and the version that should mean "ready for release" arrives by accident. Keeping everything under `0.x` until a deliberate `1.0.0` makes the major version mean something.

## Decision Drivers

* A version number must tell readers whether the project has been released, and whether an upgrade can break them
* Early development is full of breaking changes that should not inflate the major version
* Reaching `1.0.0` should be a deliberate decision, not a side effect of a commit message
* Version bumps should be derived automatically from Conventional Commits (see ADR-GIT-0001) everywhere except the move to `1.0.0`

## Considered Options

* Semver, starting at `0.1.0`, with a manual `1.0.0`
* Semver, starting at `1.0.0`
* Calendar versioning (e.g. `2026.10.31`)
* Plain build numbers

## Decision Outcome

Chosen option: "Semver, starting at `0.1.0`, with a manual `1.0.0`", because it follows the semver specification's own convention for initial development, keeps the major version meaningful, and makes the first real release an explicit decision.

### Bump rules

| Change (Conventional Commit) | While `0.x` | From `1.0.0` |
|---|---|---|
| Breaking (`!` or `BREAKING CHANGE:`) | minor | major |
| `feat` | minor | minor |
| Anything else | patch | patch |

The first tag in a repository is `v0.1.0`. CI applies these rules to every merge to `main` (see ADR-GIT-0002).

### Examples

During development:

| Merge | Version |
|---|---|
| First tag | `v0.1.0` |
| `fix: handle empty cart` | `v0.1.1` |
| `feat: add wishlists` | `v0.2.0` |
| `feat!: replace session cookies with tokens` | `v0.3.0` (not `v1.0.0`) |

Releasing `1.0.0`, by hand, on the commit being released:

```bash
git tag v1.0.0 <commit>
git push origin v1.0.0
```

The `v1.0.0` tag is then promoted through the environments like any other tag. After it, a `feat!` merge produces `v2.0.0`.

### Consequences

* Good, because the major version stays at `0` until a deliberate release
* Good, because after `1.0.0` the version tells consumers exactly when an upgrade can break them
* Good, because every version except `1.0.0` is derived automatically from commit messages
* Bad, because during `0.x` a minor bump may be breaking, so consumers must read the changes before upgrading
* Bad, because someone must remember to tag `1.0.0` when the project is released

### Confirmation

The tagging workflow must never produce a major bump while the latest version is `0.x`. Code review must flag a `1.0.0` or later tag created by automation rather than by hand, and versions that are not `vX.Y.Z`.

## Pros and Cons of the Options

### Semver, starting at `0.1.0`, with a manual `1.0.0`

* Good, because matches the semver specification's initial-development convention
* Good, because `1.0.0` marks a real release
* Bad, because breaking changes during `0.x` look like minor bumps

### Semver, starting at `1.0.0`

* Good, because standard bump rules apply from day one
* Bad, because it claims a stable API before one exists
* Bad, because early breaking changes inflate the major version quickly

### Calendar versioning

* Good, because the version shows when it was released
* Bad, because it says nothing about whether an upgrade is breaking

### Plain build numbers

* Good, because simple and always increasing
* Bad, because it says nothing about the size or impact of a change

## More Information

* [Semantic Versioning 2.0.0](https://semver.org/)
* Related: [ADR-GIT-0001 — Use Conventional Commits](ADR-GIT-0001-conventional-commits.md)
* Related: [ADR-GIT-0002 — Trunk-Based Development](ADR-GIT-0002-trunk-based-development.md)
