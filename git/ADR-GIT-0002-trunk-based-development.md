---
status: proposed
date: 2026-10-31
tags: [git, workflow, branching, ci]
---
# Trunk-Based Development

## Directive

All repositories must use trunk-based development. `main` is the only long-lived branch and must always be releasable. Work happens on short-lived branches that merge into `main` through a pull request within a day or two, using squash merge only; merged branches are deleted. `develop`, `release/*`, and other long-lived branches must not be used. Releases and deploys are cut from `main`, and unfinished work that would not be releasable must stay unreachable (behind a flag or not yet wired in) rather than held back on a branch.

## Context and Problem Statement

Long-lived branches (GitFlow's `develop` and `release/*`, or feature branches that live for weeks) drift away from `main`, so merging them becomes a large, risky event with conflicts nobody remembers the context for. They also hide integration problems until late and leave the question of "what is deployed" spread across several branches. Merging small changes into a single trunk often keeps conflicts small, keeps CI running against what will actually ship, and makes `main` the one answer to what is current.

## Decision Drivers

* Merges should be small and frequent, so conflicts stay small and easy to resolve
* CI should test the code that will actually ship, not a branch that will be merged later
* There should be one branch that answers "what is current"
* History on `main` should be one Conventional Commit per change (see ADR-GIT-0001)
* The workflow should suit small teams and AI agents producing many small changes

## Considered Options

* Trunk-based development with short-lived branches and squash merge
* GitFlow (`develop`, `release/*`, `hotfix/*`)
* Long-lived feature branches merged when the feature is complete

## Decision Outcome

Chosen option: "Trunk-based development with short-lived branches and squash merge", because it keeps every change small and integrated, keeps CI focused on what ships, and leaves `main` as the single source of truth.

### Repository settings

| Setting | Value |
|---|---|
| Default branch | `main` |
| Allowed merge methods | Squash only (merge commits and rebase merges disabled) |
| Squash commit message | Pull request title (checked against Conventional Commits in CI) |
| Delete branch on merge | Enabled |
| Branch protection on `main` | Require a pull request and passing status checks; no force pushes |

### Examples

A typical change:

```bash
git switch -c fix/session-cookie-name
# ...commit as often as useful; the branch's history is squashed away
git push -u origin fix/session-cookie-name
gh pr create --fill
# CI passes → squash merge → branch deleted
```

Shipping a feature over several pull requests without a long-lived branch: merge each piece into `main` while it is unreachable — the route not registered, the UI entry point not linked, or a flag left off — and make it reachable in the final pull request.

Applying the merge settings with the GitHub CLI:

```bash
gh repo edit --enable-squash-merge --enable-merge-commit=false \
  --enable-rebase-merge=false --delete-branch-on-merge
```

### Consequences

* Good, because merges stay small and conflicts are rare and easy to resolve
* Good, because CI always runs against the code that will ship
* Good, because `main` is the single answer to what is current and deployable
* Good, because squash merge gives `main` one Conventional Commit per pull request
* Bad, because large features must be split into pieces that are each safe to merge, sometimes behind a flag
* Bad, because squash merge discards the branch's individual commits from `main`'s history
* Bad, because a broken `main` blocks everyone, so CI and branch protection have to be in place

### Confirmation

Repository settings must match the table above. Code review must flag pull requests from long-lived branches, and branches other than `main` that outlive their pull request.

## Pros and Cons of the Options

### Trunk-based development

* Good, because small, frequent integration
* Good, because one branch to reason about
* Bad, because requires splitting large work and keeping `main` green

### GitFlow

* Good, because supports maintaining several released versions in parallel
* Bad, because long-lived `develop` and `release/*` branches drift and make merges large
* Bad, because "what is deployed" is spread across several branches

### Long-lived feature branches

* Good, because unfinished work never touches `main`
* Bad, because the branch drifts from `main` and the final merge is large and risky
* Bad, because CI does not test the combined code until the end

## More Information

* [Trunk Based Development](https://trunkbaseddevelopment.com/)
* Related: [ADR-GIT-0001 — Use Conventional Commits](ADR-GIT-0001-conventional-commits.md)
* Related: [ADR-AWS-0004 — Environments and Accounts](../aws/ADR-AWS-0004-environments-and-accounts.md)
