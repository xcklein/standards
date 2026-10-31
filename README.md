# standards

Christian Klein's personal coding standards, written as Architecture Decision Records (ADRs). AI coding tools (Claude Code, Cursor, Copilot, etc.) and people working on his projects follow these.

Start with [`STANDARDS.md`](STANDARDS.md): it lists every standard's directive in one file. Open an individual ADR only when you need its context, examples, or rationale.

## Using these standards in a project

Point the project's `CLAUDE.md` (or the equivalent for other tools) at this repository:

```markdown
## Development Standards

Baseline engineering standards are maintained centrally in the
[xcklein/standards](https://github.com/xcklein/standards) repo. Read its
`STANDARDS.md` and follow every applicable standard in addition to the
project-specific rules below. Deviations are recorded in `docs/adr/`.
```

A standard applies wherever its directive's scope matches (e.g. "All TypeScript projects", "Content sites"). Project-specific rules may add to the standards but should not contradict them without a recorded exception.

### Exceptions

When a project deliberately departs from a standard, record it as an ADR in that project's `docs/adr/` directory. The project ADR must link the standard it departs from, and say why the standard does not fit and what the project does instead. Standards that mention a "service-level ADR" mean this.

If the same exception keeps coming up across projects, change the standard instead.

## Categories

ADRs live in top-level category directories. Each category has a short uppercase code used in IDs and filenames, and its own number sequence.

| Directory | Code | Covers |
|---|---|---|
| [`api/`](api/) | `API` | HTTP API design and contracts |
| [`aws/`](aws/) | `AWS` | AWS infrastructure and CDK |
| [`database/`](database/) | `DB` | Database schema and data handling |
| [`documentation/`](documentation/) | `DOCS` | Code comments and documentation |
| [`git/`](git/) | `GIT` | Commits, branching, versioning, and releases |
| [`testing/`](testing/) | `TEST` | Test-writing practice, independent of language |
| [`typescript/`](typescript/) | `TS` | TypeScript tooling and language conventions |
| [`ui/`](ui/) | `UI` | Frontend stack, markup, accessibility, and user-facing copy |

A new category gets a new directory and code. Avoid the directory names `docs` and `templates`, which tooling treats as reserved.

## File naming

```
{category}/ADR-{CODE}-{NNNN}-{slug}.md
```

- `{NNNN}` is a four-digit, zero-padded number scoped to the category, starting at `0001`
- `{slug}` is the kebab-case title
- Numbers are never reused, even after an ADR is deleted or superseded

Examples: `aws/ADR-AWS-0001-use-cdk.md`, `typescript/ADR-TS-0004-use-pnpm.md`

## Frontmatter

```yaml
---
status: proposed | accepted | deprecated | superseded
date: YYYY-MM-DD
tags: [typescript, testing, tooling]
superseded-by: ADR-{CODE}-{NNNN}   # only when status is superseded
---
```

| Status | Meaning |
|---|---|
| `proposed` | Drafted on a branch or in a pull request; not yet binding |
| `accepted` | In force. Every ADR on `main` is accepted unless marked otherwise |
| `deprecated` | No longer applies, with nothing replacing it |
| `superseded` | Replaced by the ADR named in `superseded-by` |

`date` is when the ADR was created.

## Sections

Copy [`ADR-TEMPLATE.md`](ADR-TEMPLATE.md) and fill in the sections:

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

The Directive comes first and must stand on its own: one short paragraph of concrete, imperative rules that an agent can follow without reading the rest.

## Changing standards

- **New ADR:** copy the template, or with Claude Code and the `ck` plugin from [xcklein/claude-plugins](https://github.com/xcklein/claude-plugins), run `/ck:standards-create`. Add its row to `STANDARDS.md`.
- **Edit:** update the ADR in place. If the Directive changes, update its row in `STANDARDS.md` in the same commit.
- **Replace:** set the old ADR to `superseded` with `superseded-by`, and remove its row from `STANDARDS.md`.
- **Retire:** set the ADR to `deprecated` and remove its row from `STANDARDS.md`.
- **Removing outright** is fine for a standard that was never followed. Remove its row too, and fix any links to it.

Commit messages follow Conventional Commits (see ADR-GIT-0001), e.g. `docs: add ADR-TS-0018 typed tool configs`.
