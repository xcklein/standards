---
status: accepted
date: 2026-05-16
tags: [database, conventions, schema]
---
# Prefix Internal Columns with `_`

## Directive

All internal database columns (lifecycle timestamps, soft-delete markers) must be prefixed with `_`. Every table must include `_created_at`, `_updated_at`, and `_deleted_at`.

## Context and Problem Statement

Database tables contain two distinct categories of columns: business columns that represent domain data, and internal columns that represent infrastructure concerns such as lifecycle timestamps and soft-delete markers. Without a naming convention to distinguish them, internal columns are visually indistinct from business columns, making schemas harder to read and increasing the risk of accidentally exposing infrastructure fields in API responses.

## Decision Drivers

* Business columns and internal columns must be visually distinct at a glance
* Internal columns must be easy to exclude from API responses and DTO mappings
* The convention must be consistent across all tables

## Considered Options

* Prefix internal columns with `_` (e.g., `_created_at`)
* Suffix internal columns with `_at` / `_by` without a prefix (no distinction)
* Use a separate audit table for lifecycle metadata

## Decision Outcome

Chosen option: "Prefix internal columns with `_`", because the leading underscore immediately signals that a column is infrastructure-level, not a business field, and groups all internal columns together alphabetically at the start of the column list.

### Examples

```typescript
import { pgTable, text, timestamp } from "drizzle-orm/pg-core";

export const users = pgTable("user", {
  // Business columns — domain data
  id: text("id").primaryKey(),
  username: text("username").notNull().unique(),
  email: text("email").notNull(),

  // Internal columns — prefixed with _
  _createdAt: timestamp("_created_at", { withTimezone: true }).notNull().defaultNow(),
  _updatedAt: timestamp("_updated_at", { withTimezone: true }).notNull().defaultNow(),
  _deletedAt: timestamp("_deleted_at", { withTimezone: true }),
});
```

Standard internal columns present on every table:

| Column | Type | Purpose |
|---|---|---|
| `_created_at` | `timestamptz` | When the row was inserted |
| `_updated_at` | `timestamptz` | When the row was last modified |
| `_deleted_at` | `timestamptz` | Soft delete marker — `NULL` means active |

### Consequences

* Good, because internal columns are immediately visually distinct from business columns
* Good, because `_` prefix groups internal columns alphabetically, keeping them out of the way when scanning business fields
* Good, because easy to programmatically exclude `_` prefixed fields from DTO mappings and API responses
* Bad, because some database tools or ORMs may treat leading underscores as a special convention — verify compatibility

### Confirmation

All tables must include `_created_at`, `_updated_at`, and `_deleted_at`. Any column that is not a business domain field must be prefixed with `_`. Enforced via code review and Drizzle schema review.

## Pros and Cons of the Options

### Prefix internal columns with `_`

* Good, because visually distinct — no ambiguity between business and internal fields
* Good, because consistent grouping alphabetically
* Good, because easy to filter programmatically
* Bad, because unfamiliar to developers who haven't seen the convention before

### Suffix columns with `_at` / `_by` without a prefix

* Good, because no special convention to learn
* Bad, because timestamp suffixes are also used on business columns (e.g., `joined_at`, `placed_at`) — no clear distinction
* Bad, because internal columns are scattered throughout the column list

### Separate audit table

* Good, because business tables contain only business data
* Bad, because requires a join on every query that needs lifecycle metadata
* Bad, because `_deleted_at` for soft deletes must still live on the main table to filter active rows efficiently
