---
status: accepted
date: 2026-05-16
tags: [typescript, tooling, build, vite]
---
# Use Vite

## Directive

All frontend projects must use Vite as the build tool and dev server. A `vite.config.ts` must be present at the project root.

## Context and Problem Statement

Frontend and TypeScript projects require a build tool to bundle source code, handle assets, and provide a fast development server. Build tool choice affects developer experience through dev server startup time, hot module replacement speed, and configuration complexity. It also affects production output quality through tree-shaking, code splitting, and bundle size.

## Decision Drivers

* Dev server startup and hot module replacement must be fast regardless of project size
* The build tool must support ESM natively (see ADR-TS-0003)
* TypeScript must be supported without additional configuration
* The tool must integrate with the existing ecosystem — React, Tailwind, and vitest

## Considered Options

* Vite
* Next.js

## Decision Outcome

Chosen option: "Vite", because it provides near-instant dev server startup via native ESM, requires minimal configuration for TypeScript and React projects, and shares its module graph with vitest for consistent behaviour between testing and building. Next.js was not chosen because it imposes a server-side rendering and routing model that is not required.

### Examples

Minimal `vite.config.ts` for a React project:

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: { "@": "/src" },
  },
});
```

Standard `package.json` scripts:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview"
  }
}
```

### Consequences

* Good, because dev server starts instantly — Vite serves source files directly via native ESM without bundling
* Good, because hot module replacement is fast and reliable
* Good, because minimal configuration for TypeScript, React, and Tailwind
* Good, because production builds use Rollup, which produces well-optimised, tree-shaken output
* Good, because vitest shares the same module graph — no configuration divergence between tests and builds
* Bad, because dev and production use different bundlers (esbuild for dev, Rollup for prod) — rare but possible behaviour differences
* Bad, because does not provide SSR or file-based routing — a separate router is required for those needs

### Confirmation

All frontend projects must use Vite as the build tool and dev server. `vite.config.ts` must be present at the project root.

## Pros and Cons of the Options

### Vite

* Good, because instant dev server startup via native ESM
* Good, because minimal configuration
* Good, because fast HMR regardless of project size
* Good, because integrates with vitest, Tailwind, and React out of the box
* Bad, because dev/prod bundler difference (esbuild vs Rollup) can cause subtle inconsistencies

### Next.js

* Good, because full-stack framework — SSR, SSG, API routes, and file-based routing included
* Good, because strong ecosystem and Vercel backing
* Bad, because imposes a routing and rendering model that adds complexity for pure SPA use cases
* Bad, because tightly coupled to Vercel's deployment model for full feature support
* Bad, because vitest integration requires additional configuration — Next.js uses its own test setup

## More Information

* [Vite documentation](https://vite.dev)
* Related: [ADR-TS-0003 — Use ESM](../typescript/ADR-TS-0003-use-esm.md)
* Related: [ADR-TS-0005 — Use Vitest](../typescript/ADR-TS-0005-use-vitest.md)
