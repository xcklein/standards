---
status: proposed
date: 2026-10-31
tags: [git, workflow, branching, ci]
---
# Trunk-Based Development

## Directive

All repositories must use trunk-based development. `main` is the only long-lived branch and must always be releasable. Work happens on short-lived branches that merge into `main` through a pull request, using squash merge only; merged branches are deleted. `develop`, `release/*`, and other long-lived branches must not be used. Unfinished work that would not be releasable must stay unreachable (behind a flag or not yet wired in) rather than held back on a branch. Every merge to `main` is tagged by CI with the next version, following ADR-GIT-0003; tags are never moved or deleted. Deploys use tags, not branch heads, and a tag is promoted through the environments in a fixed order (e.g. `dev` → `prod`): it may deploy to an environment only after it deployed successfully to the one before.

## Context and Problem Statement

Long-lived branches (GitFlow's `develop` and `release/*`, or feature branches that live for weeks) drift away from `main`, so merging them becomes a large, risky event with conflicts nobody remembers the context for. They also hide integration problems until late and leave the question of "what is deployed" spread across several branches. Merging small changes into a single trunk often keeps conflicts small, keeps CI running against what will actually ship, and makes `main` the one answer to what is current.

## Decision Drivers

* Merges should be small and frequent, so conflicts stay small and easy to resolve
* CI should test the code that will actually ship, not a branch that will be merged later
* There should be one branch that answers "what is current"
* History on `main` should be one Conventional Commit per change (see ADR-GIT-0001)
* The workflow should suit small teams and AI agents producing many small changes
* Every environment should run an identifiable, immutable version, and production should only ever run a version that already worked in the environments before it

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

Tagging every merge to `main` — the bump follows the merged commit's type, using the rules in ADR-GIT-0003. Squash merge means each push to `main` is exactly one commit:

```yaml
name: tag

on:
  push:
    branches: [main]

permissions:
  contents: write
  actions: write

jobs:
  tag:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0
      - id: version
        name: Tag the next version
        run: |
          last=$(git tag --list 'v*' --merged HEAD --sort=-v:refname | head -n 1)
          if [ -z "$last" ]; then
            next=0.1.0
          else
            IFS=. read -r major minor patch <<< "${last#v}"
            subject=$(git log -1 --format=%s)
            if echo "$subject" | grep -qE '^[a-z]+(\(.+\))?!:' \
              || git log -1 --format=%b | grep -q '^BREAKING CHANGE:'; then
              bump=major
            elif echo "$subject" | grep -qE '^feat(\(.+\))?:'; then
              bump=minor
            else
              bump=patch
            fi
            # Nothing leaves 0.x automatically; 1.0.0 is tagged by hand.
            if [ "$major" = 0 ] && [ "$bump" = major ]; then bump=minor; fi
            case $bump in
              major) next="$((major + 1)).0.0" ;;
              minor) next="$major.$((minor + 1)).0" ;;
              patch) next="$major.$minor.$((patch + 1))" ;;
            esac
          fi
          git tag "v$next"
          git push origin "v$next"
          echo "tag=v$next" >> "$GITHUB_OUTPUT"
      # A tag pushed with GITHUB_TOKEN triggers no workflows, but a dispatch does.
      - name: Deploy to the first environment
        env:
          GH_TOKEN: ${{ github.token }}
          TAG: ${{ steps.version.outputs.tag }}
        run: gh workflow run deploy.yml --ref "$TAG" -f env=dev
```

Promotion — the new tag deploys to the first environment automatically, then is dispatched to each next environment in order. The deploy workflow runs on the tag itself, so the deployed commit is exactly the tagged one:

```bash
gh workflow run deploy.yml --ref v1.4.0 -f env=prod   # only after dev succeeded
```

The deploy workflow enforces the order; see ADR-AWS-0004 for the check. A rollback is a deploy of an earlier tag, promoted the same way.

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
* Good, because every environment runs a named, immutable version, and production only runs versions that already passed the earlier environments
* Bad, because large features must be split into pieces that are each safe to merge, sometimes behind a flag
* Bad, because squash merge discards the branch's individual commits from `main`'s history
* Bad, because a broken `main` blocks everyone, so CI and branch protection have to be in place
* Bad, because every change takes a deploy per environment to reach production

### Confirmation

Repository settings must match the table above, and a workflow must tag every push to `main`. Code review must flag pull requests from long-lived branches, branches other than `main` that outlive their pull request, and deploy workflows that deploy a branch head or skip the promotion order.

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
* Related: [ADR-GIT-0003 — Semantic Versioning](ADR-GIT-0003-semantic-versioning.md)
* Related: [ADR-AWS-0004 — Environments and Accounts](../aws/ADR-AWS-0004-environments-and-accounts.md)
