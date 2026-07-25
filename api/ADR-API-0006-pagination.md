---
status: accepted
date: 2026-05-16
tags: [api, pagination, http]
---
# Pagination Pattern

## Directive

All collection endpoints must use cursor-based pagination. Offset-based pagination is not permitted. Responses must include a `pagination` object and `Link` headers. Field casing within that object is not prescribed here — follow [ADR-API-0007](ADR-API-0007-naming-conventions.md).

## Context and Problem Statement

APIs returning collections must paginate results to avoid unbounded response sizes. The choice of pagination pattern affects performance, consistency guarantees, and client implementation complexity. Different patterns have different trade-offs depending on whether the underlying data changes frequently and whether clients need to seek to arbitrary positions.

## Decision Drivers

* Pagination must handle large collections without degrading performance
* Results must be stable — inserting or deleting records must not cause items to be skipped or duplicated across pages
* The pattern must be implementable efficiently on the database layer
* Clients must be able to discover next and previous pages without constructing URLs manually

## Considered Options

* Cursor-based pagination
* Offset/limit pagination
* Page number pagination

## Decision Outcome

Chosen option: "Cursor-based pagination", because it provides stable results regardless of insertions or deletions and performs consistently on large datasets using index scans rather than offset skips.

### Examples

Response envelope with pagination metadata. Field casing here follows [ADR-API-0007](ADR-API-0007-naming-conventions.md) (camelCase); this ADR does not prescribe its own casing:

```json
{
  "data": [
    { "id": "01HXYZ", "name": "Alice" },
    { "id": "01HABC", "name": "Bob" }
  ],
  "pagination": {
    "nextCursor": "01HABC",
    "hasNext": true,
    "hasPrevious": true,
    "prevCursor": "01HXYZ"
  }
}
```

Request using a cursor:

```
GET /v1/users?cursor=01HABC&limit=20
```

Link header for navigation (RFC 8288):

```http
Link: <https://api.example.com/v1/users?cursor=01HABC&limit=20>; rel="next"
Link: <https://api.example.com/v1/users?cursor=01HXYZ&limit=20>; rel="prev"
```

### Consequences

* Good, because results are stable — insertions and deletions do not affect in-progress pagination
* Good, because consistent performance on large datasets — no `OFFSET` scan
* Good, because cursors are opaque to clients, allowing the implementation to change freely
* Bad, because clients cannot seek to an arbitrary page — must paginate sequentially
* Bad, because total item count is expensive to compute and is omitted by default

### Confirmation

All collection endpoints must return a `pagination` object and `Link` headers. Offset-based pagination is not permitted. Enforced via OpenAPI schema and API contract tests.

## Pros and Cons of the Options

### Cursor-based pagination

* Good, because stable results regardless of concurrent writes
* Good, because constant-time performance on large datasets
* Good, because cursor is opaque — implementation details are hidden from clients
* Bad, because no random access — clients must paginate sequentially
* Bad, because total count requires a separate expensive query

### Offset/limit pagination

* Good, because supports random access — clients can jump to any page
* Good, because easy to implement and understand
* Bad, because unstable — inserts or deletes shift items between pages mid-pagination
* Bad, because performance degrades on large offsets (`OFFSET 100000` scans 100,000 rows)

### Page number pagination

* Good, because intuitive for human-facing UIs
* Bad, because same stability and performance problems as offset pagination
* Bad, because page numbers become invalid when total count changes

## More Information

* [RFC 8288 — Web Linking](https://www.rfc-editor.org/rfc/rfc8288)
