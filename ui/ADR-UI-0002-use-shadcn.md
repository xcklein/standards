---
status: accepted
date: 2026-05-16
tags: [ui, components, shadcn, radix, tailwind]
---
# Use shadcn/ui

## Directive

All shared UI primitives must be installed via the shadcn CLI into `src/components/ui/`. Raw Radix UI primitives must not be used directly outside of that directory.

## Context and Problem Statement

UI applications require a set of accessible, styled components for common interface elements such as buttons, dialogs, inputs, and selects. A component library must balance accessibility, customisability, and design consistency. Libraries that ship pre-compiled components are difficult to customise deeply; libraries that require building everything from scratch are slow to develop with.

## Decision Drivers

* Components must be accessible out of the box — keyboard navigation, ARIA attributes, and focus management must be handled correctly
* Components must be fully customisable — design tokens, styles, and behaviour must be overridable without fighting the library
* Components must integrate with Tailwind CSS
* New components must be addable without increasing the base bundle size

## Considered Options

* shadcn/ui
* MUI (Material UI)
* Building from scratch

## Decision Outcome

Chosen option: "shadcn/ui", because it copies component source code directly into the project rather than shipping a compiled package, giving full ownership of each component while building on Radix UI's accessible primitives and Tailwind for styling.

### Examples

Add a component using the shadcn CLI:

```bash
pnpx shadcn add button
pnpx shadcn add dialog
```

This copies the component source into `src/components/ui/`. Components are owned by the project and can be modified freely:

```tsx
import { Button } from "@/components/ui/button";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";

export function ConfirmDialog({ onConfirm }: { onConfirm: () => void }) {
  return (
    <Dialog>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Are you sure?</DialogTitle>
        </DialogHeader>
        <Button onClick={onConfirm}>Confirm</Button>
      </DialogContent>
    </Dialog>
  );
}
```

### Consequences

* Good, because components are owned by the project — no upstream breaking changes affect them
* Good, because full customisability — components are plain TypeScript and Tailwind
* Good, because Radix UI primitives provide accessibility for free
* Good, because only installed components are in the bundle — no unused code
* Bad, because updates to shadcn components are not automatic — improvements must be pulled in manually
* Bad, because customised components diverge from upstream over time, making manual updates harder

### Confirmation

All shared UI primitives must come from `src/components/ui/` and be installed via the shadcn CLI. Custom one-off components live in `src/components/`. Raw Radix UI primitives must not be used directly outside of `src/components/ui/`.

## Pros and Cons of the Options

### shadcn/ui

* Good, because full ownership — components are source, not a black-box package
* Good, because accessible via Radix UI primitives
* Good, because styled with Tailwind — consistent with the rest of the codebase
* Good, because zero bundle overhead for unused components
* Bad, because updates require manual intervention
* Bad, because components diverge from upstream with customisation

### MUI (Material UI)

* Good, because comprehensive — covers nearly every UI pattern
* Good, because well-documented with a large community
* Bad, because opinionated Material Design aesthetic is difficult to override completely
* Bad, because large bundle size
* Bad, because uses its own styling system (Emotion/sx prop) rather than Tailwind

### Building from scratch

* Good, because maximum control over every component
* Bad, because significant upfront effort to build, style, and ensure accessibility for each component
* Bad, because essentially what shadcn/ui provides — no reason to reinvent it

## More Information

* [shadcn/ui documentation](https://ui.shadcn.com)
* [Radix UI](https://www.radix-ui.com)
* Related: [ADR-UI-0003 — Use Tailwind CSS](ADR-UI-0003-use-tailwind.md)
