---
status: accepted
date: 2026-05-16
tags: [typescript, testing, tooling]
---
# Use Vitest

## Directive

All TypeScript projects must use Vitest as the test runner. Jest must not be used. A `vitest.config.ts` must be present and `package.json` must include `"test": "vitest run"`.

## Context and Problem Statement

TypeScript projects require a test runner that integrates well with the modern ESM-based toolchain. Jest, the historical default, requires significant configuration to work with ESM and TypeScript, and is slower than modern alternatives. Vitest was designed for ESM-first projects, shares Vite's configuration, and provides a Jest-compatible API that minimises migration friction.

## Decision Drivers

* The test runner must work natively with ESM (see ADR-TS-0003)
* TypeScript must be supported without a separate transpilation step
* Tests must run fast, including in CI
* The API should be familiar to developers with Jest experience

## Considered Options

* Vitest
* Jest
* Node.js built-in test runner (`node:test`)
* Mocha + Sinon

## Decision Outcome

Chosen option: "Vitest", because it is ESM-native, requires minimal configuration for TypeScript projects, and is significantly faster than Jest.

### Examples

`vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'lcov'],
    },
  },
});
```

`package.json` scripts:

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

Example test file:

```typescript
import { describe, it, expect } from 'vitest';
import { add } from './math.js';

describe('add', () => {
  it('returns the sum of two numbers', () => {
    expect(add(1, 2)).toBe(3);
  });
});
```

### Consequences

* Good, because Vitest is ESM-native — no additional configuration needed to work with `"type": "module"`
* Good, because TypeScript is supported out of the box via esbuild transpilation
* Good, because significantly faster than Jest due to Vite's optimised module graph
* Good, because Jest-compatible API means existing test knowledge transfers directly
* Good, because watch mode is fast and reliable for local development
* Bad, because Vitest is less mature than Jest — some edge cases and plugins are unsupported
* Bad, because projects using Vite for both bundling and testing are tightly coupled to the Vite ecosystem

### Confirmation

`package.json` must list `vitest` as a dev dependency. CI runs `vitest run` and fails on test failures. Coverage thresholds are enforced via `vitest run --coverage`.

## Pros and Cons of the Options

### Vitest

* Good, because ESM-native with zero extra configuration
* Good, because TypeScript supported out of the box
* Good, because fast execution due to Vite's module graph
* Good, because Jest-compatible API reduces learning curve
* Bad, because less mature than Jest
* Bad, because ties projects to the Vite ecosystem

### Jest

* Good, because the most widely adopted JavaScript test runner
* Good, because extensive plugin and matcher ecosystem
* Bad, because requires `ts-jest` or Babel to handle TypeScript
* Bad, because ESM support requires additional configuration and is still incomplete
* Bad, because significantly slower than Vitest

### Node.js built-in test runner (`node:test`)

* Good, because zero dependencies — built into Node.js
* Good, because ESM-native
* Bad, because minimal API — lacks mocking, snapshot testing, and coverage out of the box
* Bad, because less ergonomic than Vitest or Jest for large test suites
* Bad, because ecosystem tooling (reporters, plugins) is immature

### Mocha + Sinon

* Good, because highly composable — each concern (running, asserting, mocking) is a separate library
* Good, because Sinon provides powerful spies, stubs, and fakes with fine-grained control
* Good, because long-established with a large community
* Bad, because requires assembling and configuring multiple libraries (Mocha, Chai, Sinon) for a complete setup
* Bad, because ESM support in Mocha requires additional configuration
* Bad, because no built-in TypeScript support — requires ts-node or similar
* Bad, because slower than Vitest

## More Information

* [Vitest documentation](https://vitest.dev)
* Related: [ADR-TS-0003 — Use ESM](ADR-TS-0003-use-esm.md)
