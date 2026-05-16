---
status: accepted
date: 2026-05-16
tags: [api, naming, conventions, http, json]
---
# Naming Conventions

## Directive

All URL paths must use kebab-case with plural nouns. All JSON request and response fields must use camelCase. Query parameter names must use camelCase.

## Context and Problem Statement

Inconsistent naming across API URLs and JSON payloads creates friction for consumers and makes APIs harder to use predictably. Decisions about URL casing, resource naming, and JSON field casing must be made explicitly and applied uniformly so that clients can infer conventions rather than consulting documentation for every endpoint.

## Decision Drivers

* URL and field naming must be predictable and consistent across all endpoints
* Conventions must align with established HTTP and JSON community norms
* Naming must be language-agnostic — clients in any language should find the conventions natural

## Considered Options

* kebab-case URLs, camelCase JSON (industry standard REST convention)
* snake_case URLs and JSON
* camelCase URLs and JSON

## Decision Outcome

Chosen option: "kebab-case URLs, camelCase JSON", because kebab-case is the standard for URL paths and camelCase is the standard for JSON in browser and JavaScript ecosystems, making both immediately familiar to API consumers.

### Examples

URL paths use kebab-case, plural nouns for collections:

```
GET  /v1/users
GET  /v1/users/123
GET  /v1/users/123/payment-methods
POST /v1/audit-logs
```

JSON request and response bodies use camelCase:

```json
{
  "userId": "01HXYZ",
  "firstName": "Alice",
  "emailAddress": "alice@example.com",
  "createdAt": "2026-05-16T09:00:00Z"
}
```

Query parameter names use camelCase:

```
GET /v1/users?pageSize=20&sortBy=createdAt
```

### Consequences

* Good, because kebab-case URLs are consistent with web conventions and readable in browsers
* Good, because camelCase JSON maps directly to JavaScript and TypeScript object properties without transformation
* Good, because plural nouns for collections follow REST resource naming best practices
* Bad, because server-side languages that use snake_case (Python, Ruby) require a transformation layer for JSON serialisation

### Confirmation

Enforced via OpenAPI linting (e.g., Spectral rules for URL casing and JSON field casing). Reviewed in API design review before implementation.

## Pros and Cons of the Options

### kebab-case URLs, camelCase JSON

* Good, because matches the dominant convention for REST APIs targeting browser clients
* Good, because camelCase JSON requires no transformation in JavaScript/TypeScript
* Good, because kebab-case URLs are human-readable and consistent with web conventions
* Bad, because snake_case server languages need a serialisation transform for JSON

### snake_case URLs and JSON

* Good, because consistent across URLs and JSON
* Good, because natural for Python and Ruby server-side code
* Bad, because snake_case URLs are uncommon in REST APIs and feel inconsistent with web norms
* Bad, because JavaScript clients must transform snake_case to camelCase or live with inconsistency

### camelCase URLs and JSON

* Good, because consistent across URLs and JSON
* Bad, because camelCase URLs are non-standard and awkward to read
* Bad, because URL case-sensitivity rules make camelCase error-prone

## More Information

* [Google JSON Style Guide](https://google.github.io/styleguide/jsoncstyleguide.xml)
