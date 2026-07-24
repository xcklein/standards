---
status: accepted
date: 2026-05-16
tags: [typescript, logging, observability]
---
# Structured JSON Logging

## Directive

All application logs must be emitted as structured JSON. Every log entry must include `level`, `message`, `timestamp`, and `service`. `console.log`, `console.error`, and related methods must not be used in application code.

## Context and Problem Statement

Applications running in cloud environments emit logs that are collected by log aggregation systems (CloudWatch, Datadog, etc.). Plain text logs are difficult to query, filter, and alert on. Structured JSON logs are machine-readable: every field is indexable, log levels are filterable, and correlated request traces are possible. Without a standard, services emit inconsistent formats that require per-service parsing rules.

## Decision Drivers

* Logs must be queryable and filterable in log aggregation tools without custom parsers
* Log levels must be consistent across services
* Every log entry must be attributable to a specific service
* The standard must not mandate a specific library — teams may use pino, winston, or another structured logger

## Considered Options

* Structured JSON logging (library of choice)
* Plain text logging (`console.log`)
* No logging standard

## Decision Outcome

Chosen option: "Structured JSON logging", because structured logs are machine-readable, aggregation-friendly, and the de facto standard for cloud-native applications. Library choice is left to the service — the standard governs shape, not implementation.

### Examples

Required shape of every log entry:

```json
{
  "level": "info",
  "message": "Request received",
  "timestamp": "2026-05-16T12:00:00.000Z",
  "service": "olatile-api"
}
```

Additional fields are allowed and encouraged for context:

```json
{
  "level": "error",
  "message": "Database query failed",
  "timestamp": "2026-05-16T12:00:01.234Z",
  "service": "olatile-api",
  "requestId": "abc-123",
  "query": "SELECT * FROM users",
  "err": {
    "message": "connection timeout",
    "stack": "..."
  }
}
```

A thin logger interface that any library can satisfy:

```typescript
export interface Logger {
  info(msg: string, context?: Record<string, unknown>): void;
  warn(msg: string, context?: Record<string, unknown>): void;
  error(msg: string, context?: Record<string, unknown>): void;
  debug(msg: string, context?: Record<string, unknown>): void;
}
```

Initialise the logger with the service name from config:

```typescript
import { config } from "@/config.js";

// Example using pino — swap for any library that satisfies Logger
import pino from "pino";
export const logger: Logger = pino({ name: config.SERVICE_NAME });
```

### Consequences

* Good, because logs are queryable by any field in aggregation tools without custom parsers
* Good, because log levels are consistent and filterable across all services
* Good, because the `service` field makes multi-service log streams trivially filterable
* Good, because library-agnostic — teams pick the logger that fits their performance and feature needs
* Bad, because JSON logs are harder to read locally in a terminal (mitigate with pretty-print in development)
* Bad, because additional context fields require discipline — undocumented ad-hoc fields accumulate over time

### Confirmation

`console.log`, `console.error`, `console.warn`, and `console.info` must not appear in application code. ESLint's `no-console` rule enforces this automatically when enabled.

## Pros and Cons of the Options

### Structured JSON logging

* Good, because machine-readable and aggregation-friendly
* Good, because consistent, queryable schema across services
* Good, because library-agnostic — implementation flexibility retained
* Bad, because verbose output in development without a pretty-printer

### Plain text logging (`console.log`)

* Good, because zero setup — works everywhere
* Bad, because unstructured — aggregation tools cannot parse arbitrary text
* Bad, because no log levels, no service attribution, no queryable fields
* Bad, because not suitable for production observability

### No logging standard

* Good, because maximum flexibility per service
* Bad, because inconsistent formats require per-service parsing configuration
* Bad, because onboarding engineers must learn each service's logging conventions separately

## More Information

* Related: [ADR-TS-0010 — Validate Environment Variables at Startup](ADR-TS-0010-validate-env-vars.md)
* [pino documentation](https://getpino.io)
* [winston documentation](https://github.com/winstonjs/winston)
