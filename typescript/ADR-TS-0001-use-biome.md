---
status: accepted
date: 2026-05-16
tags: [typescript, linting, formatting, tooling]
---
# Use Biome

## Directive

All TypeScript projects must use Biome for linting and formatting. ESLint and Prettier must not be used. Every project must include a `biome.json` following the standard configuration.

## Context and Problem Statement

TypeScript projects require consistent formatting and linting to maintain code quality across teams. Issues should be caught automatically and early, not surfaced during code review. A unified toolchain reduces configuration overhead and eliminates inconsistencies between formatting and linting rules.

## Decision Drivers

* All TypeScript projects must have consistent formatting and linting rules
* Issues should be caught automatically, not in code review
* Tooling should be fast and easy to integrate into CI and editors
* A single tool should handle both formatting and linting to reduce configuration overhead

## Considered Options

* Biome
* ESLint + Prettier
* oxlint

## Decision Outcome

Chosen option: "Biome", because it replaces both ESLint and Prettier with a single fast tool requiring minimal configuration.

### Examples

Every TypeScript project must include a `biome.json` at the root with at minimum the following configuration:

```json
{
  "$schema": "https://biomejs.dev/schemas/latest/schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true
  },
  "formatter": {
    "indentStyle": "space"
  },
  "assist": {
    "actions": {
      "source": {
        "organizeImports": "on"
      }
    }
  },
  "overrides": [
    {
      "includes": ["**/*.test.ts"],
      "linter": {
        "rules": {
          "suspicious": {
            "noExplicitAny": "off"
          }
        }
      }
    }
  ]
}
```

Standard commands:

```bash
# check
biome check .

# apply fixes
biome check --write .
```

### Consequences

* Good, because a single tool replaces both ESLint and Prettier, reducing dependencies and configuration
* Good, because Biome is significantly faster than ESLint + Prettier due to its Rust implementation
* Good, because VCS integration ensures only changed files are checked in large repos
* Good, because consistent formatting is enforced automatically, eliminating style debates in code review
* Bad, because Biome has less ecosystem maturity than ESLint — some rules and plugins have no equivalent yet
* Bad, because teams already invested in ESLint configurations face a migration cost

### Confirmation

A pre-commit hook (see ADR for lefthook) runs `biome check --write .` on staged files before every commit.

## Pros and Cons of the Options

### Biome

* Good, because single tool handles both linting and formatting
* Good, because significantly faster than ESLint + Prettier due to Rust implementation
* Good, because minimal configuration required out of the box
* Good, because built-in VCS integration and import organisation
* Bad, because smaller ecosystem than ESLint — some rules have no equivalent yet
* Bad, because less editor plugin maturity compared to ESLint and Prettier

### ESLint + Prettier

* Good, because industry standard with extensive plugin ecosystem
* Good, because highly configurable and widely understood
* Bad, because two separate tools with separate configurations that can conflict
* Bad, because significantly slower than Biome
* Bad, because formatting and linting rules can contradict each other without careful setup

### oxlint

* Good, because extremely fast, also written in Rust
* Good, because designed as a drop-in ESLint replacement
* Bad, because does not handle formatting — still requires Prettier or similar
* Bad, because ecosystem is immature and rule coverage is incomplete
