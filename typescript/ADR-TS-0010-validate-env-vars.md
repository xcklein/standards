---
status: accepted
date: 2026-05-16
tags: [typescript, config, validation]
---
# Validate Environment Variables at Startup

## Directive

All environment variables must be parsed and validated at application startup using the standard runtime validation library (see ADR-TS-0006). A single typed `config` object must be exported from a dedicated module and used throughout the codebase. Direct access to `process.env` outside that module is not permitted.

## Context and Problem Statement

Applications depend on environment variables for configuration — API keys, database URLs, feature flags, and service endpoints. Without validation, missing or malformed variables surface as runtime errors deep in the application rather than at startup. TypeScript does not help here: `process.env` values are all `string | undefined`, so types give no indication of what is actually required.

## Decision Drivers

* Missing environment variables must cause the application to fail immediately at startup with a clear error message
* Config values must be typed — consumers must not cast or assert `process.env` values manually
* The same validation approach must be used across all services for consistency
* The solution must not introduce a new dependency beyond what ADR-TS-0006 already mandates

## Considered Options

* Runtime validation schema over `process.env` (per ADR-TS-0006)
* `envalid`
* `dotenv-safe`
* Manual validation (`if (!process.env.FOO) throw`)

## Decision Outcome

Chosen option: "Runtime validation schema over `process.env`", because it reuses the existing validation dependency, produces a fully typed config object, and surfaces all invalid variables at once on startup.

### Examples

Define and export a typed config object in a dedicated module:

```typescript
import { z } from "zod";

const schema = z.object({
  NODE_ENV: z.enum(["development", "test", "production"]),
  PORT: z.coerce.number().int().min(1).max(65535).default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  AWS_REGION: z.string().default("us-east-1"),
});

export const config = schema.parse(process.env);
export type Config = z.infer<typeof schema>;
```

If validation fails, all invalid variables are reported before the application starts:

```
ZodError: [
  { path: ["DATABASE_URL"], message: "Invalid url" },
  { path: ["JWT_SECRET"], message: "Required" }
]
```

Consume config throughout the application by importing the typed object — never `process.env` directly:

```typescript
import { config } from "@/config.js";

const server = createServer({ port: config.PORT });
```

### Consequences

* Good, because the application fails fast at startup with a clear, complete list of invalid variables
* Good, because `config` is fully typed — no casts or `!` assertions needed at the call site
* Good, because no new dependency beyond what ADR-TS-0006 already requires
* Bad, because all environment variables must be known at startup — dynamic config loaded later must be validated separately

### Confirmation

Direct `process.env` access outside the config module must be flagged in code review. A Biome lint rule or custom ESLint plugin can enforce this automatically if desired.

## Pros and Cons of the Options

### Runtime validation schema over `process.env`

* Good, because no additional dependency
* Good, because fully typed output
* Good, because consistent with the rest of the codebase
* Good, because supports coercion, defaults, and custom error messages

### `envalid`

* Good, because purpose-built for env var validation with clear output
* Bad, because an additional dependency that duplicates the existing validation library's capability
* Bad, because output type is less ergonomic than a plain inferred schema type

### `dotenv-safe`

* Good, because enforces that all variables defined in `.env.example` are present
* Bad, because only checks presence — no type coercion or format validation
* Bad, because requires maintaining a separate `.env.example` file

### Manual validation

* Good, because no dependency
* Bad, because requires writing repetitive guard clauses for every variable
* Bad, because errors surface one at a time rather than all at once
* Bad, because no typed output without additional casting

## More Information

* Related: [ADR-TS-0006 — Use Zod](ADR-TS-0006-use-zod.md)
