---
status: accepted
date: 2026-08-08
tags: [typescript, errors, style]
---
# Defining Errors

## Directive

Errors must be defined as custom classes extending `Error`, one class per error type. When a module emits multiple related error types, define a custom base error class extending `Error`, then extend that base class for each specific error. Error classes may carry custom fields for additional context. A single error class with a `code` field to differentiate error scenarios is not supported.

## Context and Problem Statement

Throwing plain `Error` instances or a single generic error class with a discriminating `code` string gives callers no way to distinguish error types using TypeScript's own type system — every `catch` block has to inspect a string field and hope it was set correctly, with no compiler backing. Distinct error classes let `instanceof` checks narrow the caught error to its concrete type, give each error type room for its own strongly-typed context fields, and make the set of possible errors a module can throw discoverable from its exports rather than from a scattered list of string codes.

## Decision Drivers

* Callers must be able to distinguish error types using `instanceof`, not string comparison
* Each error type may need to carry different structured context (e.g. a validation error's failed fields vs. a network error's status code)
* The set of errors a module can throw should be discoverable from its type exports
* A shared base class should exist when a module emits multiple related errors, so callers can catch the whole family with one `instanceof` check

## Considered Options

* Custom error classes extending `Error`, one per error type, with a shared base class when a module has multiple error types
* A single custom error class with a `code` field to differentiate scenarios
* Plain `Error` with no subclassing

## Decision Outcome

Chosen option: "Custom error classes extending `Error`", because `instanceof` gives compiler-backed narrowing that a `code` field cannot, each class can carry exactly the context fields relevant to that failure, and a shared base class still allows catching an entire family of related errors with a single check.

### Examples

A single error type:

```typescript
export class UserNotFoundError extends Error {
  constructor(public readonly userId: string) {
    super(`User not found: ${userId}`);
    this.name = "UserNotFoundError";
  }
}
```

A module with multiple related error types, sharing a base class:

```typescript
export abstract class PaymentError extends Error {
  constructor(message: string) {
    super(message);
    this.name = new.target.name;
  }
}

export class InsufficientFundsError extends PaymentError {
  constructor(public readonly shortfallCents: number) {
    super(`Insufficient funds: short by ${shortfallCents} cents`);
  }
}

export class CardDeclinedError extends PaymentError {
  constructor(public readonly declineCode: string) {
    super(`Card declined: ${declineCode}`);
  }
}
```

Catching a specific error, or the whole family:

```typescript
try {
  await chargeCard(order);
} catch (err) {
  if (err instanceof CardDeclinedError) {
    return retryWithBackupCard(order, err.declineCode);
  }
  if (err instanceof PaymentError) {
    logger.error("Payment failed", { message: err.message });
  }
  throw err;
}
```

Not supported — a single error class differentiated by a `code` field:

```typescript
// Do not do this.
class AppError extends Error {
  constructor(
    public readonly code: "USER_NOT_FOUND" | "INSUFFICIENT_FUNDS" | "CARD_DECLINED",
    message: string,
  ) {
    super(message);
  }
}
```

### Consequences

* Good, because `instanceof` narrows the caught error to its concrete type with full compiler support
* Good, because each error class carries only the context fields relevant to that specific failure
* Good, because a module's exported error classes document the errors it can throw
* Good, because a shared base class lets callers catch an entire family of related errors in one check
* Bad, because more classes must be defined and exported compared to one generic error type
* Bad, because forgetting to set `name` (or use `new.target.name` in a base class) leaves the default `"Error"` name in stack traces and serialized output

### Confirmation

Code review must flag new errors defined as a generic class with a `code`/`type` discriminant field, or thrown as plain `Error` instances, and request a dedicated subclass instead.

## Pros and Cons of the Options

### Custom error classes, one per type, shared base class when needed

* Good, because `instanceof` gives compiler-backed narrowing
* Good, because each class can declare exactly the fields relevant to it
* Good, because a base class still supports catching a whole family at once
* Bad, because more boilerplate than a single generic class

### Single custom error class with a `code` field

* Good, because only one class to define and export per module
* Bad, because narrowing on `code` requires a manual switch/if chain with no compiler-enforced exhaustiveness unless additionally paired with a discriminated union
* Bad, because every error scenario shares the same shape, so scenario-specific context fields end up optional on all of them

### Plain `Error`, no subclassing

* Good, because zero additional code
* Bad, because callers cannot distinguish error types at all without parsing the message string
* Bad, because no way to attach structured context beyond the message

## More Information

* Related: [ADR-API-0003 — Error Response Format](../api/ADR-API-0003-error-response-format.md)
