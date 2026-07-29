# Standards

A flat index of all active standards, grouped by category. Each entry links to the full ADR with its complete context, examples, and rationale. Load this file first to get a complete picture of all standards without reading every ADR individually — follow the links only when deeper context is needed.

---

## AWS

| ID | Title | Directive |
|----|-------|-----------|
| [ADR-AWS-0001](aws/ADR-AWS-0001-use-cdk.md) | Use CDK | All infrastructure must be defined and deployed using AWS CDK. Infrastructure must not be created or modified manually via the AWS console or CLI. |
| [ADR-AWS-0002](aws/ADR-AWS-0002-use-cdk-nag.md) | Use CDK Nag | Every CDK app entry point must apply `AwsSolutionsChecks` via `Aspects.of(app)`. The CDK synth step must fail on unsuppressed violations. All suppressions must include a `reason`. |
| [ADR-AWS-0003](aws/ADR-AWS-0003-tag-all-resources.md) | Tag All Resources | All AWS resources must be tagged with `x:repo`, `x:service`, and `x:env` using `cdk.Tags.of(this)` at the stack level. Additional tags may be added as needed. |

## TypeScript

| ID | Title | Directive |
|----|-------|-----------|
| [ADR-TS-0001](typescript/ADR-TS-0001-use-linter.md) | Use ESLint | All TypeScript projects must use ESLint for linting. Biome must not be used for linting. Every project must include an `eslint.config.js` (flat config) following the standard configuration. |
| [ADR-TS-0002](typescript/ADR-TS-0002-use-lefthook.md) | Use Lefthook | All TypeScript projects must use Lefthook for Git hooks. A `lefthook.yml` must be present at the project root. At minimum it must run `eslint --fix` and `prettier --write` on staged files via a `pre-commit` hook and enforce commit message format via a `commit-msg` hook (see ADR-GIT-0001). |
| [ADR-TS-0003](typescript/ADR-TS-0003-use-esm.md) | Use ESM | All TypeScript projects must use ESM, not CommonJS. `package.json` must include `"type": "module"`. Node-runtime projects use `"moduleResolution": "NodeNext"` with explicit `.js` import extensions; bundler-driven frontend projects use `"moduleResolution": "bundler"` with extensionless imports. |
| [ADR-TS-0004](typescript/ADR-TS-0004-use-pnpm.md) | Use pnpm | All TypeScript projects must use pnpm as the package manager. `package.json` must include a `"packageManager"` field pinned to a specific pnpm version. npm and Yarn must not be used. |
| [ADR-TS-0005](typescript/ADR-TS-0005-use-vitest.md) | Use Vitest | All TypeScript projects must use Vitest as the test runner. Jest must not be used. A `vitest.config.ts` must be present and `package.json` must include `"test": "vitest run"`. |
| [ADR-TS-0006](typescript/ADR-TS-0006-use-zod.md) | Use Zod | All runtime data validation must use Zod. TypeScript types at system boundaries must be derived via `z.infer` rather than declared independently. Joi and Yup must not be used. |
| [ADR-TS-0007](typescript/ADR-TS-0007-use-hono.md) | Use Hono | All HTTP APIs must be built with Hono using `@hono/zod-openapi`. Routes must be defined with `createRoute` and registered via `app.openapi`. Plain `app.get` / `app.post` may be used where `app.openapi` is not applicable (e.g., serving generated documentation). |
| [ADR-TS-0008](typescript/ADR-TS-0008-use-openapi-typescript.md) | Use openapi-typescript | API client types must be generated from the server's OpenAPI schema using `openapi-typescript`. A `gen:api` script must be present in `package.json`. The generated file must be committed to the repository. |
| [ADR-TS-0009](typescript/ADR-TS-0009-use-openapi-fetch.md) | Use openapi-fetch | All typed HTTP requests to internal APIs must use `openapi-fetch` initialised with the generated `paths` type. Direct use of `fetch` or `axios` for typed API calls is not permitted. |
| [ADR-TS-0010](typescript/ADR-TS-0010-validate-env-vars.md) | Validate Environment Variables at Startup | All environment variables must be parsed and validated at application startup using the standard runtime validation library (see ADR-TS-0006). A single typed `config` object must be exported from a dedicated module and used throughout the codebase. Direct access to `process.env` outside that module is not permitted. |
| [ADR-TS-0011](typescript/ADR-TS-0011-structured-json-logging.md) | Structured JSON Logging | All application logs must be emitted as structured JSON. Every log entry must include `level`, `message`, `timestamp`, and `service`. `console.log`, `console.error`, and related methods must not be used in application code. |
| [ADR-TS-0012](typescript/ADR-TS-0012-google-typescript-style-guide.md) | Follow Google TypeScript Style Guide | All TypeScript styling decisions not covered by the formatter must follow the Google TypeScript Style Guide. ESLint and Prettier take precedence where their rules conflict. |
| [ADR-TS-0013](typescript/ADR-TS-0013-use-formatter.md) | Use Prettier | All TypeScript projects must use Prettier for formatting. Biome must not be used for formatting. Every project must include a `prettier.config.js` and stay as close to Prettier's defaults as possible — `tabWidth` and `printWidth` are the only settings most projects should need to override. |

## API

