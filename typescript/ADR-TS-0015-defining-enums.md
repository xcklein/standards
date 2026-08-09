---
status: accepted
date: 2026-08-08
tags: [typescript, style, enums]
---
# Defining Enums

## Directive

Enums must be defined as a `const` object with `as const`, paired with a derived type of the same name using `(typeof X)[keyof typeof X]`. TypeScript's `enum` and `const enum` keywords must not be used.

```typescript
export const OrderStatus = {
  PENDING: "pending",
  PROCESSING: "processing",
  SHIPPED: "shipped",
  DELIVERED: "delivered",
  CANCELLED: "cancelled",
} as const;
export type OrderStatus = (typeof OrderStatus)[keyof typeof OrderStatus];
```

## Context and Problem Statement

TypeScript's `enum` keyword compiles to a runtime object with non-obvious semantics (numeric enums are bidirectionally mapped, `const enum` requires callers to share the same compiler settings and cannot be used with isolated file compilation) and is not erasable — code using `enum` cannot be compiled by tools that only strip types (`erasableSyntaxOnly`, isolated declarations, or transpiler-only pipelines like esbuild/swc without a separate type-stripping pass). A `const` object with `as const` produces exactly the runtime shape you write, requires no special compiler support, and pairs with a mapped type to give the same "named set of values" ergonomics as `enum` without any of its runtime surprises.

## Decision Drivers

* Code must compile under `erasableSyntaxOnly` and similar type-stripping-only pipelines
* The runtime value of an enum-like object should be exactly what is written, with no compiler-generated bidirectional mappings
* Enum values must still be usable both as a type (for annotations) and as a value (for iteration, lookup, and passing around)
* `const enum` must not be used, since it requires every consumer to share the same compiler configuration and cannot be used across isolated module compilation

## Considered Options

* `const` object with `as const` + derived mapped type
* TypeScript `enum`
* TypeScript `const enum`
* Plain union of string literal types with no runtime object

## Decision Outcome

Chosen option: "`const` object with `as const` + derived mapped type", because it compiles under erasable-syntax-only pipelines, produces a runtime object with no compiler-generated surprises, and still gives both a type and an iterable/lookupable runtime value from a single declaration.

### Examples

```typescript
export const OrderStatus = {
  PENDING: "pending",
  PROCESSING: "processing",
  SHIPPED: "shipped",
  DELIVERED: "delivered",
  CANCELLED: "cancelled",
} as const;
export type OrderStatus = (typeof OrderStatus)[keyof typeof OrderStatus];
```

Using it as a type annotation:

```typescript
function transition(order: Order, status: OrderStatus): void { ... }

transition(order, OrderStatus.SHIPPED); // ok
transition(order, "shipped");           // ok — the type is a union of the literal values
transition(order, "bogus");             // type error
```

Iterating over all values, e.g. for validation or a `z.enum`:

```typescript
const values = Object.values(OrderStatus);
// ["pending", "processing", "shipped", "delivered", "cancelled"]
```

Not supported:

```typescript
// Do not do this — not erasable, bidirectional numeric mapping, requires
// shared compiler config for `const enum`.
enum OrderStatus {
  Pending,
  Processing,
  Shipped,
}
```

### Consequences

* Good, because it compiles under `erasableSyntaxOnly` and other type-stripping-only pipelines
* Good, because the runtime object is exactly what is written — no compiler-generated reverse mapping
* Good, because the same declaration provides both a type (for annotations) and a value (for iteration and lookup)
* Good, because it composes naturally with libraries that expect a plain object or array of values (e.g. `z.enum(Object.values(X))`)
* Good, because individual values can be strongly referenced by name (`OrderStatus.SHIPPED`) anywhere a value is needed — not just when annotating a field's type — with full autocomplete and refactor-safety
* Bad, because it requires two lines (the object and the derived type) instead of one `enum` declaration
* Bad, because nothing prevents accidentally mutating the object at runtime if `as const` is omitted — `as const` is required, not optional

### Confirmation

Code review must flag any use of TypeScript's `enum` or `const enum` keywords and request the `const` object pattern instead. A lint rule (e.g. `no-restricted-syntax` targeting `TSEnumDeclaration`) is recommended to catch this automatically.

## Pros and Cons of the Options

### `const` object with `as const` + derived mapped type

* Good, because erasable — compiles under type-stripping-only pipelines
* Good, because runtime shape is exactly what is written, no bidirectional mapping
* Good, because one declaration provides both type and value
* Good, because individual values can be strongly referenced by name outside of type annotations, e.g. passed as arguments or compared directly (`status === OrderStatus.SHIPPED`)
* Bad, because slightly more verbose than a single `enum` declaration

### TypeScript `enum`

* Good, because a single, familiar declaration provides both type and value
* Bad, because not erasable — cannot compile under type-stripping-only pipelines
* Bad, because numeric enums generate a confusing bidirectional runtime mapping

### TypeScript `const enum`

* Good, because inlined at compile time, no runtime object at all
* Bad, because requires every consumer to share the same compiler configuration
* Bad, because incompatible with isolated module compilation (each file compiled independently, as most modern transpilers do)

### Plain union of string literal types, no runtime object

* Good, because simplest possible type-only definition, fully erasable
* Bad, because individual values cannot be strongly referenced by name — there is no `OrderStatus.SHIPPED` to reach for, only the raw string literal `"shipped"` typed out by hand at every use site, with no autocomplete and no refactor-safety if a value is ever renamed
* Bad, because there is no runtime value to iterate, validate against, or pass around without re-declaring the same literals elsewhere
* Bad, because the set of valid values exists only in type-space, invisible at runtime and to non-TypeScript consumers

## More Information

* [TypeScript Handbook — `erasableSyntaxOnly`](https://www.typescriptlang.org/tsconfig/#erasableSyntaxOnly)
