---
status: accepted
date: 2026-05-16
tags: [git, commits, changelog]
---
# Use Conventional Commits

## Directive

All commits must follow the Conventional Commits specification. The format is `<type>[optional scope]: <description>`. Breaking changes must be indicated with `!` after the type or a `BREAKING CHANGE:` footer.

## Context and Problem Statement

Without a commit message convention, history becomes a mix of styles that is difficult to scan, search, or automate against. A consistent format makes it immediately clear what kind of change a commit represents, enables automated changelog generation, and provides the input needed for semantic version bumping tools.

## Decision Drivers

* Commit history must be scannable — the type and intent of each change must be clear at a glance
* Breaking changes must be unambiguously identifiable in the commit log
* The format must be parseable by tooling for changelog generation and semantic versioning
* The convention must be well-documented and widely understood

## Considered Options

* Conventional Commits
* No convention

## Decision Outcome

Chosen option: "Conventional Commits", because it is a lightweight, widely adopted specification that is tooling-friendly, human-readable, and covers the full range of change types including breaking changes.

### Examples

Standard commits:

```
feat: add cursor-based pagination to users endpoint
fix: correct JWT expiry validation on refresh
docs: update README with pnpm setup instructions
chore: bump vitest to 2.1.0
ci: add Spectral OpenAPI lint step to pipeline
refactor: extract config validation into dedicated module
```

Scoped commit:

```
feat(auth): add RS256 token verification
```

Breaking change — using `!`:

```
feat!: remove offset pagination support from all endpoints
```

Breaking change — using footer:

```
feat: redesign error response format

BREAKING CHANGE: error responses now use application/problem+json and require type, title, status, and detail fields
```

### Consequences

* Good, because commit history is immediately scannable by type without reading full messages
* Good, because breaking changes are unambiguously marked and easy to grep
* Good, because enables automated changelog generation and semantic version bumping
* Good, because the specification is publicly documented — new contributors can be pointed to it
* Bad, because requires discipline — tooling (see Confirmation) is needed to enforce it reliably
* Bad, because short descriptions must still be meaningful — the format alone does not guarantee useful messages

### Confirmation

A Lefthook `commit-msg` hook using `commitlint` enforces the format on every commit:

```yaml
# lefthook.yml
commit-msg:
  commands:
    commitlint:
      run: pnpm commitlint --edit {1}
```

```js
// commitlint.config.js
export default { extends: ["@commitlint/config-conventional"] };
```

## Pros and Cons of the Options

### Conventional Commits

* Good, because lightweight — the core specification fits on one page
* Good, because machine-readable — type and breaking change markers are unambiguous
* Good, because widely adopted — most changelog and release tooling supports it natively
* Good, because human-readable — type prefix makes scanning history fast
* Bad, because requires tooling or discipline to enforce consistently

### No convention

* Good, because no rules to learn or enforce
* Bad, because history becomes inconsistent and difficult to scan
* Bad, because breaking changes have no standard marker
* Bad, because automated changelog generation is not possible

## More Information

* [Conventional Commits specification](https://www.conventionalcommits.org)
* Related: [ADR-TS-0002 — Use Lefthook](../typescript/ADR-TS-0002-use-lefthook.md)
