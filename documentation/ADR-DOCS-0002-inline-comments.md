---
status: accepted
date: 2026-07-24
tags: [documentation, style, code-review]
---
# Inline Comments

## Directive

Inline code comments must be used sparingly. They are permitted only to explain unexpected or non-obvious behavior, or to explicitly note the deliberate absence of something (e.g., a `catch` block with no `throw`). Comments must not restate what the code already says.

## Context and Problem Statement

Inline comments are not enforced by the compiler or tests and can drift out of sync with the code they describe as the code changes underneath them. Comments that merely restate what the following line already says add visual noise without adding information, and train readers to skim past comments entirely. At the same time, some behavior is genuinely surprising — a workaround for a specific bug, a non-obvious ordering requirement — and some omissions are deliberate rather than accidental; without a comment, both look like bugs and invite a future "fix" that reintroduces the original problem.

## Decision Drivers

* Comments that restate the code add noise without adding information
* Unenforced comments can silently drift out of sync with the code they describe
* Genuinely surprising behavior must be explained once, at the point of surprise, to prevent future regressions
* Deliberate omissions (no `throw`, no `default` case, no `return`) are indistinguishable from bugs unless explicitly marked as intentional

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
// first connection after a cold start (see INC-4821).
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

### Consequences

* Good, because every comment that exists carries real information, so comments are read and trusted rather than skimmed past
* Good, because fewer comments means fewer opportunities for a comment to drift out of sync with the code
* Good, because deliberate omissions are distinguished from bugs, preventing well-intentioned "fixes" that reintroduce a problem
* Bad, because relies on developer and reviewer judgment to identify what counts as "non-obvious"
* Bad, because an author close to the problem may under-comment reasoning that is obvious to them but not to future readers

### Confirmation

Code review must flag inline comments that restate the following line and request their removal, and must flag non-obvious logic or deliberate omissions left unexplained.

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
