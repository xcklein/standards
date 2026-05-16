---
status: accepted
date: 2026-05-16
tags: [api, versioning, http]
---
# Versioning Strategy

## Directive

All API routes must include a version prefix (`/v{n}/`). Deprecated versions must respond with `Deprecation` and `Sunset` headers.

## Context and Problem Statement

APIs evolve over time and breaking changes are sometimes unavoidable. A versioning strategy allows clients to depend on a stable API contract while new versions are developed in parallel. Without a consistent strategy, breaking changes are unpredictable and clients cannot safely upgrade on their own schedule.

## Decision Drivers

* Breaking changes must not affect existing clients without their consent
* The versioning scheme must be visible and explicit — not hidden in headers
* API versions must be easy to route, document, and deprecate independently
* The scheme must work with standard HTTP tooling, proxies, and gateways

## Considered Options

* URL path versioning (`/v1/users`)
* `Accept` header versioning (`Accept: application/vnd.api+json;version=1`)
* Custom request header (`API-Version: 1`)

## Decision Outcome

Chosen option: "URL path versioning", because the version is explicit in the URL, trivially routable, and visible in logs, browser tools, and documentation without additional configuration.

### Examples

Version prefix as the first path segment:

```
https://api.example.com/v1/users
https://api.example.com/v1/users/123
https://api.example.com/v2/users
```

Deprecation notice via response header (RFC 8594):

```http
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 31 Dec 2026 23:59:59 GMT
Link: <https://api.example.com/v2/users>; rel="successor-version"
```

### Consequences

* Good, because the version is visible in URLs, logs, and browser devtools without extra configuration
* Good, because routing by version is trivial in any API gateway or load balancer
* Good, because each version can be deployed, documented, and deprecated independently
* Bad, because URLs are not technically "pure" REST — the version is not a resource property
* Bad, because clients must update base URLs when upgrading versions

### Confirmation

All API routes must include a version prefix (`/v{n}/`). Deprecated versions must respond with `Deprecation` and `Sunset` headers. Enforced via API gateway routing rules and OpenAPI spec review.

## Pros and Cons of the Options

### URL path versioning

* Good, because version is explicit and visible everywhere
* Good, because trivially routable without inspecting headers
* Good, because easy to document and test per version
* Bad, because violates strict REST resource purity
* Bad, because base URL changes on upgrade

### `Accept` header versioning

* Good, because URLs remain stable across versions
* Good, because follows HTTP content negotiation semantics
* Bad, because version is invisible in URLs, logs, and browser tools
* Bad, because harder to route at the gateway layer
* Bad, because increases client complexity — must set headers correctly on every request

### Custom request header

* Good, because URLs remain stable
* Bad, because non-standard — no RFC backing
* Bad, because ignored by proxies and gateways that only inspect standard headers
* Bad, because invisible in URLs and difficult to test in a browser

## More Information

* [RFC 8594 — The Sunset HTTP Header Field](https://www.rfc-editor.org/rfc/rfc8594)
