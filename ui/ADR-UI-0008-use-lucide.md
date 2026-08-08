---
status: accepted
date: 2026-08-08
tags: [ui, react, icons]
---
# Use Lucide

## Directive

All icons must come from `lucide-react`. Icon imports must use the `XxxIcon` suffixed export (e.g. `DownloadIcon`) — never the deprecated unsuffixed alias (e.g. `Download`). Enforcing this with a lint rule is recommended over relying on convention or code review alone.

## Context and Problem Statement

`lucide-react` ships two names for every icon: a legacy unsuffixed export (`Download`) kept for backward compatibility, and an `Icon`-suffixed export (`DownloadIcon`) that is now the documented, forward-looking name. The unsuffixed form is more prone to colliding with unrelated identifiers of the same name (component props, DOM types, other libraries) since it carries no indication that it is an icon. Without an enforced rule, a codebase accumulates a mix of both forms as different contributors reach for whichever autocomplete suggests first.

## Decision Drivers

* Icon components must be visually and grammatically distinguishable from other imports at a glance
* The unsuffixed export name is more likely to collide with unrelated identifiers
* Catching the naming convention with tooling is preferable to relying on code review alone
* The library must have a comprehensive icon set and be actively maintained

## Considered Options

* `lucide-react`
* `react-icons`
* `@heroicons/react`

## Decision Outcome

Chosen option: "`lucide-react`", because it has a large, actively maintained icon set, ships first-class TypeScript types, and its `Icon`-suffixed export naming is straightforward to enforce with tooling to keep every import unambiguous.

### Examples

```tsx
import { DownloadIcon, ShareIcon } from "lucide-react";

export function ExportControls() {
  return (
    <div className="flex gap-2">
      <button aria-label="Download">
        <DownloadIcon />
      </button>
      <button aria-label="Share">
        <ShareIcon />
      </button>
    </div>
  );
}
```

Bad — the deprecated unsuffixed alias:

```tsx
import { Download, Share } from "lucide-react";
```

Example ESLint rule enforcing the suffix (add to the project's `no-restricted-syntax` rules) — one possible implementation, not the only valid one:

```javascript
{
  selector:
    "ImportDeclaration[source.value='lucide-react'] > ImportSpecifier[imported.name=/^(?!LucideIcon$|LucideProps$|createLucideIcon$)[A-Z][A-Za-z0-9]*(?<!Icon)$/]",
  message:
    "Import the `XxxIcon` suffixed export from lucide-react instead of the unsuffixed alias.",
}
```

The example excludes `LucideIcon` and `LucideProps` (the library's own type exports) and `createLucideIcon` (the factory function used to build custom icons), since none of those are icon components and none should carry the `Icon` suffix.

### Consequences

* Good, because every icon import is unambiguous at a glance — the `Icon` suffix marks it as an icon, not a component, prop, or unrelated identifier
* Good, because the unsuffixed alias's naming collisions are eliminated entirely
* Good, because a lint rule catches the convention automatically instead of relying on code review to notice it
* Bad, because a lint-rule selector like the example above must be maintained if the library adds new non-icon exports that don't fit the `Icon`-suffix pattern
* Neutral, because adopting the recommended lint rule is one additional custom rule beyond the project's base ESLint config

### Confirmation

Code review must flag unsuffixed `lucide-react` icon imports. Adopting a lint rule (such as the example above) to catch this automatically in CI (see [ADR-TS-0001](../typescript/ADR-TS-0001-use-linter.md)) is recommended.

## Pros and Cons of the Options

### `lucide-react`

* Good, because large, actively maintained icon set with first-class TypeScript types
* Good, because the `Icon`-suffixed naming convention is mechanically enforceable with one lint rule
* Bad, because the legacy unsuffixed aliases still exist and must be actively excluded

### `react-icons`

* Good, because aggregates multiple icon sets (Font Awesome, Material, Feather, etc.) behind one API
* Bad, because bundle size grows quickly if imports aren't tree-shaken carefully across sets
* Bad, because visual inconsistency is easy to introduce by mixing icon sets within one UI

### `@heroicons/react`

* Good, because designed to pair with Tailwind CSS, matching this project's styling approach (see [ADR-UI-0003](ADR-UI-0003-use-tailwind.md))
* Bad, because a smaller icon set than Lucide — more likely to require a second library for missing icons
* Bad, because separate `outline`/`solid` import paths add friction compared to Lucide's single export

## More Information

* Related: [ADR-UI-0001 — Use React](ADR-UI-0001-use-react.md)
* Related: [ADR-TS-0001 — Use ESLint](../typescript/ADR-TS-0001-use-linter.md)
* [Lucide icon library](https://lucide.dev)
