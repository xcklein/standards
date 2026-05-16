---
status: accepted
date: 2026-05-16
tags: [api, openapi, documentation, tooling]
---
# Emit an OpenAPI Schema

## Directive

Every API must serve an OpenAPI 3.1 schema at a well-known path (e.g. `/openapi.json`). The schema must be validated with Spectral in CI.

## Context and Problem Statement

APIs without a machine-readable contract require consumers to rely on prose documentation, which drifts from the implementation over time. An OpenAPI schema serves as the single source of truth for an API's shape, enabling automatic client generation, contract testing, interactive documentation, and linting. All APIs must publish an OpenAPI schema to ensure consumers have an accurate, up-to-date contract.

## Decision Drivers

* API contracts must be machine-readable and kept in sync with the implementation
* Client SDKs and types must be generatable from the schema without manual effort
* Documentation must be automatically derivable from the schema
* Contract testing must be possible without relying on live environments

## Considered Options

* OpenAPI 3.1 (schema-first or code-first)
* No formal schema (prose documentation only)
* GraphQL schema

## Decision Outcome

Chosen option: "OpenAPI 3.1", because it is the industry standard for describing REST APIs, is supported by the broadest tooling ecosystem, and aligns with the JSON Schema specification for validation.

### Examples

The schema must be served at a well-known path. The exact route may vary by project (e.g. `/openapi.json`, `/api/openapi.json`), but it must be documented and consistent within the service.

Minimum viable OpenAPI 3.1 document structure:

```yaml
openapi: 3.1.0
info:
  title: Example API
  version: 1.0.0
paths:
  /v1/users:
    get:
      summary: List users
      parameters:
        - name: cursor
          in: query
          schema:
            type: string
      responses:
        '200':
          description: A paginated list of users
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'
        '401':
          $ref: '#/components/responses/Unauthorized'
components:
  schemas:
    UserListResponse:
      type: object
      required: [data, pagination]
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/Pagination'
```

### Consequences

* Good, because the schema is the authoritative contract — documentation and clients are always derived from it
* Good, because client SDKs can be generated in any language from the schema
* Good, because Spectral and similar tools can lint the schema for convention compliance
* Good, because interactive documentation (Scalar, Swagger UI) is available for free
* Bad, because maintaining a schema alongside the implementation requires discipline to avoid drift
* Bad, because code-first approaches can produce verbose or incomplete schemas if annotations are missed

### Confirmation

Every API must serve its schema at a documented, well-known path. CI validates the schema using Spectral with the configured ruleset. Schema drift is caught via contract tests that validate responses against the schema.

## Pros and Cons of the Options

### OpenAPI 3.1

* Good, because industry standard with the broadest tooling support
* Good, because fully aligned with JSON Schema for component validation
* Good, because enables client generation, contract testing, and linting from a single document
* Bad, because verbose for large APIs
* Bad, because code-first generation requires careful framework configuration to produce complete schemas

### No formal schema

* Good, because zero upfront effort
* Bad, because documentation drifts from the implementation immediately
* Bad, because consumers have no machine-readable contract to generate clients from
* Bad, because contract testing is not possible without a schema

### GraphQL schema

* Good, because introspection is built into the protocol
* Good, because strongly typed by default
* Bad, because only applicable to GraphQL APIs — not relevant for REST
* Bad, because a different paradigm requiring a full architectural commitment

## More Information

* [OpenAPI Specification 3.1.0](https://spec.openapis.org/oas/v3.1.0)
* [Spectral — OpenAPI linter](https://stoplight.io/open-source/spectral)
