---
status: proposed
date: 2026-10-31
tags: [ui, ux, errors, copy, security]
---
# Hide Implementation Details from Users

## Directive

User-facing text (error messages, toasts, empty states, help pages, and other in-app or published user documentation) must describe what happened and what the user can do, not how the system is built. Raw error messages, stack traces, HTTP status codes, API field names, internal service or table names, and third-party provider names must not be shown to users. Errors must be mapped to written messages; anything unmapped falls back to a generic message, optionally with a reference ID for support, while the raw error is logged or reported.

## Context and Problem Statement

The easiest way to show an error is to render whatever the error object says: `error.message`, a status code, or the API's `detail`. That text was written for developers, so users see "TypeError: Failed to fetch", "409", or "WorkOS session expired", none of which tells them what to do next. It also reveals which libraries and providers the product runs on, and it ties the copy to the implementation, so swapping a provider or renaming a field quietly changes what users read. Help pages and user docs drift the same way when they explain features in terms of internal services.

## Decision Drivers

* Users need to know what happened and what to do next, in their terms
* Developer-facing text (exception messages, status codes) is not written for users and often leaks internals
* Copy should not change when a provider, library, or internal name changes
* The raw error must still reach developers through logs or error reporting

## Considered Options

* Map errors to written messages, with a generic fallback and reference ID
* Show the API's or exception's message directly
* Show a generic message for every error

## Decision Outcome

Chosen option: "Map errors to written messages, with a generic fallback and reference ID", because users get actionable copy for known cases, nothing internal leaks for unknown ones, and developers still get the raw error.

### What must not appear in user-facing text

| Detail | Example |
|---|---|
| Raw error text | `error.message`, `String(error)`, "TypeError: Failed to fetch" |
| Stack traces | Any part of `error.stack` |
| Protocol details | "Error 409", "500 Internal Server Error", JSON bodies |
| API or storage names | Field names like `displayName`, table or service names |
| Provider names | "WorkOS session expired", "Stripe error: card_declined" |

A provider name may appear where the user knowingly interacts with that provider — "Sign in with Google", or a Stripe-hosted checkout page — but not in errors or copy that merely happen to pass through it.

### Examples

Bad — shows the raw error:

```tsx
onError: (error) => toast.error(error.message),
```

Good — maps known problems, falls back to a generic message with a reference ID, and reports the raw error:

```typescript
const MESSAGES: Record<string, string> = {
  "https://api.example.com/problems/email-taken":
    "An account with this email already exists. Try signing in instead.",
  "https://api.example.com/problems/insufficient-credits":
    "You don't have enough credits for this. Buy more to continue.",
};

export function toUserMessage(problem: Problem | undefined, referenceId: string): string {
  return (
    (problem && MESSAGES[problem.type]) ??
    `Something went wrong. Please try again. (Reference: ${referenceId})`
  );
}
```

```tsx
onError: (error) => {
  const referenceId = reportError(error);
  toast.error(toUserMessage(asProblem(error), referenceId));
},
```

Bad help text — explains the implementation:

> Your progress is stored in DynamoDB and synced by a Lambda every five minutes.

Good — explains the behavior:

> Your progress is saved automatically every few minutes.

### Consequences

* Good, because users see what happened and what to do next
* Good, because nothing about libraries, providers, or internal names leaks through the UI
* Good, because copy stays the same when the implementation changes
* Good, because a reference ID connects a user's report to the logged error
* Bad, because every user-visible error case needs a written message
* Bad, because a generic fallback gives the user less detail for cases nobody mapped yet

### Confirmation

Code review must flag UI code that renders `error.message`, `String(error)`, an error's stack, a status code, or an API response's `detail` directly, and user-facing copy or docs that name internal services, fields, or providers outside the exception above.

## Pros and Cons of the Options

### Map errors to written messages

* Good, because actionable for users and safe by default
* Bad, because each error case needs copy

### Show the API's or exception's message directly

* Good, because no extra work
* Bad, because the text is written for developers and leaks internals
* Bad, because copy changes whenever the implementation does

### Show a generic message for every error

* Good, because nothing leaks
* Bad, because users can't tell a typo from an outage, or know what to do next

## More Information

* Related: [ADR-API-0003 — Error Response Format](../api/ADR-API-0003-error-response-format.md)
* Related: [ADR-API-0009 — Hide Implementation Details](../api/ADR-API-0009-hide-implementation-details.md)
* Related: [ADR-TS-0014 — Defining Errors](../typescript/ADR-TS-0014-defining-errors.md)
