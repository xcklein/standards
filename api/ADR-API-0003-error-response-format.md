---
status: accepted
date: 2026-05-16
tags: [api, errors, rfc9457, http]
---
# Error Response Format

## Directive

All error responses must use RFC 9457 Problem Details format with `Content-Type: application/problem+json`. Every error response must include `type`, `title`, `status`, and `detail` fields. `type` URIs do not need to be HTTP-resolvable.

## Context and Problem Statement

APIs must return errors in a consistent, machine-readable format so that clients can handle failures predictably. Without a standard, teams invent their own error shapes, leading to inconsistency across services and increased client integration effort. RFC 9457 (which obsoletes RFC 7807) defines a standard "Problem Details" format for HTTP error responses that is widely understood and tooling-friendly.

## Decision Drivers

* Error responses must be consistent across all APIs
* Clients must be able to programmatically distinguish error types without parsing message strings
* The format must carry enough detail for developers to diagnose issues
* The format should follow an established standard rather than a bespoke convention

## Considered Options

* RFC 9457 Problem Details
* Bespoke JSON error envelope
* GraphQL-style error array

## Decision Outcome

Chosen option: "RFC 9457 Problem Details", because it is an IETF standard with broad tooling support that provides a structured, extensible error format without requiring custom conventions.

### Examples

Content-Type must be `application/problem+json`.

```json
{
  "type": "https://example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "The 'email' field must be a valid email address.",
  "instance": "/users/register"
}
```

Extended with custom fields for additional context:

```json
{
  "type": "https://example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "One or more fields failed validation.",
  "instance": "/users/register",
  "errors": [
    { "field": "email", "message": "Must be a valid email address." },
    { "field": "age", "message": "Must be a positive integer." }
  ]
}
```

### Consequences

* Good, because clients can rely on a consistent error shape across all endpoints
* Good, because `type` URI provides a stable, linkable identifier for each error class
* Good, because the format is extensible — additional fields can be added without breaking the standard
* Good, because many HTTP frameworks have built-in RFC 7807 support
* Bad, because `type` URIs must be maintained and documented per error class
* Bad, because teams unfamiliar with RFC 7807 need to learn the format

### Confirmation

All error responses must use `Content-Type: application/problem+json` and include at minimum `type`, `title`, `status`, and `detail` fields. Enforced via API contract tests and OpenAPI schema validation.

## Pros and Cons of the Options

### RFC 9457 Problem Details

* Good, because IETF standard with broad tooling and framework support
* Good, because extensible without breaking the base format
* Good, because `type` URI provides stable error class identifiers
* Bad, because requires maintaining a URI registry for error types

### Bespoke JSON error envelope

* Good, because full control over the shape
* Bad, because inconsistent across teams and projects without strict enforcement
* Bad, because clients must learn a custom format per API
* Bad, because no tooling support out of the box

### GraphQL-style error array

* Good, because familiar to GraphQL consumers
* Bad, because designed for GraphQL — maps poorly to REST HTTP semantics
* Bad, because loses the HTTP status code as the primary error signal

## More Information

* [RFC 9457 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457) (obsoletes RFC 7807)
