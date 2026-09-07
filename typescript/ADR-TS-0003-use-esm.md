---
status: accepted
date: 2026-05-16
tags: [typescript, esm, modules, tooling]
---
# Use ESM

## Directive

All TypeScript projects must use ESM, not CommonJS. `package.json` must include `"type": "module"`. The `tsconfig.json` module settings depend on how the project is run:

* **Node-runtime projects** (APIs, CLIs, scripts run directly by Node) must set `"module"` and `"moduleResolution"` to `"NodeNext"` (or `"Node16"`, which applies the same loader rules), and relative import paths must carry an explicit extension. Either spelling of that extension is acceptable:
  * `.js` — the emitted path, written by hand.
  * `.ts` — the real path, with `"allowImportingTsExtensions"` and `"rewriteRelativeImportExtensions"` set so TypeScript rewrites it to `.js` on emit. Requires TypeScript 5.7 or later.
* **Bundler-driven frontend projects** (React via Vite, see [ADR-UI-0006](../ui/ADR-UI-0006-use-vite.md)) must set `"module": "ESNext"` and `"moduleResolution": "bundler"`, and import paths must **not** include file extensions.

A project must pick one extension style and apply it consistently; mixing `.js` and `.ts` relative imports in the same project is not permitted.

## Context and Problem Statement

JavaScript has two competing module systems: ECMAScript Modules (ESM) and CommonJS (CJS). ESM is the official standard defined by the ECMAScript specification and is natively supported by all modern runtimes and browsers. CommonJS is a legacy format introduced by Node.js before a standard existed. New projects should align with the standard module system to ensure long-term compatibility and access to modern language features.

`NodeNext` module resolution enforces Node's own ESM loader rules — notably that relative imports must include an explicit file extension. Historically that extension had to be written as `.js` even though the file on disk was `.ts`, which reads as a lie about the filesystem and breaks editor "go to file" on the literal path. TypeScript 5.7's `rewriteRelativeImportExtensions` removes that trade-off: source can import `./thing.ts` and the compiler rewrites it to `./thing.js` on emit. Both spellings satisfy Node's loader; the choice is a readability preference, so the standard permits either as long as a project is internally consistent. Bundler-driven frontend tooling (Vite, and the `bundler` resolution mode built for it) does its own module resolution and expects extensionless imports; applying `NodeNext` rules to a Vite/React project produces incorrect import paths or fights the framework's own conventions. The choice of ESM over CJS is universal, but the mechanical tsconfig settings that enforce it are not — they must match how the code is actually loaded.

## Decision Drivers

* All new TypeScript projects must use the standard module system
* Tooling (bundlers, linters, runtimes) increasingly assume ESM as the default
* Top-level `await` and other modern features require ESM
* Static analysis and tree-shaking work more reliably with ESM's static import/export structure
* The specific module resolution setting must match how the code is actually loaded — Node's own loader for backend/CLI projects, the bundler's resolver for frontend projects

## Considered Options

* ESM (ECMAScript Modules)
* CommonJS

## Decision Outcome

Chosen option: "ESM", because it is the JavaScript standard and is natively supported by all modern runtimes, browsers, and tooling.

### Examples

`package.json` (all projects):

```json
{
  "type": "module"
}
```

**Node-runtime projects** — `tsconfig.json`:

```json
{
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext"
  }
}
```

Imports must use explicit file extensions:

```typescript
import { foo } from './foo.js';
```

**Bundler-driven frontend projects** — `tsconfig.json`:

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler"
  }
}
```

Imports must not include file extensions — Vite resolves them:

```typescript
import { Foo } from './foo';
```

### Consequences

* Good, because ESM is the JavaScript standard — supported natively in browsers, Node.js, Deno, and Bun without transpilation
* Good, because static imports enable reliable tree-shaking and dead code elimination in bundlers
* Good, because top-level `await` is available without workarounds
* Good, because import/export structure is statically analysable, improving IDE and tooling support
* Bad, because some older npm packages are CJS-only and require dynamic `import()` or workarounds to consume
* Bad, because Node-runtime projects require file extensions in import paths, which differs from the CommonJS convention
* Bad, because two different module-resolution configurations must be documented and chosen correctly depending on project type
* Neutral, because interop between ESM and CJS is possible but adds friction

### Confirmation

`package.json` must include `"type": "module"` in every project. Node-runtime projects must set `"module"`/`"moduleResolution"` to `"NodeNext"` or `"Node16"` in `tsconfig.json`, with an explicit extension on every relative import — `.js`, or `.ts` alongside `"allowImportingTsExtensions"` and `"rewriteRelativeImportExtensions"`. Bundler-driven frontend projects must set `"module": "ESNext"` / `"moduleResolution": "bundler"`, with no file extensions on relative imports. Enforced via ESLint and code review.

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
