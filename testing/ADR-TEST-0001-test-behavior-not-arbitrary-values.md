---
status: proposed
date: 2026-09-10
tags: [testing, unit-tests, quality]
---
# Test Behavior, Not Arbitrary Values

## Directive

Unit tests must validate behavior that is actually required — business rules, contracts, and edge cases that matter — not arbitrary or incidental implementation values that happen to be true today. A value should only be pinned in a test if something (a business requirement, a spec, an external contract) actually mandates that value; otherwise the value is free to change and a test asserting it only creates busywork. Unit tests must also never duplicate checks that a linter, type checker, schema validator, or other static/validation tool already enforces.

## Context and Problem Statement

A unit test's purpose is to fail when behavior that matters is broken. When a test instead pins an arbitrary or incidental value — a constant with no stated rationale, a default that could just as easily be something else — it does not protect any actual requirement; it only locks in whatever the implementation currently does. Every time that incidental value is tweaked, the test must be updated too, even though nothing meaningful changed. The same problem shows up when a test duplicates a check a linter or type checker already performs: the tool already catches the mistake, faster and at a different point in the workflow, so the test adds maintenance cost without adding protection.

## Decision Drivers

* Unit tests should encode requirements and contracts, not incidental implementation details
* A value with no stated business rule behind it is free to change without behavior actually breaking, so pinning it in a test manufactures false failures
* A test that duplicates what a linter, type checker, or schema validator already enforces provides no additional protection and doubles the maintenance cost of that check
* Test suites must stay small enough to read and maintain; assertions on arbitrary values or tool-covered checks bloat the suite without adding confidence
* When a value genuinely matters, it should be tested because it matters — deliberately, with the requirement named — not incidentally because it happened to be observable

## Considered Options

* Test only requirement-backed behavior — skip arbitrary values and tool-covered checks
* Test every observable value or output, regardless of whether it stems from a stated requirement
* Duplicate lint/type-checker validation in unit tests for extra safety

## Decision Outcome

Chosen option: "Test only requirement-backed behavior", because it keeps the test suite failing only when something that actually matters breaks, avoids busywork from routine tweaks to incidental values, and avoids paying twice for checks a linter or type checker already performs.

### Examples

Bad — pins an arbitrary value with no requirement behind it:

```typescript
it("returns 42 for the default page size", () => {
  expect(getDefaultPageSize()).toBe(42);
});
```

`42` is just whatever the constant currently is. If it's changed to `50` next month for no particular reason, this test breaks even though nothing meaningful changed.

Good — the value is dictated by an actual business rule, and the test says so:

```typescript
it("waives shipping for orders over the $50 free-shipping threshold", () => {
  expect(calculateShipping({ subtotal: 51 })).toBe(0);
});
```

Bad — duplicates a check the type system already performs:

```typescript
it("rejects a non-string name", () => {
  expect(() => createUser({ name: 123 as any })).toThrow();
});
```

TypeScript already rejects `123` for a `string` parameter at compile time; this test asserts a runtime behavior that only exists because the test bypassed the type system (`as any`) to force it.

Good — tests actual runtime/business behavior instead:

```typescript
it("throws when the email is already registered", () => {
  expect(() => createUser({ name: "Ada", email: existingEmail })).toThrow(
    EmailAlreadyRegisteredError,
  );
});
```

### Consequences

* Good, because the test suite fails only when behavior that actually matters breaks, not when an incidental value is tweaked
* Good, because the suite stays smaller and cheaper to maintain
* Good, because it avoids paying twice for checks a linter, type checker, or schema validator already performs
* Bad, because distinguishing "arbitrary" from "requirement-backed" is a judgment call that isn't always obvious at the time a test is written
* Bad, because a value that starts arbitrary and later becomes business-critical needs a test added at that point — it won't already have one

### Confirmation

Code review must flag new unit tests that assert an incidental or arbitrary value with no stated requirement behind it, and must flag tests that assert something already enforced by a linter, type checker, or schema validator — asking for the assertion to be removed, or for the requirement it's protecting to be named in the test description.

## Pros and Cons of the Options

### Test only requirement-backed behavior

* Good, because the suite only breaks when something that matters breaks
* Good, because smaller suite, less maintenance
* Bad, because requires judgment calls about what counts as "arbitrary"

### Test every observable value or output

* Good, because nothing is missed — every value is covered by some assertion
* Bad, because routine changes to incidental values cause unnecessary test churn
* Bad, because a bloated suite makes it harder to tell which failures represent a real regression

### Duplicate lint/type-checker validation in unit tests

* Good, because the check is visible directly in the test suite, not only in tool config
* Bad, because the same mistake is now caught (and must be maintained) in two places instead of one
* Bad, because it slows down the suite for a check a linter or compiler already performs in seconds

## More Information

* Related: [ADR-DOCS-0003 — Tests Over Comments](../documentation/ADR-DOCS-0003-tests-over-comments.md)
