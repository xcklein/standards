# standards

Christian Klein's personal coding standards, structured as Architecture Decision Records (ADRs).

## What's in here

Each file is a single decision — one tool, one rule, one pattern. Files are verbose: they include context, rationale, alternatives considered, and code examples for the chosen approach.

## Categories

ADRs are organized into top-level category directories. Each category has a short uppercase code used in filenames.

Examples: `aws/`, `typescript/` (code: `TS`), `python/` (code: `PY`)

## File naming

```
{category}/ADR-{CODE}-{NNNN}-<short-slug>.md
```

- Category directory at the root (e.g. `aws/`, `typescript/`)
- `ADR` prefix followed by the category code and a four-digit zero-padded number
- Numbers are scoped per category — each category has its own sequence starting at `0001`
- Kebab-case slug matching the document title
- Examples:
  - `aws/ADR-AWS-0001-use-cdk.md`
  - `typescript/ADR-TS-0004-use-pnpm.md`

## Frontmatter

Every ADR begins with a YAML frontmatter block:

```yaml
---
status: proposed | accepted | deprecated | superseded
date: YYYY-MM-DD
tags: [typescript, testing, tooling, …]
superseded-by: ADR-NNNN-<slug>   # only when status: superseded
---
```

## Creating a new ADR

Copy `templates/ADR-TEMPLATE.md`, rename it following the convention above, and fill in the sections.

If using Claude Code, invoke the `adr-create` skill (`.claude/skills/adr-create/SKILL.md`):

```
/adr-create
```

## Reference

`STANDARDS.md` lists every active ADR with its one-line directive, grouped by category. Load this file for a quick overview without reading individual ADRs.

## Sections

| Section | Required? |
|---|---|
| Directive | yes |
| Context and Problem Statement | yes |
| Decision Drivers | optional |
| Considered Options | yes |
| Decision Outcome | yes |
| Examples (subsection of Decision Outcome) | yes |
| Consequences (subsection of Decision Outcome) | optional |
| Confirmation (subsection of Decision Outcome) | optional |
| Pros and Cons of the Options | yes |
| More Information | optional |
