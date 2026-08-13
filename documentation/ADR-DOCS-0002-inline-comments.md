---
status: accepted
date: 2026-07-24
tags: [documentation, style, code-review]
---
# Inline Comments

## Directive

Inline code comments must be used sparingly. They are permitted only to explain unexpected or non-obvious behavior, or to explicitly note the deliberate absence of something (e.g., a `catch` block with no `throw`). Comments must not restate what the code already says. An inline comment should be limited to about three lines. If that is not enough to explain the situation, create a separate supporting document and reference it from the comment rather than letting the comment grow indefinitely. Inline comments must never reference git commits, branches, pull requests, or issues/tickets — only source-controlled documentation (in this repository or another repository).

## Context and Problem Statement

Inline comments are not enforced by the compiler or tests and can drift out of sync with the code they describe as the code changes underneath them. Comments that merely restate what the following line already says add visual noise without adding information, and train readers to skim past comments entirely. At the same time, some behavior is genuinely surprising — a workaround for a specific bug, a non-obvious ordering requirement — and some omissions are deliberate rather than accidental; without a comment, both look like bugs and invite a future "fix" that reintroduces the original problem.

## Decision Drivers

* Comments that restate the code add noise without adding information
* Unenforced comments can silently drift out of sync with the code they describe
* Genuinely surprising behavior must be explained once, at the point of surprise, to prevent future regressions
* Deliberate omissions (no `throw`, no `default` case, no `return`) are indistinguishable from bugs unless explicitly marked as intentional
* An inline comment must stay short enough to read in place without breaking the reader's flow through the surrounding code; an explanation that needs more room belongs in a separate document, not a sprawling comment block
* A reference in an inline comment must remain resolvable to anyone with just the codebase — commits, branches, and PR/issue numbers depend on an external tracker's retention and access policy, while source-controlled documentation does not

## Considered Options

* Sparing inline comments — only for non-obvious behavior or deliberate omissions
* Liberal inline comments — comment every non-trivial block
* No inline comments — rely entirely on doc comments and naming

## Decision Outcome

Chosen option: "Sparing inline comments", because it keeps the signal-to-noise ratio high — every comment that exists tells the reader something they could not have inferred from the code itself, so comments are trusted rather than skimmed past.

### Examples

Bad — restates what the code already says:

```typescript
// Loop through all users
for (const user of users) {
  // Increment the counter
  count++;
}
```

Good — explains genuinely non-obvious behavior:

```typescript
// Retry once before failing: the upstream API occasionally drops the
// first connection after a cold start (see docs/incidents/2026-06-cold-start.md).
const response = await fetchWithRetry(url, { retries: 1 });
```

Bad — references a ticket instead of documentation:

```typescript
// Retry once before failing — see INC-4821.
const response = await fetchWithRetry(url, { retries: 1 });
```

Good — marks a deliberate omission:

```typescript
try {
  await cache.warm();
} catch {
  // Intentionally swallowed: a cold cache is a performance hit, not a
  // failure — the request path below still works without it.
}
```

Good — the explanation doesn't fit in three lines, so it links out instead of sprawling:

```typescript
// Merge order here matters — see docs/cart-reconciliation.md for why
// server-wins-on-conflict was chosen over last-write-wins.
const merged = reconcile(local, remote);
```

Good — a cross-repo documentation reference is allowed:

```typescript
// Kept for mobile Safari compatibility — see docs/browser-compat.md
// in xcklein/olatile-www for the full history.
const useLegacyLayout = true;
```

### Consequences

* Good, because every comment that exists carries real information, so comments are read and trusted rather than skimmed past
* Good, because fewer comments means fewer opportunities for a comment to drift out of sync with the code
* Good, because deliberate omissions are distinguished from bugs, preventing well-intentioned "fixes" that reintroduce a problem
* Good, because capping comments at about three lines keeps them readable in place instead of interrupting the surrounding code with a wall of prose
* Good, because referencing documentation instead of VCS metadata keeps every comment resolvable regardless of an external tracker's retention or renumbering
* Bad, because relies on developer and reviewer judgment to identify what counts as "non-obvious"
* Bad, because an author close to the problem may under-comment reasoning that is obvious to them but not to future readers

### Confirmation

Code review must flag inline comments that restate the following line and request their removal, must flag non-obvious logic or deliberate omissions left unexplained, and must flag comments that have grown past about three lines or that reference a commit, branch, or PR/issue number, asking for a linked supporting document instead.

## Pros and Cons of the Options

### Sparing inline comments

* Good, because high signal-to-noise ratio — comments are trusted
* Good, because fewer comments to keep in sync as code changes
* Bad, because requires judgment calls about what qualifies as "non-obvious"

### Liberal inline comments

* Good, because more context is available for unfamiliar readers
* Bad, because most comments restate the code, adding noise
* Bad, because a large volume of comments makes stale ones harder to spot

### No inline comments

* Good, because zero risk of comments drifting out of sync
* Bad, because genuinely surprising behavior and deliberate omissions go unexplained, inviting regressions

## More Information

* Related: [ADR-DOCS-0001 — Doc Comments](ADR-DOCS-0001-doc-comments.md)
* Related: [ADR-TS-0012 — Follow Google TypeScript Style Guide](../typescript/ADR-TS-0012-google-typescript-style-guide.md)
