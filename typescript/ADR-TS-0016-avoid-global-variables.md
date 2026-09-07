---
status: proposed
date: 2026-09-07
tags: [typescript, style, state-management]
---
# Avoid Global Variables

## Directive

Mutable module-level state (`let`/`var` bindings at module scope, or exported mutable singletons) must not be used. Module-scoped `const` bindings that are genuinely immutable (configuration values, lookup tables, compiled regexes, etc.) are fine. State that changes over time must be owned by and passed through an explicit carrier — a function parameter, a class instance, a dependency-injection container, or framework-level state (e.g. React state/context) — rather than read or written through a shared global binding.

## Context and Problem Statement

A mutable module-level variable is implicitly shared by every caller that imports the module, for the lifetime of the process. Any code anywhere can read or write it, so reasoning about its value at a given point requires tracing every import site rather than just the local call stack. This makes behavior order-dependent (whichever code runs first sets the state others observe), makes unit tests leak state into one another unless carefully reset, and makes concurrent or repeated invocations (e.g. serverless cold starts reused across requests, or parallel test runners) prone to cross-contamination bugs that are hard to reproduce.

## Decision Drivers

* State mutated from multiple call sites is hard to reason about without tracing every import
* Global mutable state leaks between unit tests unless manually reset in `beforeEach`/`afterEach`
* Shared mutable state is a correctness hazard when a module instance is reused across requests or run concurrently
* Immutable module-level constants (config values, lookup tables) carry none of these risks and are not restricted by this rule
* Explicit state carriers (parameters, class instances, DI, framework state) make dependencies visible in a function's signature instead of hidden in its body

## Considered Options

* No mutable module-level state — pass state explicitly
* Mutable module-level variables, disciplined by convention and code review
* Singleton classes exposing mutable static state

## Decision Outcome

Chosen option: "No mutable module-level state", because it makes every dependency a function or class has visible in its signature, eliminates cross-test and cross-request state leakage, and pushes lifetime/ownership decisions to call sites where they can be reasoned about locally.

### Examples

Bad — mutable module-level state:

```typescript
let currentUser: User | null = null;

export function setCurrentUser(user: User): void {
  currentUser = user;
}

export function getGreeting(): string {
  return `Hello, ${currentUser?.name ?? "guest"}`;
}
```

Good — state passed explicitly:

```typescript
export function getGreeting(user: User | null): string {
  return `Hello, ${user?.name ?? "guest"}`;
}
```

Good — state owned by a class instance instead of a module:

```typescript
export class Session {
  private currentUser: User | null = null;

  setCurrentUser(user: User): void {
    this.currentUser = user;
  }

  getGreeting(): string {
    return `Hello, ${this.currentUser?.name ?? "guest"}`;
  }
}
```

Fine — immutable module-level constant:

```typescript
export const MAX_RETRIES = 3;
export const STATUS_LABELS: Record<OrderStatus, string> = {
  pending: "Pending",
  processing: "Processing",
  shipped: "Shipped",
  delivered: "Delivered",
  cancelled: "Cancelled",
};
```

### Consequences

* Good, because a function's or class's dependencies are visible in its signature instead of hidden in module-level bindings
* Good, because unit tests do not need to reset shared state between runs
* Good, because behavior no longer depends on which code happened to run first and mutate the shared binding
* Good, because module instances are safe to reuse across requests or run concurrently without cross-contamination
* Bad, because explicit state carriers add parameters or constructor arguments that a global variable would have avoided
* Bad, because some state (e.g. a process-wide cache) is legitimately singleton in nature and requires a deliberately scoped container instead of a plain module variable

### Confirmation

Code review must flag any `let`/`var` declared at module scope, and any exported mutable object intended to be written from multiple call sites, asking for the state to be passed explicitly or owned by a class instance instead. A lint rule (e.g. `no-restricted-syntax` targeting a `VariableDeclaration` with `let`/`var` at the top level of a module) is recommended to catch this automatically.

## Pros and Cons of the Options

### No mutable module-level state

* Good, because dependencies are explicit in function signatures and constructors
* Good, because eliminates cross-test and cross-request state leakage
* Bad, because requires threading state through call chains that a global would have shortcut

### Mutable module-level variables, disciplined by convention and code review

* Good, because no restructuring needed — quickest to write
* Bad, because relies entirely on reviewer vigilance to catch, with no compiler or lint backing
* Bad, because a missed case reintroduces the exact bugs this rule exists to prevent

### Singleton classes exposing mutable static state

* Good, because groups related state and behavior together, better than a bare module variable
* Bad, because still process-wide shared mutable state — the visibility and test-isolation problems of a global variable remain, only relocated onto a class

## More Information

* Related: [ADR-TS-0015 — Defining Enums](ADR-TS-0015-defining-enums.md)
