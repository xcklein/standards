---
status: accepted
date: 2026-05-16
tags: [typescript, style, conventions]
---
# Follow Google TypeScript Style Guide

## Directive

All TypeScript styling decisions not covered by the formatter must follow the Google TypeScript Style Guide. ESLint and Prettier take precedence where their rules conflict.

## Context and Problem Statement

ESLint and Prettier handle formatting and a broad set of lint rules, but do not cover every stylistic decision — naming conventions, type annotation style, use of language features, and code organisation are all left to the developer. Without a reference standard for these cases, engineers make inconsistent choices across services that accumulate into a codebase with no coherent style.

## Decision Drivers

* Stylistic decisions not enforced by tooling must have a documented answer
* The style guide must be TypeScript-first, not a JavaScript guide adapted for TypeScript
* The guide must not conflict with ESLint's and Prettier's formatting and lint rules
* The guide must be publicly available and well-maintained so it can be linked to directly

## Considered Options

* Google TypeScript Style Guide
* Airbnb JavaScript/TypeScript Style Guide
* No additional style guide

## Decision Outcome

Chosen option: "Google TypeScript Style Guide", because it is written specifically for TypeScript, is comprehensive, actively maintained, and covers the areas ESLint and Prettier do not — naming, type annotations, language feature usage, and module organisation.

### Examples

A few representative rules from the guide that ESLint does not enforce:

**Naming conventions** — use `camelCase` for variables and functions, `PascalCase` for classes and types, `SCREAMING_SNAKE_CASE` for module-level constants:

```typescript
const MAX_RETRIES = 3;

class UserService { ... }

function fetchUserById(id: string): Promise<User> { ... }
```

**Type annotations** — omit annotations where TypeScript can infer the type; annotate function return types explicitly:

```typescript
// Good — inferred
const count = 0;

// Good — explicit return type on exported functions
export function parseConfig(raw: unknown): Config {
  ...
}
```

**No `any`** — use `unknown` for values of unknown type and narrow explicitly:

```typescript
function handle(value: unknown) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  }
}
```

### Consequences

* Good, because engineers have a documented answer for stylistic questions not covered by ESLint or Prettier
* Good, because the guide is TypeScript-first and covers the language deeply
* Good, because it is publicly available — teams can link to specific rules rather than explaining them verbally
* Bad, because not all rules in the guide are enforceable by tooling — adherence relies on code review

### Confirmation

The Google TypeScript Style Guide is the reference for code review discussions about style not covered by ESLint or Prettier. Link to the relevant section of the guide when raising style feedback.

## Pros and Cons of the Options

### Google TypeScript Style Guide

* Good, because TypeScript-first — written for TypeScript, not adapted from JavaScript
* Good, because comprehensive coverage of naming, types, and language features
* Good, because actively maintained by Google
* Bad, because some rules overlap with or are superseded by ESLint/Prettier — requires knowing which takes precedence

### Airbnb Style Guide

* Good, because widely recognised and commonly used in the JavaScript community
* Bad, because primarily a JavaScript guide — TypeScript coverage is an add-on, not first-class
* Bad, because more opinionated on React patterns, which overlaps with other ADRs

### No additional style guide

* Good, because no rules beyond what tooling enforces
* Bad, because leaves naming conventions, type annotation style, and language feature usage undefined
* Bad, because inconsistency accumulates across services with no reference to resolve disagreements

## More Information

* [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
* Related: [ADR-TS-0001 — Use ESLint](ADR-TS-0001-use-linter.md)
* Related: [ADR-TS-0013 — Use Prettier](ADR-TS-0013-use-formatter.md)
