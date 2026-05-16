---
status: accepted
date: 2026-05-16
tags: [ui, react, framework]
---
# Use React

## Directive

All UI must be built with React. Component files must use the `.tsx` extension. `StrictMode` must be enabled in the root entry point.

## Context and Problem Statement

Frontend applications require a UI framework for building component-based interfaces. The choice of framework affects the ecosystem of available libraries, the hiring pool, long-term support, and interoperability with tooling such as component libraries and state management solutions.

## Decision Drivers

* The framework must have a large, stable ecosystem of compatible libraries
* TypeScript support must be first-class
* The framework must support server-side rendering and static export for future flexibility
* Developer familiarity and hiring considerations favour widely adopted solutions

## Considered Options

* React
* Vue
* Angular

## Decision Outcome

Chosen option: "React", because it has the largest ecosystem, first-class TypeScript support, and is the foundation for the component and state management libraries already in use (shadcn/ui, TanStack Query, Zustand).

### Examples

Root application entry point:

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { App } from "./app.tsx";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

Component with typed props:

```tsx
interface UserCardProps {
  username: string;
  color: string;
}

export function UserCard({ username, color }: UserCardProps) {
  return (
    <div style={{ borderColor: color }}>
      <span>{username}</span>
    </div>
  );
}
```

### Consequences

* Good, because the largest frontend ecosystem — shadcn/ui, TanStack Query, Zustand, React Router all assume React
* Good, because first-class TypeScript support
* Good, because React 19 and the React Compiler reduce the need for manual memoisation
* Bad, because more verbose than Vue for simple components
* Bad, because React's rendering model requires understanding of hooks rules and the component lifecycle

### Confirmation

All UI code must use React. Component files use the `.tsx` extension. `StrictMode` must be enabled in the root entry point.

## Pros and Cons of the Options

### React

* Good, because largest ecosystem and community
* Good, because first-class TypeScript support
* Good, because foundation for all other UI libraries in use
* Bad, because more boilerplate than Vue for simple cases
* Bad, because hooks rules add cognitive overhead

### Vue

* Good, because approachable syntax with Single File Components
* Good, because strong TypeScript support in Vue 3
* Bad, because smaller ecosystem than React for the specific libraries in use
* Bad, because team familiarity is lower

### Angular

* Good, because opinionated, batteries-included framework — routing, forms, HTTP, DI all built in
* Good, because strong TypeScript support — Angular is TypeScript-first
* Bad, because steep learning curve — decorators, modules, and dependency injection add significant complexity
* Bad, because large bundle size and slower startup compared to React
* Bad, because ecosystem of React-specific libraries (shadcn/ui, TanStack Query, Zustand) is not compatible
