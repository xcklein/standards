---
status: accepted
date: 2026-07-24
tags: [typescript, formatting, tooling]
---
# Use Prettier

## Directive

All TypeScript projects must use Prettier for formatting. Biome must not be used for formatting. Every project must include a `prettier.config.js` and stay as close to Prettier's defaults as possible — `tabWidth` and `printWidth` are the only settings most projects should need to override.

## Context and Problem Statement

TypeScript projects require consistent formatting so style is enforced automatically and never debated in code review. Biome was previously standard for both linting and formatting, but the linter is now decided separately (see [ADR-TS-0001](ADR-TS-0001-use-linter.md)) in favor of ESLint, and a Biome-only formatter alongside an ESLint linter would mean running two independent Rust/JS toolchains with no shared configuration surface. Prettier is the natural pairing for an ESLint-based project, with `eslint-config-prettier` disabling any overlapping rules.

## Decision Drivers

* Formatting must be enforced automatically, not debated in code review
* The formatter must integrate cleanly with ESLint (see [ADR-TS-0001](ADR-TS-0001-use-linter.md)) without rule conflicts
* The same formatter must apply to every project — no per-project conditional tooling
* Configuration should stay close to defaults so projects are easy to reason about and diff against one another

## Considered Options

* Prettier
* Biome
* dprint

## Decision Outcome

Chosen option: "Prettier", because it is the de facto standard formatter for the ESLint ecosystem, integrates cleanly via `eslint-config-prettier`, and is supported by every major editor and CI tool.

### Examples

Every TypeScript project must include a `prettier.config.js` at the root. Do not override defaults beyond `tabWidth` and `printWidth` unless there is a specific, documented reason:

```javascript
/** @type {import('prettier').Config} */
export default {};
```

Most projects will only need to override `tabWidth` and `printWidth`:

```javascript
/** @type {import('prettier').Config} */
export default {
  printWidth: 90,
  tabWidth: 2,
};
```

A project-specific plugin (e.g. `prettier-plugin-tailwindcss`) is fine to add on top of this, but does not justify changing any other default.

Standard commands:

```bash
# format
prettier --write .

# check formatting without writing
prettier --check .
```

### Consequences

* Good, because formatting is enforced automatically and consistently across every project
* Good, because `eslint-config-prettier` (see [ADR-TS-0001](ADR-TS-0001-use-linter.md)) removes any conflict between lint and format rules
* Good, because Prettier has the widest editor and CI support of any TypeScript formatter
* Good, because sticking to defaults (aside from `tabWidth`/`printWidth`) keeps configuration diffable and easy to reason about across projects
* Bad, because Prettier is slower than Biome's formatter, particularly on large codebases
* Bad, because projects previously on Biome incur a one-time migration cost

### Confirmation

A pre-commit hook (see [ADR-TS-0002](ADR-TS-0002-use-lefthook.md)) runs `prettier --write` on staged files before every commit. CI runs `prettier --check .` to fail builds on unformatted code.

## Pros and Cons of the Options

### Prettier

* Good, because de facto standard, with the widest editor and CI support
* Good, because pairs cleanly with ESLint via `eslint-config-prettier`
* Bad, because slower than Biome's formatter
* Bad, because minimal configuration options — opinionated by design (a plus for consistency, a minus for customization)

### Biome

* Good, because significantly faster than Prettier due to its Rust implementation
* Good, because bundled with a linter, reducing tool count
* Bad, because pairing a Biome formatter with an ESLint linter means two independent toolchains with no shared configuration
* Bad, because smaller plugin/editor ecosystem than Prettier

### dprint

* Good, because fast, written in Rust, and plugin-based
* Bad, because far smaller adoption and community support than Prettier
* Bad, because less mature editor integration

## More Information

* Related: [ADR-TS-0001 — Use ESLint](ADR-TS-0001-use-linter.md)
* Related: [ADR-TS-0002 — Use Lefthook](ADR-TS-0002-use-lefthook.md)
