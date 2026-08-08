---
status: accepted
date: 2026-07-24
tags: [documentation, testing, style]
---
# Tests Over Comments

## Directive

When code is hardened against a specific edge case, regression, or an ambiguous requirement, that knowledge must be encoded as a unit test rather than as an inline comment. Tests are executable and fail when the behavior they protect regresses; comments are inert prose that can go stale, be ignored, or drift out of sync with the code.

## Context and Problem Statement

Code accumulates hard-won knowledge over time — edge cases discovered in production, requirements that were ambiguous until a bug clarified them, behavior that looks wrong but is deliberate. The default way to preserve that knowledge is a comment explaining it, but a comment is prose: nothing checks that it still describes the code beneath it, and nothing stops the behavior it describes from silently regressing. A unit test capturing the same knowledge is enforced automatically — it fails the moment the protected behavior changes, and its assertions are an unambiguous, verifiable statement of what the code is supposed to do.

## Decision Drivers

* Knowledge about edge cases and regressions must be enforced, not just documented
* Comments are not checked by CI and can silently drift out of sync with the code they describe
* A failing test is immediate, unambiguous feedback; a stale or deleted comment produces none
* Ambiguous requirements need an executable definition of correct behavior, not just prose explaining an author's intent

## Considered Options

* Encode hardening knowledge as a unit test
* Encode hardening knowledge as an inline comment
* Encode hardening knowledge in external documentation (wiki page, ticket)

## Decision Outcome

Chosen option: "Unit test", because it is enforced automatically by CI and cannot silently drift out of sync the way a comment or an external document can — the knowledge stays true for as long as the test suite passes.

### Examples

Bad — knowledge lives only in a comment, unenforced:

```typescript
// NOTE: must handle negative amounts correctly — we had a production bug
// where refunds (negative amounts) broke this calculation.
function applyDiscount(amount: number, pct: number): number {
  return amount - amount * pct;
}
```

Good — the same knowledge encoded as a regression test:

```typescript
it("applies the discount correctly to negative amounts (refunds)", () => {
  expect(applyDiscount(-100, 0.1)).toBe(-90);
});
```

The test fails loudly if a future change breaks this case; the comment would simply sit there, true or not, until someone happened to read it.

### Consequences

* Good, because regressions are caught automatically instead of relying on a comment being read and heeded
* Good, because test names and assertions document the requirement in an executable, unambiguous form
* Good, because it reduces the number of stale or misleading comments accumulating in the codebase
* Bad, because it requires a test harness and the discipline to write the test at the time the hardening happens
* Bad, because some edge cases (timing, concurrency, external system behavior) are costly to encode as a test and may still need a short comment as a stopgap

### Confirmation

Code review must treat a comment that explains a bug fix, edge case, or previously ambiguous requirement as a signal to ask whether a regression test should exist instead. A PR that fixes a reported bug should include a test that would have failed before the fix.

## Pros and Cons of the Options

### Unit test

* Good, because enforced by CI — fails loudly the moment the protected behavior regresses
* Good, because doubles as an executable specification of the expected behavior
* Bad, because requires test infrastructure and more upfront effort than writing a comment

### Inline comment

* Good, because fast to write, no infrastructure required
* Bad, because not enforced — nothing prevents it from going stale or being silently deleted
* Bad, because relies entirely on a future reader noticing and reading it

### External documentation

* Good, because can hold richer narrative and context than a test or comment
* Bad, because it lives separately from the code and is easily forgotten
* Bad, because not enforced by CI and not colocated with the behavior it describes

## More Information

* Related: [ADR-DOCS-0002 — Inline Comments](ADR-DOCS-0002-inline-comments.md)
* Related: [ADR-TS-0005 — Use Vitest](../typescript/ADR-TS-0005-use-vitest.md)
