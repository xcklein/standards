---
status: accepted
date: 2026-05-16
tags: [ui, tailwind, css, styling]
---
# Use Tailwind CSS

## Directive

All component styling must use Tailwind CSS utility classes. Separate `.css` files are only permitted for global styles. The `cn()` utility must be used for all conditional class application.

## Context and Problem Statement

Styling React components requires a consistent, maintainable approach to CSS. Without a system, teams accumulate ad-hoc class names, conflicting stylesheets, and inconsistent design tokens. A utility-first CSS framework applied directly in markup eliminates the need for separate stylesheet maintenance and enforces consistency through a shared design system.

## Decision Drivers

* Styles must be co-located with components — no separate stylesheet files to maintain
* Design tokens (colours, spacing, typography) must be consistent across the application
* Unused styles must be eliminated from the production bundle automatically
* The styling system must integrate with shadcn/ui components

## Considered Options

* Tailwind CSS
* CSS Modules
* styled-components / Emotion

## Decision Outcome

Chosen option: "Tailwind CSS", because it provides a utility-first system that co-locates styles with components, eliminates unused CSS automatically, and is the foundation of the shadcn/ui component library.

### Examples

`tailwind.css` imports Tailwind:

```css
@import "tailwindcss";
```

Utility classes applied directly in JSX:

```tsx
export function UserCard({ username }: { username: string }) {
  return (
    <div className="flex items-center gap-3 rounded-lg border p-4">
      <span className="text-sm font-medium text-foreground">{username}</span>
    </div>
  );
}
```

Use `clsx` and `tailwind-merge` to conditionally apply classes without conflicts:

```tsx
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Usage
<div className={cn("rounded-lg p-4", isActive && "bg-primary text-primary-foreground")} />
```

Custom design tokens are defined in CSS variables and referenced via Tailwind:

```css
/* theme.css */
:root {
  --color-background: oklch(1 0 0);
  --color-foreground: oklch(0.145 0 0);
}
```

### Consequences

* Good, because styles are co-located with components — no context switching between files
* Good, because unused CSS is removed at build time — zero dead styles in production
* Good, because consistent design tokens via utility classes prevent arbitrary magic values
* Good, because required by shadcn/ui — no additional styling system needed
* Bad, because class strings in JSX can become long and hard to read for complex components
* Bad, because Tailwind-specific knowledge is required — not transferable to projects without Tailwind
* Bad, because custom values outside the design system require inline styles or `[]` escape syntax

### Confirmation

All component styling must use Tailwind utility classes. Separate `.css` files are only permitted for global styles (`index.css`, `theme.css`) and Tailwind configuration. The `cn()` utility must be used for all conditional class application.

## Pros and Cons of the Options

### Tailwind CSS

* Good, because co-located styles — no separate stylesheet files
* Good, because automatic dead code elimination
* Good, because consistent design tokens out of the box
* Good, because required foundation for shadcn/ui
* Bad, because verbose class strings for complex components
* Bad, because requires learning Tailwind's utility naming

### CSS Modules

* Good, because scoped styles — no class name collisions
* Good, because standard CSS — no new syntax to learn
* Bad, because requires a separate `.module.css` file per component
* Bad, because no built-in design token system
* Bad, because does not integrate with shadcn/ui

### styled-components / Emotion

* Good, because styles are co-located in JavaScript
* Good, because full CSS power including dynamic styles
* Bad, because runtime CSS-in-JS adds overhead — styles are injected at runtime
* Bad, because incompatible with shadcn/ui's Tailwind-based design system
* Bad, because adds a significant dependency with its own mental model

## More Information

* [Tailwind CSS documentation](https://tailwindcss.com)
* Related: [ADR-UI-0002 — Use shadcn/ui](ADR-UI-0002-use-shadcn.md)
