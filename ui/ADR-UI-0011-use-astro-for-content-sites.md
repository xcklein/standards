---
status: proposed
date: 2026-10-31
tags: [ui, astro, build, seo, performance]
---
# Use Astro for Content Sites

## Directive

Content sites — sites whose pages are the same for every visitor and are meant to be found through search, such as marketing sites, landing pages, blogs, documentation, and portfolios — must be built with Astro using static output. Interactive parts are React components (see ADR-UI-0001) rendered as islands, hydrated with the least eager `client:*` directive that works. Vite configuration for an Astro site lives under `vite` in `astro.config.ts`, and `site` must be set.

## Context and Problem Statement

A Vite React SPA serves an empty HTML shell and renders everything in the browser. That is fine for an app behind sign-in, but for a content site it means search engines and link previews see little or nothing until JavaScript runs, every visitor downloads the whole app to read one page, and client-side navigation breaks tools that count page loads (analytics, ad networks). Astro renders each page to HTML at build time and ships JavaScript only for the interactive islands on that page, while still using React for those islands, Tailwind for styling, and Vite underneath.

## Decision Drivers

* Content pages must be fully rendered HTML for search engines and link previews
* Visitors should not download JavaScript for parts of a page that are not interactive
* Interactive components should use the same React, shadcn/ui, and Tailwind stack as the rest of the UI standards
* The build should stay on Vite, so Vite plugins and Vitest keep working

## Considered Options

* Astro with static output and React islands
* Vite React SPA for everything
* Next.js

## Decision Outcome

Chosen option: "Astro with static output and React islands", because it serves pre-rendered HTML with JavaScript only where a page is interactive, keeps React and the existing UI stack for that interactivity, and is built on Vite.

### Examples

`astro.config.ts`:

```typescript
import react from "@astrojs/react";
import sitemap from "@astrojs/sitemap";
import tailwindcss from "@tailwindcss/vite";
import { defineConfig } from "astro/config";

export default defineConfig({
  site: "https://example.com",
  integrations: [react(), sitemap()],
  vite: {
    plugins: [tailwindcss()],
  },
});
```

A page with one interactive island — the rest of the page ships no JavaScript:

```astro
---
import Layout from "../layouts/layout.astro";
import { PriceCalculator } from "../components/price-calculator.tsx";
---

<Layout title="Pricing">
  <h1>Pricing</h1>
  <p>Simple plans for every team.</p>
  <PriceCalculator client:visible />
</Layout>
```

Hydration directives, least eager first — use the first one that works:

| Directive | Hydrates |
|---|---|
| none | Never; rendered to static HTML |
| `client:visible` | When scrolled into view |
| `client:idle` | When the browser is idle |
| `client:load` | Immediately on page load |

Tests keep their own `vitest.config.ts`, since they exercise plain TypeScript and React components rather than `.astro` files.

### Consequences

* Good, because content pages arrive as complete HTML, which search engines and link previews can read
* Good, because only interactive islands ship JavaScript
* Good, because islands are ordinary React components using the same UI stack
* Good, because each navigation is a full page load, which analytics and ad tools expect
* Bad, because there are two frontend setups to know (Astro and Vite SPA) instead of one
* Bad, because state shared between islands needs a store such as Zustand rather than React context, since islands are separate React roots
* Neutral, because TanStack Query (ADR-UI-0004) is rarely needed on a static site with no server data

### Confirmation

Code review must flag content sites built as a Vite SPA, Astro sites using server output without a documented reason, and islands using `client:load` where `client:visible` or `client:idle` would work.

## Pros and Cons of the Options

### Astro with static output and React islands

* Good, because pre-rendered HTML with JavaScript only where needed
* Good, because keeps React, Tailwind, shadcn/ui, and Vite
* Bad, because a second frontend setup alongside Vite SPAs

### Vite React SPA for everything

* Good, because one setup for every frontend
* Bad, because content pages are empty until JavaScript runs, which hurts search and link previews
* Bad, because every visitor downloads the whole app

### Next.js

* Good, because supports static, server, and client rendering in one framework
* Bad, because it brings a server rendering and routing model that a static content site does not need
* Bad, because it is a heavier framework with its own build tooling instead of Vite

## More Information

* [Astro — Islands architecture](https://docs.astro.build/en/concepts/islands/)
* [Astro — Client directives](https://docs.astro.build/en/reference/directives-reference/#client-directives)
* Related: [ADR-UI-0001 — Use React](ADR-UI-0001-use-react.md)
* Related: [ADR-UI-0006 — Use Vite](ADR-UI-0006-use-vite.md)
* Reference implementation: xcklein/raster-www
