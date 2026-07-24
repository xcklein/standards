---
status: accepted
date: 2026-07-24
tags: [typescript, linting, tooling, react]
---
# Use ESLint

## Directive

All TypeScript projects must use ESLint for linting. Biome must not be used for linting. Every project must include an `eslint.config.js` (flat config) following the standard configuration.

## Context and Problem Statement

TypeScript projects require consistent linting to catch bugs and enforce conventions automatically rather than in code review. Biome was previously standard for both linting and formatting, but its plugin ecosystem is noticeably thinner than ESLint's — most visibly for React and JSX, which matters for the majority of projects that include a React frontend (see [ADR-UI-0001](../ui/ADR-UI-0001-use-react.md)), but also for other project-specific rule sets as they come up. Linting and formatting are decided as two separate standards (see [ADR-TS-0013](ADR-TS-0013-use-formatter.md) for formatting) so each tool can be evaluated on its own merits.

## Decision Drivers

* Lint rules must catch issues automatically, not in code review
* React and JSX-heavy codebases need mature, well-maintained lint rule coverage
* The same linter must apply to every project — no per-project conditional tooling
* The linter must not own formatting concerns — that is a separate decision

## Considered Options

* ESLint
* Biome
* oxlint

## Decision Outcome

Chosen option: "ESLint", because its plugin ecosystem — particularly `eslint-plugin-react`, `eslint-plugin-react-hooks`, and `typescript-eslint` — provides React rule coverage Biome does not match, and a single linter choice across all projects avoids conditional per-project tooling.

### Examples

Every TypeScript project must include an `eslint.config.js` at the root:

```javascript
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import prettier from "eslint-config-prettier";

export default tseslint.config(
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    files: ["**/*.test.ts"],
    rules: {
      "@typescript-eslint/no-explicit-any": "off",
    },
  },
  prettier,
);
```

The trailing `prettier` import is `eslint-config-prettier` — it disables ESLint's own formatting rules so they cannot conflict with the project's formatter (see [ADR-TS-0013](ADR-TS-0013-use-formatter.md)).

React projects add `eslint-plugin-react` and `eslint-plugin-react-hooks` on top of this base config (see [ADR-UI-0001](../ui/ADR-UI-0001-use-react.md)):

```javascript
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";

export default tseslint.config(
  // ...base config above
  {
    files: ["**/*.tsx"],
    plugins: { react, "react-hooks": reactHooks },
    settings: { react: { version: "detect" } },
    rules: {
      ...react.configs.recommended.rules,
      ...reactHooks.configs.recommended.rules,
      "react/react-in-jsx-scope": "off",
    },
  },
);
```

Standard commands:

```bash
# lint
eslint .

# lint with fixes
eslint . --fix
```

### Consequences

* Good, because `eslint-plugin-react` and `eslint-plugin-react-hooks` give React/JSX projects rule coverage Biome lacks
* Good, because a single linter applies across every project — no conditional per-project selection
* Good, because ESLint's plugin ecosystem covers project-specific needs (imports, a11y, etc.) as they arise
* Bad, because ESLint is slower than Biome, particularly on large codebases
* Bad, because projects previously on Biome incur a one-time migration cost

### Confirmation

A pre-commit hook (see [ADR-TS-0002](ADR-TS-0002-use-lefthook.md)) runs `eslint --fix` on staged files before every commit.

## Pros and Cons of the Options

### ESLint

* Good, because industry-standard plugin ecosystem, including mature React/JSX support
* Good, because highly configurable and widely understood
* Bad, because slower than Biome, especially at scale
* Bad, because configuration can grow complex as plugins accumulate

### Biome

* Good, because fast, with minimal configuration required out of the box
* Good, because bundled with a formatter, reducing tool count
* Bad, because React/JSX rule coverage is noticeably behind ESLint's plugin ecosystem
* Bad, because smaller ecosystem overall — some ESLint rules and plugins have no equivalent

### oxlint

* Good, because extremely fast, written in Rust
* Good, because designed as a drop-in ESLint replacement
* Bad, because ecosystem is immature and rule coverage, including React, is incomplete

## More Information

* Related: [ADR-TS-0013 — Use Prettier](ADR-TS-0013-use-formatter.md)
* Related: [ADR-TS-0002 — Use Lefthook](ADR-TS-0002-use-lefthook.md)
* Related: [ADR-UI-0001 — Use React](../ui/ADR-UI-0001-use-react.md)
