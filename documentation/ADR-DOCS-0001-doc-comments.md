---
status: accepted
date: 2026-07-24
tags: [documentation, jsdoc, javadoc, style]
---
# Doc Comments

## Directive

All exported/public functions, classes, methods, and types must have a documentation comment (JSDoc, Javadoc, or the language's equivalent). Doc comments must describe purpose, parameters, return values, and thrown/rejected errors where applicable. The free-form description (excluding `@param`/`@returns`/`@throws` tags) should be limited to around a dozen lines. If that is not enough to explain the symbol, create a separate supporting document and reference it from the doc comment rather than letting the comment grow indefinitely. Doc comments must never reference git commits, branches, pull requests, or issues/tickets — only source-controlled documentation (in this repository or another repository).

## Context and Problem Statement

Public API surfaces are consumed by other developers, other services, and AI coding agents without those readers first tracing the implementation. Without a documentation comment on every exported symbol, understanding a function's contract requires reading its body, its call sites, or asking the original author. Doc comment formats (JSDoc, Javadoc) are colocated with the code, surfaced automatically by IDEs and language servers as hover documentation, and are the format both human readers and AI agents already expect.

## Decision Drivers

* Public API surfaces must be understandable without reading the implementation
* IDEs and language servers surface JSDoc/Javadoc as inline hover documentation automatically
* AI coding agents rely on doc comments to infer intent and contracts without traversing the full call graph
* Documentation colocated with code is far less likely to drift out of sync than a separate docs site or wiki
* A doc comment must stay short enough to read at a glance in an editor tooltip or hover card; a description that has grown into a multi-page explanation belongs in a separate document, not an ever-expanding comment block
* A reference in a doc comment must remain resolvable to anyone with just the codebase — commits, branches, and PR/issue numbers depend on an external tracker's retention and access policy, while source-controlled documentation does not

## Considered Options

* Doc comments (JSDoc/Javadoc) required on all public API surface
* No required documentation standard — rely on naming and code review
* External documentation only (separate docs site or wiki)

## Decision Outcome

Chosen option: "Doc comments required on all public API surface", because it is colocated with the code it describes, is enforced by tooling, is surfaced automatically in editors, and stays in sync with signature changes far more reliably than external documentation.

### Examples

JSDoc on an exported TypeScript function:

```typescript
/**
 * Parses and validates a raw configuration object.
 *
 * @param raw - Untyped input, typically `process.env`.
 * @returns The validated, fully typed configuration.
 * @throws {ZodError} If any required field is missing or malformed.
 */
export function parseConfig(raw: unknown): Config {
  return schema.parse(raw);
}
```

Javadoc on a public Java method:

```java
/**
 * Parses and validates a raw configuration object.
 *
 * @param raw untyped input, typically system environment variables
 * @return the validated, fully typed configuration
 * @throws ValidationException if any required field is missing or malformed
 */
public Config parseConfig(Map<String, String> raw) {
    return schema.parse(raw);
}
```

Private/internal helpers are not required to carry a doc comment unless their behavior is non-obvious (see [ADR-DOCS-0002](ADR-DOCS-0002-inline-comments.md)).

When a symbol's behavior needs more than about a dozen lines of description, link out to a supporting document instead of letting the comment sprawl:

```typescript
/**
 * Reconciles the local cart with the server's authoritative state.
 *
 * See `docs/cart-reconciliation.md` for the full merge algorithm and the
 * edge cases it exists to handle.
 *
 * @param local - The cart as currently held in client state.
 * @param remote - The cart as last returned by the server.
 * @returns The merged cart to render and persist.
 */
export function reconcileCart(local: Cart, remote: Cart): Cart { ... }
```

Bad — references a PR instead of documentation:

```typescript
/**
 * Reconciles the local cart with the server's authoritative state.
 *
 * See PR #1842 for why server-wins-on-conflict was chosen.
 */
export function reconcileCart(local: Cart, remote: Cart): Cart { ... }
```

Good — a cross-repo documentation reference is allowed:

```typescript
/**
 * Reconciles the local cart with the server's authoritative state.
 *
 * See docs/cart-reconciliation.md in xcklein/olatile-api for the full
 * merge algorithm this mirrors on the client.
 */
export function reconcileCart(local: Cart, remote: Cart): Cart { ... }
```

### Consequences

* Good, because public APIs are self-documenting and understandable without reading the implementation
* Good, because IDEs and language servers surface the documentation automatically as hover text
* Good, because AI coding agents can infer intent and contracts directly from the doc comment
* Good, because documentation colocated with code drifts out of sync far less than external docs
* Good, because capping description length keeps doc comments quick to read in an editor tooltip instead of growing into unreadable blocks
* Good, because referencing documentation instead of VCS metadata keeps every doc comment resolvable regardless of an external tracker's retention or renumbering
* Bad, because doc comments add maintenance overhead when signatures change
* Bad, because doc comments can still go stale if not enforced in code review or tooling

### Confirmation

A lint rule (e.g. `eslint-plugin-jsdoc`'s `require-jsdoc` for TypeScript, Checkstyle's `JavadocMethod` for Java) must flag exported/public symbols missing a doc comment. Code review is the fallback for languages without an equivalent lint rule, and must also flag doc comment descriptions that have grown well past a dozen lines, or that reference a commit, branch, or PR/issue number, asking for a linked supporting document instead.

## Pros and Cons of the Options

### Doc comments required on public API surface

* Good, because colocated with code — changes to behavior and documentation happen in the same diff
* Good, because tooling (IDEs, lint rules) can enforce and surface them automatically
* Bad, because adds authoring overhead to every public symbol

### No required documentation standard

* Good, because zero authoring overhead
* Bad, because understanding any public API requires reading its implementation or call sites
* Bad, because inconsistent — some symbols end up documented, most do not

### External documentation only

* Good, because can include longer-form narrative and examples than a doc comment allows
* Bad, because lives separately from the code and reliably drifts out of sync
* Bad, because not surfaced automatically by IDEs at the call site

## More Information

* Related: [ADR-DOCS-0002 — Inline Comments](ADR-DOCS-0002-inline-comments.md)
* Related: [ADR-TS-0001 — Use ESLint](../typescript/ADR-TS-0001-use-linter.md)
* Related: [ADR-TS-0012 — Follow Google TypeScript Style Guide](../typescript/ADR-TS-0012-google-typescript-style-guide.md)