| ID | Title | Directive |
|----|-------|-----------|
| [ADR-API-0001](api/ADR-API-0001-prefer-rest.md) | Prefer REST | All new external-facing APIs must follow REST conventions over HTTP. Deviations (GraphQL, gRPC, tRPC) must be documented and justified in a service-level ADR. |
| [ADR-API-0002](api/ADR-API-0002-openapi-schema.md) | OpenAPI Schema | Every API must serve an OpenAPI 3.1 schema at a well-known path (e.g. `/openapi.json`). The schema must be validated with Spectral in CI. |
| [ADR-API-0003](api/ADR-API-0003-error-response-format.md) | Error Response Format | All error responses must use RFC 9457 Problem Details format with `Content-Type: application/problem+json`. Every error response must include `type`, `title`, `status`, and `detail` fields. `type` URIs do not need to be HTTP-resolvable. |
| [ADR-API-0004](api/ADR-API-0004-authentication.md) | Authentication | All protected API endpoints must require a JWT bearer token issued via OAuth 2.0. Tokens must be signed with RS256 and verified against the issuer's public key. Expiry must be enforced. |
| [ADR-API-0005](api/ADR-API-0005-versioning-strategy.md) | Versioning Strategy | All API routes must include a version prefix (`/v{n}/`). Deprecated versions must respond with `Deprecation` and `Sunset` headers. |
| [ADR-API-0006](api/ADR-API-0006-pagination.md) | Pagination | All collection endpoints must use cursor-based pagination. Offset-based pagination is not permitted. Responses must include a `pagination` object and `Link` headers. |
| [ADR-API-0007](api/ADR-API-0007-naming-conventions.md) | Naming Conventions | All URL paths must use kebab-case with plural nouns. All JSON request and response fields must use camelCase. Query parameter names must use camelCase. |
| [ADR-API-0008](api/ADR-API-0008-content-negotiation.md) | Content Negotiation | All API responses must use `Content-Type: application/json`. Error responses must use `Content-Type: application/problem+json`. File upload endpoints are the only exception and must use `multipart/form-data`. |

## Database

| ID | Title | Directive |
|----|-------|-----------|
| [ADR-DB-0001](database/ADR-DB-0001-internal-column-prefix.md) | Internal Column Prefix | All internal database columns (lifecycle timestamps, soft-delete markers) must be prefixed with `_`. Every table must include `_created_at`, `_updated_at`, and `_deleted_at`. |
| [ADR-DB-0002](database/ADR-DB-0002-prefer-soft-deletes.md) | Prefer Soft Deletes | All delete operations must set `_deleted_at` rather than removing the row. All active-record queries must filter `WHERE _deleted_at IS NULL`. Hard deletes are not permitted without an explicit service-level ADR. |

## Documentation

| ID | Title | Directive |
|----|-------|-----------|
| [ADR-DOCS-0001](documentation/ADR-DOCS-0001-doc-comments.md) | Doc Comments | All exported/public functions, classes, methods, and types must have a documentation comment (JSDoc, Javadoc, or the language's equivalent). Doc comments must describe purpose, parameters, return values, and thrown/rejected errors where applicable. |
| [ADR-DOCS-0002](documentation/ADR-DOCS-0002-inline-comments.md) | Inline Comments | Inline code comments must be used sparingly. They are permitted only to explain unexpected or non-obvious behavior, or to explicitly note the deliberate absence of something (e.g., a `catch` block with no `throw`). Comments must not restate what the code already says. |

## Git

| ID | Title | Directive |
|----|-------|-----------|
| [ADR-GIT-0001](git/ADR-GIT-0001-conventional-commits.md) | Use Conventional Commits | All commits must follow the Conventional Commits specification. The format is `<type>[optional scope]: <description>`. Breaking changes must be indicated with `!` after the type or a `BREAKING CHANGE:` footer. |

## UI

| ID | Title | Directive |
|----|-------|-----------|
| [ADR-UI-0001](ui/ADR-UI-0001-use-react.md) | Use React | All UI must be built with React. Component files must use the `.tsx` extension. `StrictMode` must be enabled in the root entry point. |
| [ADR-UI-0002](ui/ADR-UI-0002-use-shadcn.md) | Use shadcn/ui | All shared UI primitives must be installed via the shadcn CLI into `src/components/ui/`. Raw Radix UI primitives must not be used directly outside of that directory. |
| [ADR-UI-0003](ui/ADR-UI-0003-use-tailwind.md) | Use Tailwind CSS | All component styling must use Tailwind CSS utility classes. Separate `.css` files are only permitted for global styles. The `cn()` utility must be used for all conditional class application. |
| [ADR-UI-0004](ui/ADR-UI-0004-use-tanstack-query.md) | Use TanStack Query | All server data fetching must use TanStack Query's `useQuery` and `useMutation` hooks. Direct `fetch` calls inside components or `useEffect` data fetching are not permitted. |
| [ADR-UI-0005](ui/ADR-UI-0005-use-zustand.md) | Use Zustand | All shared client UI state must be managed with Zustand. Each store must have a single clear responsibility. Server state belongs in TanStack Query; local component state belongs in `useState`. |
| [ADR-UI-0006](ui/ADR-UI-0006-use-vite.md) | Use Vite | All frontend projects must use Vite as the build tool and dev server. A `vite.config.ts` must be present at the project root. |
| [ADR-UI-0007](ui/ADR-UI-0007-use-semantic-html.md) | Use Semantic HTML | Semantic HTML elements must be used instead of generic `div` or `span` elements whenever a semantic element is appropriate for the content's meaning or role. `div` and `span` are reserved for cases with no matching semantic meaning, typically styling wrappers. |
