---
status: accepted
date: 2026-05-16
tags: [api, rest, architecture, http]
---
# Prefer REST APIs

## Directive

All new external-facing APIs must follow REST conventions over HTTP. Deviations (GraphQL, gRPC, tRPC) must be documented and justified in a service-level ADR.

## Context and Problem Statement

Multiple API paradigms exist for exposing services over HTTP. Without a clear default, teams make independent choices leading to inconsistent interfaces, fragmented tooling, and increased integration complexity across services. A preferred paradigm should be established so that teams start from a common baseline and only deviate when there is a clear, documented reason.

## Decision Drivers

* APIs must be interoperable across services regardless of implementation language
* The paradigm must have broad tooling support for documentation, testing, and client generation
* The default choice must be approachable for all developers regardless of experience level
* Deviations from the standard must be explicit and justified

## Considered Options

* REST over HTTP
* GraphQL
* gRPC
* tRPC

## Decision Outcome

Chosen option: "REST over HTTP", because it is the most widely understood API paradigm, requires no special client tooling, and integrates with the existing standards in this repository (OpenAPI, RFC 7807, JWT, cursor pagination).

### Examples

A well-structured REST API follows resource-oriented URLs, uses HTTP methods semantically, and returns standard HTTP status codes:

```
GET    /v1/users           → 200 list of users
POST   /v1/users           → 201 created user
GET    /v1/users/123       → 200 single user
PATCH  /v1/users/123       → 200 updated user
DELETE /v1/users/123       → 204 no content

GET    /v1/users/123/orders → 200 orders belonging to user 123
```

HTTP methods must be used according to their semantics:

| Method | Semantics | Idempotent |
|---|---|---|
| GET | Read, no side effects | yes |
| POST | Create or non-idempotent action | no |
| PUT | Full replace | yes |
| PATCH | Partial update | no |
| DELETE | Remove | yes |

### Consequences

* Good, because REST is universally understood and requires no special client libraries
* Good, because integrates directly with all other API standards in this repository
* Good, because HTTP tooling (curl, Postman, browsers) works without configuration
* Good, because OpenAPI provides full schema and documentation support for REST
* Bad, because REST can be chatty for complex data requirements — multiple round trips for related resources
* Bad, because REST has no built-in subscriptions or streaming — requires WebSocket or SSE for real-time use cases
* Neutral, because gRPC or tRPC may be more appropriate for internal service-to-service communication where performance is critical — such deviations must be documented in a service-level ADR

### Confirmation

All new external-facing APIs must be REST unless a service-level ADR documents and justifies the deviation. Internal service-to-service APIs may use gRPC where performance requirements justify it, subject to the same documentation requirement.

## Pros and Cons of the Options

### REST over HTTP

* Good, because universally understood — no client library required
* Good, because broad tooling support for documentation, testing, and code generation
* Good, because stateless and horizontally scalable by default
* Bad, because can require multiple requests to fetch related resources
* Bad, because no native streaming or subscription support

### GraphQL

* Good, because clients fetch exactly the data they need — reduces over- and under-fetching
* Good, because strongly typed schema with built-in introspection
* Bad, because requires a GraphQL client library — not usable with plain HTTP tools
* Bad, because caching is more complex — queries are POST requests, bypassing HTTP cache semantics
* Bad, because error handling does not follow HTTP conventions (200 responses with errors in the body)
* Bad, because OpenAPI and other REST tooling does not apply

### gRPC

* Good, because extremely high performance via HTTP/2 and binary Protobuf encoding
* Good, because strongly typed contracts via `.proto` files
* Bad, because requires a gRPC client — not usable from browsers without a proxy
* Bad, because binary protocol makes debugging harder than JSON over HTTP
* Bad, because tooling ecosystem is narrower than REST

### tRPC

* Good, because end-to-end type safety in TypeScript monorepos with zero schema duplication
* Good, because minimal boilerplate for internal APIs
* Bad, because TypeScript-only — not usable across language boundaries
* Bad, because no standard schema format for documentation or non-TypeScript clients

## More Information

* [RFC 7231 — HTTP/1.1 Semantics and Content](https://www.rfc-editor.org/rfc/rfc7231)
* Related: [ADR-API-0002 — Emit an OpenAPI Schema](ADR-API-0002-openapi-schema.md)
* Related: [ADR-API-0003 — Error Response Format](ADR-API-0003-error-response-format.md)
