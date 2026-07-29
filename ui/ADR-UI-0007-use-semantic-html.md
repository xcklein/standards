---
status: accepted
date: 2026-07-24
tags: [ui, html, accessibility, react]
---
# Use Semantic HTML

## Directive

Semantic HTML elements must be used instead of generic `div` or `span` elements whenever a semantic element is appropriate for the content's meaning or role (e.g., `nav`, `main`, `header`, `footer`, `article`, `section`, `button`, `ul`/`li`). `div` and `span` are reserved for cases with no matching semantic meaning, typically styling wrappers.

## Context and Problem Statement

Markup built entirely from `div` and `span` conveys no structure or meaning to assistive technology, browsers, or search engines — a screen reader cannot distinguish a page's navigation from its main content, or a clickable action from a decorative box, unless that structure is expressed in the markup itself. Semantic elements communicate this structure for free and often come with built-in behavior (keyboard focus, default ARIA roles) that would otherwise have to be reimplemented by hand.

## Decision Drivers

* Assistive technology (screen readers, browser accessibility trees) depends on semantic structure to let users navigate and understand a page
* Interactive elements (`button`, `a`) provide keyboard accessibility and default ARIA roles for free; a `div` with a click handler does not
* Semantic structure improves SEO by giving crawlers a clear document outline
* Consistent use of semantic elements reduces the amount of manual ARIA markup needed to reach the same accessibility bar

## Considered Options

* Semantic HTML elements where appropriate, `div`/`span` otherwise
* `div`/`span` everywhere, with ARIA roles added manually
* `div`/`span` everywhere, with no accessibility remediation

## Decision Outcome

Chosen option: "Semantic HTML elements where appropriate", because it gives correct accessibility semantics and keyboard behavior by default, requires less manual ARIA to reach the same bar, and costs nothing over using a `div` — the semantic element is a drop-in replacement in JSX.

### Examples

Bad — no structural or interactive meaning:

```tsx
<div onClick={handleSubmit}>Submit</div>

<div>
  <div>Home</div>
  <div>About</div>
</div>
```

Good — semantic elements convey structure and behavior:

```tsx
<button onClick={handleSubmit}>Submit</button>

<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>
```

`div`/`span` remain correct where there is no semantic meaning to express — e.g., a purely visual wrapper used only for layout or styling:

```tsx
<div className="flex gap-4">
  <Sidebar />
  <MainContent />
</div>
```

### Consequences

* Good, because assistive technology gets an accurate document structure and interactive elements for free
* Good, because interactive semantic elements (`button`, `a`) provide keyboard focus and activation without extra code
* Good, because less manual ARIA is needed to reach the same accessibility bar
* Good, because semantic structure improves SEO
* Bad, because some semantic elements carry default browser styling that must be reset (e.g., `button`, `ul`) — mitigated by the project's utility-CSS conventions (see [ADR-UI-0003](ADR-UI-0003-use-tailwind.md))
* Bad, because choosing the correct semantic element requires more upfront knowledge of the HTML spec than defaulting to `div`

### Confirmation

An accessibility/JSX lint rule (e.g. `eslint-plugin-jsx-a11y`) must flag clickable `div`/`span` elements and missing landmark elements. Code review must flag markup that uses `div`/`span` where a semantic element clearly applies.

## Pros and Cons of the Options

### Semantic HTML elements where appropriate

* Good, because correct accessibility semantics and keyboard behavior come for free
* Good, because reduces the amount of manual ARIA required
* Bad, because requires knowing which semantic element fits which case

### `div`/`span` everywhere with manual ARIA

* Good, because visual output is identical regardless of markup choice
* Bad, because reimplements behavior (keyboard focus, roles) that semantic elements provide natively
* Bad, because manual ARIA is easy to get wrong or let drift out of sync with behavior

### `div`/`span` everywhere with no remediation

* Good, because fastest to write with no accessibility considerations
* Bad, because inaccessible to assistive technology users
* Bad, because provides no document outline for SEO or tooling

## More Information

* Related: [ADR-UI-0001 — Use React](ADR-UI-0001-use-react.md)
* Related: [ADR-UI-0003 — Use Tailwind CSS](ADR-UI-0003-use-tailwind.md)
* [MDN — HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
