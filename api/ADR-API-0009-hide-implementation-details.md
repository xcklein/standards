---
status: accepted
date: 2026-10-31
tags: [api, contracts, security, schema]
---
# Hide Implementation Details

## Directive

External contracts (request and response bodies, error responses, headers, and URLs) and their documentation (OpenAPI descriptions and examples, published API docs, changelogs) must describe the API's domain, not how it is implemented. Every response body must come from an explicit contract schema that internal data is mapped into; database rows, ORM entities, and third-party SDK objects must never be returned directly. Contracts must not expose internal columns, database table or column names, third-party provider names or IDs, infrastructure names, stack traces, or raw exception messages.

## Context and Problem Statement

When an API returns its internal objects as-is, every internal change becomes a breaking change for clients: renaming a column, switching ORMs, or replacing a payment or auth provider changes the response. It also leaks information an attacker can use, such as table names from a database error, a provider's customer IDs, or a stack trace that reveals libraries and file paths. Mapping internal data into a contract schema keeps the contract stable while the implementation changes underneath it, and makes every field a client sees a deliberate choice.

## Decision Drivers

* Internal changes (schema, ORM, providers, infrastructure) must not force breaking API changes
* Every field in a response should be there on purpose, not because it happened to be on an internal object
* Error responses must not reveal database structure, file paths, library versions, or provider details
* Clients must not come to depend on details the API never meant to promise
* API documentation is part of the contract: a description that names a table or provider promises that detail as surely as a response field does, and goes stale when it changes

## Considered Options

* Explicit contract schemas, with internal data mapped into them
* Return internal objects, omitting sensitive fields case by case
* Return internal objects as-is

## Decision Outcome

Chosen option: "Explicit contract schemas, with internal data mapped into them", because it decouples the contract from the implementation and makes exposure opt-in rather than opt-out.

### What must not appear in a contract

| Detail | Example |
|---|---|
| Internal columns | `_created_at`, `_deleted_at` (see ADR-DB-0001) |
| Storage names | Table, column, index, or constraint names |
| Provider details | Stripe customer IDs, WorkOS user IDs, which provider is used |
| Infrastructure names | ARNs, bucket names, function names, internal hostnames |
| Error internals | Stack traces, raw exception messages, SQL errors |
| Server software | `X-Powered-By`, framework or library version headers |
| Documentation internals | OpenAPI descriptions or changelog entries that mention tables, providers, queues, or internal services ("Reads from the `users` table", "Calls WorkOS to …") |

A provider value may appear only when the client genuinely needs it to call that provider directly (e.g. a Stripe publishable key or client secret for checkout). In that case it is part of the contract on purpose and documented as such.

### Examples

Bad — returns the database row:

```typescript
app.openapi(getUser, async (c) => {
  const user = await db.query.users.findFirst({ where: eq(users.id, id) });
  return c.json(user); // exposes _deleted_at, workos_user_id, stripe_customer_id, ...
});
```

Good — maps into the contract schema:

```typescript
const UserResponse = z.object({
  id: z.string(),
  displayName: z.string(),
  joinedAt: z.string().datetime(),
});

function toUserResponse(user: UserRow): z.infer<typeof UserResponse> {
  return {
    id: user.id,
    displayName: user.display_name,
    joinedAt: user._created_at.toISOString(),
  };
}

app.openapi(getUser, async (c) => {
  const user = await findUser(id);
  return c.json(toUserResponse(user), 200);
});
```

Bad — forwards the raw error:

```json
{
  "type": "about:blank",
  "title": "Internal Server Error",
  "status": 500,
  "detail": "duplicate key value violates unique constraint \"users_email_key\""
}
```

Good — describes the problem for the client and logs the internal error instead:

```json
{
  "type": "https://api.example.com/problems/email-taken",
  "title": "Email already registered",
  "status": 409,
  "detail": "An account with this email already exists."
}
```

### Consequences

* Good, because internal schema, ORM, provider, and infrastructure changes do not break clients
* Good, because every field in a response is a deliberate choice
* Good, because error responses do not leak information useful to an attacker
* Bad, because every response type needs a mapping function, even when it looks like a copy of the internal type
* Bad, because a new internal field never reaches clients until someone adds it to the contract, which is the point but adds a step

### Confirmation

Code review must flag handlers that return database rows, ORM entities, or SDK objects directly, spread an internal object into a response, or put an exception's `message` or stack into an error response, and must flag OpenAPI descriptions, examples, or published docs that describe internals rather than behavior. API contract tests should validate responses against the OpenAPI schema with unknown properties disallowed, so a leaked field fails the test.

## Pros and Cons of the Options

### Explicit contract schemas

* Good, because the contract is independent of the implementation
* Good, because exposure is opt-in
* Bad, because more mapping code

### Return internal objects, omitting sensitive fields case by case

* Good, because less code
* Bad, because exposure is opt-out: any new internal field leaks until someone notices
* Bad, because the contract still changes whenever the internal type changes

### Return internal objects as-is

* Good, because no extra code
* Bad, because internal columns, provider IDs, and storage names leak to clients
* Bad, because every internal change is a breaking API change

## More Information

* Related: [ADR-API-0002 — OpenAPI Schema](ADR-API-0002-openapi-schema.md)
* Related: [ADR-API-0003 — Error Response Format](ADR-API-0003-error-response-format.md)
* Related: [ADR-API-0006 — Pagination](ADR-API-0006-pagination.md)
* Related: [ADR-DB-0001 — Internal Column Prefix](../database/ADR-DB-0001-internal-column-prefix.md)
* Related: [ADR-TS-0007 — Use Hono](../typescript/ADR-TS-0007-use-hono.md)
* Related: [ADR-UI-0010 — Hide Implementation Details from Users](../ui/ADR-UI-0010-hide-implementation-details.md)
