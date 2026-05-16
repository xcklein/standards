---
name: adr-create
description: Use when creating a new Architecture Decision Record in this standards repository. Guides through filling out the ADR template, determines the next ADR number within the chosen category, and writes the file.
---

# Creating a New ADR

## Step 1 — Determine the category

Ask the user: **Which category does this ADR belong to?**

List existing categories by scanning for top-level directories in the repository root (exclude `.git`, `.claude`, `docs`, `templates`). If none exist yet, say so.

The user may pick an existing category or name a new one. If new:
- Ask for the short uppercase category code (e.g. `AWS`, `TS`, `PY`)
- The directory will be created when the file is written

## Step 2 — Determine the next ADR number

Scan the chosen category directory for files matching `ADR-{CODE}-*.md`. Find the highest four-digit number and increment by one. Zero-pad to four digits.

If no ADRs exist in the category yet, start at `0001`.

Use the Glob tool:
- Pattern: `{category}/ADR-{CODE}-*.md`
- Parse the four-digit number from each matching filename

## Step 3 — Gather information from the user

Ask the following questions **one at a time**. Do not ask the next question until the user has answered the current one.

1. **Title**: What is a short, descriptive title for this decision? (Will become the H1 and the filename slug.)
2. **Directive**: State the rule in one to three sentences. Be concrete and imperative — what must or must not be done? This becomes the TL;DR an agent reads first.
3. **Context**: What problem or situation prompted this decision? (2–4 sentences.)
4. **Decision Drivers**: What forces, constraints, or goals influenced this decision? (Bulleted list, or skip.)
5. **Considered Options**: What alternatives did you evaluate? (List each option by name.)
6. **Chosen Option**: Which option did you choose, and why? (One-line justification.)
7. **Examples**: Show one or more code samples demonstrating correct usage of the chosen option.
8. **Consequences**: What are the good and bad outcomes of this decision? (Optional — skip if none.)
9. **Confirmation**: How will you verify this decision is being followed? (Optional — skip if none.)
10. **Pros and Cons per Option**: For each option listed in step 5, provide Good/Bad/Neutral bullets. No code needed here.
11. **Tags**: What tags apply? (e.g., `typescript`, `testing`, `tooling`, `ci`, `formatting`)
12. **More Information**: Any links, related ADRs, or further reading? (Optional — skip if none.)

## Step 4 — Derive the filename

1. Convert the title to kebab-case
2. Strip any characters that are not letters, digits, or hyphens
3. Combine: `{category}/ADR-{CODE}-{NNNN}-{slug}.md`

Examples:
- "Use Biome for Linting" in `typescript` (code `TS`) → `typescript/ADR-TS-0001-use-biome-for-linting.md`
- "Use S3 for Static Assets" in `aws` (code `AWS`) → `aws/ADR-AWS-0001-use-s3-for-static-assets.md`

## Step 5 — Write the file

Write the ADR to the category directory using the structure from `templates/ADR-TEMPLATE.md`.

Set:
- `status: proposed`
- `date`: today's date in `YYYY-MM-DD` format
- `tags`: from the user's answer in step 11

The `## Directive` section must appear immediately after the H1 title, before `## Context and Problem Statement`.

Omit optional sections (`Decision Drivers`, `Consequences`, `Confirmation`, `More Information`) if the user skipped them.

Create the category directory if it does not exist.

## Step 6 — Update STANDARDS.md

Add a row for the new ADR to the appropriate category table in `STANDARDS.md`. The row format is:

```
| [ADR-{CODE}-{NNNN}]({relative-path}) | {Title} | {Directive text} |
```

Append the row at the end of the category table. If the category does not yet exist in `STANDARDS.md`, add a new `## {Category}` section with a table header before the row.

## Step 7 — Confirm and commit

Show the user the final file path and a summary of the sections written. Ask if they want to commit the new file.

If yes:

```bash
git add {filepath}
git commit -m "feat: add {filename}"
```
