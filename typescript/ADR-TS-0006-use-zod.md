---
status: accepted
date: 2026-05-16
tags: [typescript, validation, schema, tooling]
---
# Use Zod

## Directive

All runtime data validation must use Zod. TypeScript types at system boundaries must be derived via `z.infer` rather than declared independently. Joi and Yup must not be used.

## Context and Problem Statement

TypeScript's type system is erased at runtime, meaning that data arriving at system boundaries — API responses, environment variables, user input, configuration files — has no guaranteed shape. Runtime validation is required to ensure this data conforms to expected types before it is used. A schema library should bridge the gap between TypeScript types and runtime validation without requiring types to be declared twice.

## Decision Drivers

* Data at system boundaries must be validated at runtime, not just at compile time
* TypeScript types must be derived from schemas to eliminate duplication
* The library must be lightweight, tree-shakeable, and have no runtime dependencies
* Error messages must be actionable and easy to surface to callers

## Considered Options

* Zod
* Joi

## Decision Outcome

Chosen option: "Zod", because it provides a first-class TypeScript experience with automatic type inference, a chainable API, and no external dependencies.

### Examples

Define a schema and infer its TypeScript type:

```typescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  age: z.number().int().min(0),
});

type User = z.infer<typeof UserSchema>;
```

Parse and validate data at a boundary (throws on failure):

```typescript
const user = UserSchema.parse(rawJson);
```

Safe parse — returns a result object instead of throwing:

```typescript
const result = UserSchema.safeParse(rawJson);

if (!result.success) {
  console.error(result.error.flatten());
} else {
  const user = result.data;
}
```

Validate environment variables at startup:

```typescript
const EnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  PORT: z.coerce.number().default(3000),
});

export const env = EnvSchema.parse(process.env);
```

### Consequences

* Good, because TypeScript types are inferred directly from schemas — no duplication
* Good, because validation errors are structured and easy to map to user-facing messages
* Good, because zero external runtime dependencies
* Good, because `safeParse` enables error handling without try/catch
* Bad, because bundle size is larger than Valibot for projects where size is critical
* Bad, because complex discriminated union schemas can become verbose

### Confirmation

All data entering the system at boundaries (HTTP request bodies, environment variables, external API responses, config files) must be validated with a Zod schema. TypeScript types at boundaries must be derived via `z.infer` rather than declared independently.

## Pros and Cons of the Options

### Zod

* Good, because first-class TypeScript support with automatic type inference
* Good, because chainable, readable API
* Good, because zero external dependencies
* Good, because large ecosystem of integrations (tRPC, React Hook Form, etc.)
* Bad, because verbose for complex union types

### Joi

* Good, because mature and widely adopted, particularly in Node.js server-side contexts
* Good, because expressive and readable API
* Bad, because TypeScript support is a secondary concern — types are often inaccurate or require manual declaration
* Bad, because no automatic type inference from schemas — types must be declared separately
* Bad, because designed for JavaScript-first use, not TypeScript-first

## More Information

* [Zod documentation](https://zod.dev)
