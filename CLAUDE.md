# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository contains Christian Klein's personal coding standards, written as ADRs, for AI coding tools (Claude Code, Cursor, Copilot, etc.) to follow across projects. It is not a software project; there is nothing to build or test.

[`README.md`](README.md) is the reference for categories, numbering, frontmatter, statuses, required sections, and how standards change. Read it before adding or restructuring ADRs.

## Writing standards

- One decision per ADR: one tool, one rule, or one pattern.
- The Directive states concrete, imperative rules and stands on its own; an agent reading only `STANDARDS.md` must be able to follow it. Keep rationale out of the Directive and in the rest of the ADR.
- Scope each rule to what was actually decided. Don't add requirements, examples, or tooling choices beyond what the decision covers; ask instead.
- Don't restate what any competent developer already does.
- Prefer generic, domain-neutral examples (e.g. orders, users) over ones tied to a specific project.
- When a rule touches another ADR, link it under More Information and check that the two don't contradict each other.

## Keeping the repo consistent

- `STANDARDS.md` has one row per active ADR: link, title (matching the ADR's H1), and its Directive, condensed if long but never contradicting the ADR. Add, update, or remove the row in the same change as the ADR.
- New ADRs on `main` are `accepted`; `proposed` is only for drafts on a branch or in a pull request.
- Take the next number in the category from the existing files; numbers are never reused.
- Commit messages follow Conventional Commits (ADR-GIT-0001), e.g. `docs: add ADR-UI-0012 …`.
