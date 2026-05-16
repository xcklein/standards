---
status: accepted
date: 2026-05-16
tags: [ui, react, state, zustand]
---
# Use Zustand

## Directive

All shared client UI state must be managed with Zustand. Each store must have a single clear responsibility. Server state belongs in TanStack Query; local component state belongs in `useState`.

## Context and Problem Statement

UI applications require client-side state management for data that is not server state — UI state, user preferences, and shared ephemeral state that must be accessible across multiple components. React's built-in `useState` and `useContext` are sufficient for local and lightly-shared state, but become unwieldy for larger state objects that are read and written across many components. A dedicated client state library provides a clean, performant solution without the ceremony of Redux.

## Decision Drivers

* Client state that is shared across multiple components must be accessible without prop drilling
* The solution must be lightweight and not require a provider or boilerplate
* State updates must trigger re-renders only in components that consume the changed slice
* The library must be TypeScript-first

## Considered Options

* Zustand
* Redux Toolkit
* Jotai
* React Context + useReducer

## Decision Outcome

Chosen option: "Zustand", because it provides a minimal, boilerplate-free store API that is TypeScript-first, requires no provider, and supports fine-grained subscriptions to prevent unnecessary re-renders.

### Examples

Define a typed store:

```typescript
import { create } from "zustand";

interface GridState {
  width: number | null;
  height: number | null;
  tiles: Map<string, Tile>;
  setDimensions: (width: number, height: number) => void;
  setTile: (key: string, tile: Tile) => void;
}

export const useGridStore = create<GridState>((set) => ({
  width: null,
  height: null,
  tiles: new Map(),
  setDimensions: (width, height) => set({ width, height }),
  setTile: (key, tile) =>
    set((state) => ({ tiles: new Map(state.tiles).set(key, tile) })),
}));
```

Consume only the slice needed — only re-renders when `width` changes:

```tsx
const width = useGridStore((state) => state.width);
```

### Consequences

* Good, because no provider required — stores are importable anywhere without wrapping the component tree
* Good, because fine-grained selectors prevent unnecessary re-renders
* Good, because minimal boilerplate — a store is a single `create` call
* Good, because TypeScript-first with full inference
* Bad, because stores are global singletons — testing requires resetting store state between tests
* Bad, because without discipline, stores can accumulate unrelated state and become difficult to reason about

### Confirmation

Zustand is used for client-side UI state only — not for server state (use TanStack Query) or local component state (use `useState`). Each store must have a clear, single responsibility.

## Pros and Cons of the Options

### Zustand

* Good, because no provider or boilerplate required
* Good, because fine-grained subscriptions for performance
* Good, because TypeScript-first
* Bad, because global singletons complicate test isolation

### Redux Toolkit

* Good, because powerful DevTools and time-travel debugging
* Good, because well-established patterns for large teams
* Bad, because significant boilerplate even with Redux Toolkit
* Bad, because overkill for most client state use cases
* Bad, because requires a provider

### Jotai

* Good, because atomic model — granular state with minimal re-renders
* Good, because no provider required
* Bad, because atomic model is a different mental model from object stores
* Bad, because smaller community than Zustand

### React Context + useReducer

* Good, because built into React — no additional dependency
* Bad, because any context value change re-renders all consumers — performance degrades with large state
* Bad, because requires a provider for every store
* Bad, because verbose compared to Zustand

## More Information

* [Zustand documentation](https://zustand.docs.pmnd.rs)
* Related: [ADR-UI-0004 — Use TanStack Query](ADR-UI-0004-use-tanstack-query.md)
