---
status: proposed
date: 2026-10-11
tags: [ui, accessibility, wcag, a11y]
---
# Meet WCAG 2.2 AA

## Directive

All user-facing UI must conform to WCAG 2.2 Level AA. Every UI project must lint with `eslint-plugin-jsx-a11y`'s recommended config. Automated checks against the rendered DOM (e.g. axe) are recommended for what static linting cannot see, such as color contrast. New interactive flows must be checked by hand with a keyboard alone, because automated tools catch only a fraction of WCAG failures.

## Context and Problem Statement

"Accessible" without a named target is not reviewable: one reviewer accepts a missing focus ring, another blocks on it. WCAG 2.2 Level AA is the conformance level most accessibility laws and procurement rules reference, and it is achievable for ordinary product UI. The existing standards (semantic HTML in ADR-UI-0007, Radix-based components in ADR-UI-0002) cover a lot of AA by default, but not color contrast, focus visibility, target size, or form error handling. Those have to be checked deliberately.

## Decision Drivers

* Accessibility needs a concrete, external target that reviewers can check against
* Level AA is the level legal and procurement requirements typically cite
* Level AAA includes criteria that cannot be met for all content, so it is not a practical blanket target
* Linting catches markup mistakes cheaply, but cannot see computed styles such as contrast
* Automated tools miss many failures (keyboard traps, logical focus order, meaningful alt text), so some manual checking is unavoidable

## Considered Options

* WCAG 2.2 Level AA
* WCAG 2.2 Level A
* WCAG 2.2 Level AAA
* No named target — rely on component libraries and judgment

## Decision Outcome

Chosen option: "WCAG 2.2 Level AA", because it is the level most requirements cite, it is achievable for ordinary product UI, and naming it turns "is this accessible?" into a checkable question.

### Requirements most often missed

These AA criteria are not covered by semantic HTML or Radix components alone:

| Criterion | Requirement |
|---|---|
| 1.4.3 Contrast (Minimum) | Text contrast at least 4.5:1, or 3:1 for large text |
| 1.4.11 Non-text Contrast | UI component boundaries, states, and icons at least 3:1 |
| 2.4.7 Focus Visible | Every focusable element shows a visible focus indicator |
| 2.4.11 Focus Not Obscured | A focused element is not hidden behind sticky headers, banners, or overlays |
| 2.5.8 Target Size (Minimum) | Pointer targets at least 24×24 CSS pixels, or spaced so a 24px circle around each does not overlap another |
| 1.4.10 Reflow | Content works at 320 CSS pixels wide without horizontal scrolling |
| 3.3.1 / 3.3.2 Error Identification, Labels | Every input has a visible label; errors name the field and say what is wrong in text |
| 1.1.1 Non-text Content | Informative images and icon-only buttons have a text alternative; decorative ones are hidden from assistive technology |

### Examples

Icon-only button with a text alternative:

```tsx
<button type="button" onClick={onClose} aria-label="Close dialog">
  <XIcon aria-hidden="true" />
</button>
```

Removing the default outline only with a visible replacement:

```tsx
<button className="outline-none focus-visible:ring-2 focus-visible:ring-ring">
  Save
</button>
```

Field error tied to its input:

```tsx
<label htmlFor="email">Email</label>
<input
  id="email"
  aria-invalid={!!error}
  aria-describedby={error ? "email-error" : undefined}
/>
{error && <p id="email-error">{error}</p>}
```

Linting, in `eslint.config.ts`:

```typescript
import jsxA11y from "eslint-plugin-jsx-a11y";

export default [
  // ...base config
  jsxA11y.flatConfigs.recommended,
];
```

### Consequences

* Good, because reviewers have an external, checkable definition of "accessible"
* Good, because AA meets the level that legal and procurement requirements usually cite
* Good, because linting catches common markup mistakes before review
* Bad, because contrast and target-size limits constrain visual design choices
* Bad, because manual keyboard checks add time to work on new interactive flows
* Neutral, because conformance is reached per page or flow; third-party embeds may need their own remediation or a documented exception

### Confirmation

`eslint-plugin-jsx-a11y`'s recommended config must be enabled and must pass in CI. Code review must ask for a keyboard-only walkthrough of any new interactive flow, and must flag removed focus outlines without a replacement, icon-only controls without a text alternative, and color pairs below the contrast minimums. Automated axe checks against rendered pages or components (e.g. `@axe-core/playwright` or `vitest-axe`) are recommended. They check the rendered DOM, so they cover contrast and other computed results that the linter cannot see, and do not duplicate it (see ADR-TEST-0001).

## Pros and Cons of the Options

### WCAG 2.2 Level AA

* Good, because it is the level most requirements cite
* Good, because it is achievable for ordinary product UI
* Bad, because contrast, focus, and target-size criteria take deliberate design and review effort

### WCAG 2.2 Level A

* Good, because it is the least effort
* Bad, because it omits contrast, visible focus, and reflow, which affect many users
* Bad, because it falls short of what most requirements cite

### WCAG 2.2 Level AAA

* Good, because it is the strongest accessibility bar
* Bad, because W3C does not recommend AAA as a general policy, since some criteria cannot be met for all content
* Bad, because criteria such as 7:1 contrast heavily restrict design

### No named target

* Good, because no extra process
* Bad, because "accessible" means something different to each reviewer
* Bad, because gaps that libraries do not cover (contrast, focus, errors) go unchecked

## More Information

* [WCAG 2.2](https://www.w3.org/TR/WCAG22/)
* [Understanding WCAG 2.2 — conformance levels](https://www.w3.org/WAI/WCAG22/Understanding/conformance)
* Related: [ADR-UI-0002 — Use shadcn/ui](ADR-UI-0002-use-shadcn.md)
* Related: [ADR-UI-0007 — Use Semantic HTML](ADR-UI-0007-use-semantic-html.md)
* Related: [ADR-TEST-0001 — Test Behavior, Not Arbitrary Values](../testing/ADR-TEST-0001-test-behavior-not-arbitrary-values.md)
