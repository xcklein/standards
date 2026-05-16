---
status: accepted
date: 2026-05-16
tags: [typescript, openapi, api, codegen, tooling]
---
# Use openapi-typescript

## Directive

API client types must be generated from the server's OpenAPI schema using `openapi-typescript`. A `gen:api` script must be present in `package.json`. The generated file must be committed to the repository.

## Context and Problem Statement

Consuming a REST API from a TypeScript client requires type definitions for all request parameters and response shapes. Writing and maintaining these types by hand is error-prone and drifts from the actual API over time. `openapi-typescript` generates TypeScript types directly from an OpenAPI schema, ensuring the client types are always in sync with the server contract.

## Decision Drivers

* Client-side API types must be derived from the server's OpenAPI schema, not written manually
* Type generation must be automatable as part of the development workflow
* Generated types must be usable with a type-safe HTTP client without additional wiring

## Considered Options

* openapi-typescript
* Manually maintained type definitions
* OpenAPI Generator (Java-based, generates full client SDKs)

## Decision Outcome

Chosen option: "openapi-typescript", because it generates lightweight TypeScript type definitions directly from an OpenAPI schema with zero runtime overhead, and integrates seamlessly with `openapi-fetch`.

### Examples

Generate types from a local or remote OpenAPI schema:

```bash
# From a running local server
pnpx openapi-typescript http://localhost:3000/openapi.json -o ./src/lib/api.ts

# From a file
pnpx openapi-typescript ./openapi.json -o ./src/lib/api.ts
```

Add as a `package.json` script:

```json
{
  "scripts": {
    "gen:api": "pnpx openapi-typescript http://localhost:3000/openapi.json -o ./src/lib/api.ts"
  }
}
```

The generated file exports `paths`, `components`, and `operations` interfaces that describe the full API surface. These are consumed by `openapi-fetch` for type-safe requests (see ADR-TS-0009).

### Consequences

* Good, because API types are always derived from the server contract — drift is impossible if generation is run regularly
* Good, because zero runtime overhead — types are erased at compile time
* Good, because the generated file is a single TypeScript file that can be committed and reviewed like any other source file
* Bad, because types must be regenerated whenever the API changes — stale generated types can cause false confidence if regeneration is skipped
* Bad, because the API server must be running (or a schema file available) to regenerate types

### Confirmation

A `gen:api` script must be present in `package.json`. The generated `api.ts` file must be committed to the repository. Types must be regenerated when the API schema changes.

## Pros and Cons of the Options

### openapi-typescript

* Good, because lightweight — generates types only, no runtime client code
* Good, because zero dependencies in the output file
* Good, because integrates directly with `openapi-fetch`
* Bad, because requires regeneration when the API changes

### Manually maintained type definitions

* Good, because no tooling required
* Bad, because drifts from the API immediately — requires discipline to keep in sync
* Bad, because duplicates work already done by the server's OpenAPI schema

### OpenAPI Generator

* Good, because generates full client SDKs in many languages
* Bad, because generates large amounts of boilerplate code with runtime dependencies
* Bad, because requires Java and is slow to run
* Bad, because generated code is often difficult to customise or extend

## More Information

* [openapi-typescript on GitHub](https://github.com/openapi-ts/openapi-typescript)
* Related: [ADR-API-0002 — Emit an OpenAPI Schema](../api/ADR-API-0002-openapi-schema.md)
* Related: [ADR-TS-0009 — Use openapi-fetch](ADR-TS-0009-use-openapi-fetch.md)
