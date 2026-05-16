---
status: accepted
date: 2026-05-16
tags: [typescript, http, framework, openapi]
---
# Use Hono

## Directive

All HTTP APIs must be built with Hono using `@hono/zod-openapi`. Routes must be defined with `createRoute` and registered via `app.openapi`. Plain `app.get` / `app.post` may be used where `app.openapi` is not applicable (e.g., serving generated documentation).

## Context and Problem Statement

TypeScript APIs require an HTTP framework to handle routing, middleware, and request/response lifecycle. The framework must integrate with the existing toolchain — particularly Zod for validation and OpenAPI for schema generation — without requiring separate wiring for each concern. A framework that unifies routing, validation, and OpenAPI definition in a single declaration reduces boilerplate and eliminates the risk of the implementation drifting from the published schema.

## Decision Drivers

* Routes must be type-safe end-to-end — request inputs and response shapes must be checked at compile time
* OpenAPI schemas must be generated from the route definition, not maintained separately
* The framework must be runtime-agnostic and deployable to AWS Lambda without a compatibility layer
* Middleware must be composable and typed

## Considered Options

* Hono with `@hono/zod-openapi`
* Express
* Fastify

## Decision Outcome

Chosen option: "Hono with `@hono/zod-openapi`", because it is runtime-agnostic, provides end-to-end type safety from route definition through handler, and generates OpenAPI schemas directly from Zod schemas with no duplication.

### Examples

Each route is defined in its own file, exporting a `route` (the OpenAPI definition) and a `handler` (the implementation):

```typescript
import { createRoute, z } from "@hono/zod-openapi";
import type { ApiRouteHandler } from "../../lib/routing.ts";

const ParamSchema = z.object({ userId: z.string().nonempty() });
const ResponseSchema = z.object({
  id: z.string(),
  username: z.string(),
});

export const route = createRoute({
  method: "get",
  path: "/users/{userId}",
  summary: "Get a user",
  tags: ["User"],
  request: { params: ParamSchema },
  responses: {
    200: {
      content: { "application/json": { schema: ResponseSchema } },
      description: "Success.",
    },
  },
});

export const handler: ApiRouteHandler<typeof route> = async (ctx) => {
  const { userId } = ctx.req.valid("param");
  // ...
  return ctx.json({ id: userId, username: "alice" }, 200);
};
```

Routes are registered on an `OpenAPIHono` app instance:

```typescript
import { OpenAPIHono } from "@hono/zod-openapi";
import * as users_id_get from "./users.id.get.ts";

export const app = new OpenAPIHono<ApiEnv>();
app.openapi(users_id_get.route, users_id_get.handler);
```

### Consequences

* Good, because route definition, validation, and OpenAPI schema are colocated in a single declaration
* Good, because the handler is fully typed against the route — incorrect response shapes are compile errors
* Good, because Hono runs natively on AWS Lambda, Cloudflare Workers, Bun, and Node.js without adapters
* Good, because middleware is typed via the `ApiEnv` generic, making context variables type-safe
* Bad, because `@hono/zod-openapi` adds ceremony compared to plain Hono routing — every route requires explicit request/response schema declarations
* Bad, because the ecosystem is smaller than Express or Fastify

### Confirmation

All HTTP routes must be defined using `createRoute` from `@hono/zod-openapi` and registered via `app.openapi`. Plain `app.get` / `app.post` are permitted where `app.openapi` is not applicable (e.g., serving generated documentation).

## Pros and Cons of the Options

### Hono with `@hono/zod-openapi`

* Good, because runtime-agnostic — runs on Lambda, Workers, Node.js, Bun without changes
* Good, because end-to-end type safety from route definition through handler
* Good, because OpenAPI schema is generated from Zod schemas — no drift possible
* Bad, because smaller ecosystem than Express or Fastify
* Bad, because route definitions are more verbose than plain routing

### Express

* Good, because the most widely known Node.js framework — minimal learning curve
* Good, because enormous ecosystem of middleware and plugins
* Bad, because no built-in TypeScript support — requires manual typing
* Bad, because no native OpenAPI integration — schema must be maintained separately
* Bad, because tied to Node.js — not deployable to edge or Workers runtimes

### Fastify

* Good, because fast and production-proven
* Good, because JSON Schema validation built in
* Bad, because JSON Schema is more verbose and less ergonomic than Zod
* Bad, because OpenAPI integration requires additional plugins and configuration
* Bad, because not runtime-agnostic — Node.js only

## More Information

* [Hono documentation](https://hono.dev)
* [@hono/zod-openapi](https://github.com/honojs/middleware/tree/main/packages/zod-openapi)
* Related: [ADR-TS-0006 — Use Zod](ADR-TS-0006-use-zod.md)
* Related: [ADR-API-0002 — Emit an OpenAPI Schema](../api/ADR-API-0002-openapi-schema.md)
