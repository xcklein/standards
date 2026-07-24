---
status: accepted
date: 2026-05-16
tags: [typescript, esm, modules, tooling]
---
# Use ESM

## Directive

All TypeScript projects must use ESM. `package.json` must include `"type": "module"` and `tsconfig.json` must set `"module": "NodeNext"` and `"moduleResolution": "NodeNext"`. Import paths must use explicit `.js` extensions.

## Context and Problem Statement

JavaScript has two competing module systems: ECMAScript Modules (ESM) and CommonJS (CJS). ESM is the official standard defined by the ECMAScript specification and is natively supported by all modern runtimes and browsers. CommonJS is a legacy format introduced by Node.js before a standard existed. New projects should align with the standard module system to ensure long-term compatibility and access to modern language features.

## Decision Drivers

* All new TypeScript projects must use the standard module system
* Tooling (bundlers, linters, runtimes) increasingly assume ESM as the default
* Top-level `await` and other modern features require ESM
* Static analysis and tree-shaking work more reliably with ESM's static import/export structure

## Considered Options

* ESM (ECMAScript Modules)
* CommonJS

## Decision Outcome

Chosen option: "ESM", because it is the JavaScript standard and is natively supported by all modern runtimes, browsers, and tooling.

### Examples

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext"
  }
}
```

`package.json`:

```json
{
  "type": "module"
}
```

Imports must use explicit file extensions:

```typescript
import { foo } from './foo.js';
```

### Consequences

* Good, because ESM is the JavaScript standard — supported natively in browsers, Node.js, Deno, and Bun without transpilation
* Good, because static imports enable reliable tree-shaking and dead code elimination in bundlers
* Good, because top-level `await` is available without workarounds
* Good, because import/export structure is statically analysable, improving IDE and tooling support
* Bad, because some older npm packages are CJS-only and require dynamic `import()` or workarounds to consume
* Bad, because file extensions are required in import paths, which differs from the CommonJS convention
* Neutral, because interop between ESM and CJS is possible but adds friction

### Confirmation

`tsconfig.json` must set `"module": "NodeNext"` and `"moduleResolution": "NodeNext"`. `package.json` must include `"type": "module"`. Enforced via ESLint and code review.

## Pros and Cons of the Options

### ESM

* Good, because it is the official ECMAScript standard
* Good, because natively supported in all modern runtimes and browsers
* Good, because enables top-level `await`
* Good, because static structure enables tree-shaking and better tooling
* Bad, because CJS interop can be awkward for older dependencies
* Bad, because requires explicit file extensions in import paths

### CommonJS

* Good, because universally supported across all Node.js versions
* Good, because no file extension required in import paths
* Good, because the entire npm ecosystem supports it
* Bad, because not a standard — it is Node.js-specific and not supported in browsers natively
* Bad, because does not support top-level `await`
* Bad, because dynamic `require()` prevents static analysis and tree-shaking
