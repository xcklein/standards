---
status: accepted
date: 2026-05-16
tags: [database, conventions, schema]
---
# Prefer Soft Deletes

## Directive

All delete operations must set `_deleted_at` rather than removing the row. All active-record queries must filter `WHERE _deleted_at IS NULL`. Hard deletes are not permitted without an explicit service-level ADR.

## Context and Problem Statement

Deleting rows permanently from a database is irreversible and destroys audit history. Referential integrity can break when related records reference a deleted row, and accidental deletions have no recovery path without restoring from a backup. Soft deletes preserve the row by marking it with a deletion timestamp, keeping data recoverable and history intact.

## Decision Drivers

* Deleted data must be recoverable without a full backup restore
* Audit history must be preserved — who created and modified a record, and when it was removed
* Foreign key relationships must remain intact after a record is removed from active use
* Active and deleted records must be efficiently distinguishable in queries

## Considered Options

* Soft deletes via `_deleted_at` timestamp
* Hard deletes (permanent row removal)
* Archive table (move deleted rows to a separate table)

## Decision Outcome

Chosen option: "Soft deletes via `_deleted_at`", because it preserves audit history, keeps foreign key relationships intact, and allows recovery without a backup restore, at the cost of requiring a `WHERE _deleted_at IS NULL` filter on all active-record queries.

### Examples

Mark a record as deleted:

```typescript
await db
  .update(grids)
  .set({ _deletedAt: new Date(), _updatedAt: new Date() })
  .where(and(eq(grids.id, gridId), isNull(grids._deletedAt)))
  .returning();
```

All queries for active records must filter on `_deleted_at IS NULL`:

```typescript
// Active records only
const activeUsers = await db
  .select()
  .from(users)
  .where(isNull(users._deletedAt));

// Including deleted records (admin/audit use only)
const allUsers = await db.select().from(users);
```

An index on `_deleted_at` is required on every table to keep active-record queries efficient:

```typescript
export const users = pgTable(
  "user",
  { /* columns */ },
  (t) => [
    index("users_deleted_at_idx").on(t._deletedAt),
  ],
);
```

### Consequences

* Good, because deleted records are recoverable without a backup restore
* Good, because audit history and foreign key relationships are preserved
* Good, because deletion is reversible — setting `_deleted_at` back to `NULL` restores the record
* Bad, because every active-record query must include `WHERE _deleted_at IS NULL` — omitting it is a silent bug that returns deleted records
* Bad, because table size grows indefinitely — a separate purge or archival process is needed for truly obsolete data
* Bad, because unique constraints must account for soft-deleted rows (e.g., a unique username may be held by a deleted user)

### Confirmation

All tables must include a `_deleted_at` column (see ADR-DB-0001). Delete operations must set `_deleted_at` rather than removing the row. All active-record queries must filter `WHERE _deleted_at IS NULL`. Enforced via code review.

## Pros and Cons of the Options

### Soft deletes via `_deleted_at`

* Good, because data is recoverable and audit history is preserved
* Good, because foreign key relationships remain intact
* Good, because deletion is reversible
* Bad, because every query must carry a `_deleted_at IS NULL` filter
* Bad, because unique constraints require special handling for deleted rows
* Bad, because tables grow indefinitely without a purge strategy

### Hard deletes

* Good, because queries are simpler — no filter required
* Good, because tables stay lean
* Bad, because deletion is permanent — no recovery without a backup
* Bad, because foreign key constraints must be carefully managed to avoid orphaned references
* Bad, because audit history is lost

### Archive table

* Good, because active table stays lean and queries are clean
* Good, because deleted data is preserved in a separate location
* Bad, because requires a migration step on delete — more complex than setting a timestamp
* Bad, because foreign keys pointing to the main table break when a row is moved to the archive
* Bad, because querying across active and archived data requires a union

## More Information

* Related: [ADR-DB-0001 — Prefix Internal Columns with `_`](ADR-DB-0001-internal-column-prefix.md)
