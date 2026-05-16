---
status: accepted
date: 2026-05-16
tags: [api, content-negotiation, http, json]
---
# Content Negotiation

## Directive

All API responses must use `Content-Type: application/json`. Error responses must use `Content-Type: application/problem+json`. File upload endpoints are the only exception and must use `multipart/form-data`.

## Context and Problem Statement

HTTP supports content negotiation, allowing clients and servers to agree on the format of request and response bodies. In practice, most APIs serve a single format. An explicit decision on media type avoids ambiguity, ensures consistent `Content-Type` headers, and establishes a clear contract for clients about what to send and expect.

## Decision Drivers

* All APIs must use a consistent, explicit media type for requests and responses
* The media type must be standard and supported by all HTTP clients and tooling
* The format must be human-readable for debugging and tooling-friendly for parsing

## Considered Options

* `application/json`
* `application/vnd.api+json` (JSON:API)
* `application/xml`
* `multipart/form-data` (for file uploads only)

## Decision Outcome

Chosen option: "`application/json`", because it is the universal standard for HTTP APIs, supported by every HTTP client and framework without additional configuration.

### Examples

All requests with a body must set `Content-Type`:

```http
POST /v1/users HTTP/1.1
Content-Type: application/json
Accept: application/json

{
  "email": "alice@example.com",
  "firstName": "Alice"
}
```

All responses with a body must set `Content-Type`:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "01HXYZ",
  "email": "alice@example.com",
  "firstName": "Alice",
  "createdAt": "2026-05-16T09:00:00Z"
}
```

File upload endpoints are the only exception — use `multipart/form-data`:

```http
POST /v1/documents HTTP/1.1
Content-Type: multipart/form-data; boundary=----FormBoundary
```

Error responses use `application/problem+json` (see ADR-API-0001).

### Consequences

* Good, because `application/json` is universally supported with zero additional configuration
* Good, because JSON is human-readable and supported by all debugging and testing tools
* Good, because no structural envelope is imposed — response shape is fully controlled by the API
* Bad, because JSON has no built-in schema enforcement at the transport layer — validation must be done in application code

### Confirmation

All non-file API responses must use `Content-Type: application/json` (or `application/problem+json` for errors). Enforced via OpenAPI schema validation and API contract tests.

## Pros and Cons of the Options

### `application/json`

* Good, because universal support across all HTTP clients, frameworks, and tooling
* Good, because human-readable and debuggable
* Good, because no structural constraints imposed on the response body
* Bad, because no transport-level schema enforcement

### `application/vnd.api+json` (JSON:API)

* Good, because standardised envelope for resources, relationships, and links
* Good, because reduces bikeshedding on response structure
* Bad, because imposes a rigid envelope that increases payload verbosity
* Bad, because most clients require a dedicated JSON:API library to consume correctly
* Bad, because the specification is complex and rarely followed completely

### `application/xml`

* Good, because supports namespaces and schema validation via XSD
* Bad, because verbose and harder to read than JSON
* Bad, because poor support in modern JavaScript/TypeScript clients
* Bad, because largely superseded by JSON for HTTP APIs

### `multipart/form-data`

* Good, because required for file uploads
* Neutral, because only applicable to endpoints that accept file data — not a general choice

## More Information

* [RFC 7231 — HTTP/1.1 Semantics and Content](https://www.rfc-editor.org/rfc/rfc7231)
* Related: [ADR-API-0003 — Error Response Format](ADR-API-0003-error-response-format.md)
