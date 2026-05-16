---
status: accepted
date: 2026-05-16
tags: [api, authentication, jwt, oauth2, security]
---
# Authentication Scheme

## Directive

All protected API endpoints must require a JWT bearer token issued via OAuth 2.0. Tokens must be signed with RS256 and verified against the issuer's public key. Expiry must be enforced.

## Context and Problem Statement

APIs must authenticate callers to protect resources and enforce access control. The choice of authentication scheme affects security, scalability, developer experience, and integration complexity. A consistent approach across all APIs reduces the surface area for misconfiguration and simplifies client development.

## Decision Drivers

* Authentication must be stateless to support horizontally scaled services
* The scheme must be interoperable with standard HTTP clients and tooling
* Tokens must be verifiable without a database lookup on every request
* The scheme must support delegated access for third-party integrations

## Considered Options

* JWT (RFC 7519) issued via OAuth 2.0 (RFC 6749)
* Opaque session tokens (server-side sessions)
* API keys

## Decision Outcome

Chosen option: "JWT via OAuth 2.0", because it is stateless, standardised, and supports delegated access through established grant flows without requiring shared session state.

### Examples

Bearer token in the `Authorization` header (RFC 6750):

```http
GET /api/users/me HTTP/1.1
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

Minimum required JWT claims:

```json
{
  "sub": "user_01HXYZ",
  "iss": "https://auth.example.com",
  "aud": "https://api.example.com",
  "exp": 1716000000,
  "iat": 1715996400
}
```

Tokens must be signed with RS256 (asymmetric) so services can verify without sharing a secret.

### Consequences

* Good, because stateless verification — no database lookup required per request
* Good, because claims carry identity and authorisation data directly in the token
* Good, because OAuth 2.0 supports machine-to-machine, user-delegated, and service account flows
* Bad, because JWTs cannot be revoked before expiry without a token blocklist
* Bad, because tokens grow large with many claims — keep payloads minimal
* Bad, because key rotation requires coordination across all verifying services

### Confirmation

All protected endpoints must require a valid `Authorization: Bearer <jwt>` header. Tokens must be verified against the issuer's public key. Expiry must be enforced. Reviewed in security audits and API contract tests.

## Pros and Cons of the Options

### JWT via OAuth 2.0

* Good, because stateless — scales horizontally without shared session state
* Good, because IETF standard with broad library and tooling support
* Good, because supports multiple grant flows for different client types
* Bad, because tokens are irrevocable until expiry without additional infrastructure
* Bad, because misconfigured JWT validation (e.g., accepting `alg: none`) is a critical vulnerability

### Opaque session tokens

* Good, because instantly revocable by deleting the session
* Good, because simple to implement
* Bad, because requires shared session storage (database or cache) — adds infrastructure dependency
* Bad, because does not scale horizontally without sticky sessions or a distributed store
* Bad, because not suitable for service-to-service authentication

### API keys

* Good, because simple to issue and use
* Good, because suitable for server-to-server integrations
* Bad, because no standard expiry or rotation mechanism
* Bad, because keys are long-lived secrets — compromise requires manual rotation
* Bad, because no standard for scoping or delegated access

## More Information

* [RFC 7519 — JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519)
* [RFC 6749 — OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749)
* [RFC 6750 — Bearer Token Usage](https://www.rfc-editor.org/rfc/rfc6750)
