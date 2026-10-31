---
status: accepted
date: 2026-10-31
tags: [typescript, node, tooling, ci]
---
# Use Volta

## Directive

All TypeScript projects must pin an exact Node version with Volta, in the `volta.node` field of `package.json`. CI must read the Node version from `package.json` rather than repeating it. `.nvmrc` and `.node-version` must not be used to pin Node.

## Context and Problem Statement

Without a pinned Node version, each developer machine, CI runner, and deploy target runs whatever Node it happens to have. Behavior differs between versions (native type stripping, `fetch`, ESM resolution, `node:` APIs), so code that works locally can fail in CI or production. Volta reads the pin from `package.json` and switches Node automatically when entering the project, so the pin is applied without anyone remembering to run `nvm use`. Keeping the pin in `package.json` also puts it next to the `packageManager` pnpm pin (see ADR-TS-0004).

## Decision Drivers

* Every environment should run the same Node version
* The pin should be applied automatically, not by a manual step
* There should be one source of truth for the version, not a value repeated in several files
* Node version files and tools vary; one repo-wide convention avoids guessing which one a project uses

## Considered Options

* Volta (`volta.node` in `package.json`)
* `.nvmrc` / `.node-version` with nvm, fnm, or similar
* `engines.node` only
* No pin

## Decision Outcome

Chosen option: "Volta", because it switches Node automatically per project, keeps the pin in `package.json` beside the pnpm pin, and `actions/setup-node` can read it directly.

### Examples

`package.json`:

```json
{
  "packageManager": "pnpm@10.25.0",
  "volta": {
    "node": "24.14.1"
  }
}
```

CI reads the same pin:

```yaml
- uses: actions/setup-node@v5
  with:
    node-version-file: package.json
```

Deploy targets that need a runtime setting (e.g. a Lambda `runtime`) must use the same Node major version as the pin.

### Consequences

* Good, because local, CI, and deploy environments run the same Node version
* Good, because Volta switches versions automatically on entering the project
* Good, because the version is changed in one place
* Bad, because every developer must install Volta
* Bad, because a Lambda or other managed runtime only pins the major version, so patch versions can still differ in production

### Confirmation

Code review must flag a `package.json` without an exact `volta.node` version, CI workflows that hardcode a Node version instead of using `node-version-file: package.json`, and new `.nvmrc` or `.node-version` files.

## Pros and Cons of the Options

### Volta

* Good, because it switches Node automatically per project
* Good, because the pin lives in `package.json` with the pnpm pin
* Good, because `actions/setup-node` reads it with `node-version-file: package.json`
* Bad, because it is one more tool to install

### `.nvmrc` / `.node-version`

* Good, because widely supported by version managers and `actions/setup-node`
* Bad, because nvm needs a manual `nvm use` unless a shell hook is configured
* Bad, because it is a separate file from the rest of the project's toolchain pins

### `engines.node` only

* Good, because it needs no extra tooling
* Bad, because it is a compatibility range, not a pin, and does not switch versions
* Bad, because pnpm only warns on a mismatch unless `engine-strict` is set

### No pin

* Good, because nothing to maintain
* Bad, because each environment runs whatever Node is installed

## More Information

* [Volta](https://volta.sh/)
* [actions/setup-node — `node-version-file`](https://github.com/actions/setup-node/blob/main/docs/advanced-usage.md#node-version-file)
* Related: [ADR-TS-0004 — Use pnpm](ADR-TS-0004-use-pnpm.md)
