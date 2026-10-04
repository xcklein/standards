---
status: proposed
date: 2026-10-04
tags: [aws, cdk, accounts, environments, deployment, ci]
---
# Environments and Accounts

## Directive

Each environment gets a dedicated AWS account; an account must never host more than one environment. Every deployable repository has a single CDK app that deploys unchanged into any environment: the target account and region come from the deploying credentials (`CDK_DEFAULT_ACCOUNT`/`CDK_DEFAULT_REGION`), the environment name comes from an `env` context value that selects per-environment configuration and fails synth if unknown, and stack IDs are identical across environments. Stacks are split by concern and named `{Service}{Concern}Stack`. Deployments to any shared environment run from CI through a per-account GitHub OIDC role; production deploys only from `main`, after CI passes on the same commit.

## Context and Problem Statement

Putting several environments in one AWS account couples their blast radius, quotas, and bills: a runaway staging function can exhaust the account's Lambda concurrency quota and throttle production, an over-broad IAM policy in staging can reach production data, and Cost Explorer can only separate them as well as their tags are maintained. Separating environments inside one account forces every stack ID to carry an environment suffix and every resource name to avoid collisions, which leaks environment logic throughout the infrastructure code. Separate accounts give hard isolation for free and let the same CDK app — with the same stack IDs — deploy into each one, with only a small, explicit configuration object varying per environment.

## Decision Drivers

* A failure, quota exhaustion, or runaway cost in a non-production environment must not be able to affect production
* A security mistake in a non-production environment must not grant access to production resources
* Per-environment cost must be visible from the account boundary alone, without relying on tag hygiene
* Infrastructure code should not need environment suffixes on stack IDs or resource names to avoid collisions
* Everything that differs between environments should be visible in one place rather than scattered through stacks as conditionals
* Deploying the wrong environment's configuration into an account must fail loudly, not silently
* Shared environments must not depend on long-lived access keys or on whatever happens to be on a developer's machine

## Considered Options

* Account per environment, environment selected by credentials
* One account, environments separated by stack-name suffixes

## Decision Outcome

Chosen option: "Account per environment", because the account boundary gives hard isolation of blast radius, quotas, permissions, and cost with no environment logic in stack IDs or resource names, and lets one CDK app deploy unchanged into every environment.

### Layout

| Concern | Convention | Olatile example |
|---|---|---|
| Account | One per environment | `olatile-prod` |
| Environment names | `dev`, `staging`, `prod` — `prod` always; others only as needed | `prod` |
| CDK app | `infra/app.ts`, one per deployable repository | `olatile-api/infra/app.ts` |
| Stacks | `infra/stacks/{concern}-stack.ts`, ID `{Service}{Concern}Stack` | `ApiComputeStack`, `ApiWafStack` |
| Account/region | From credentials, never hardcoded | `CDK_DEFAULT_ACCOUNT` |
| Environment tag | `x:env` equals the `env` context value (see ADR-AWS-0003) | `x:env=prod` |
| Deploy role | One GitHub OIDC role per account | `AWS_GITHUB_ROLE` secret |

Split stacks by concern and lifecycle, keeping stateful resources (secrets, log groups, data stores, certificates) out of the stack that holds frequently replaced compute, so compute can be torn down or rebuilt without touching state.

### Examples

`infra/app.ts` — one app, all environment differences in one config object:

```typescript
import { App, Stack, Tags } from "aws-cdk-lib";

const CONFIG = {
  staging: { domain: "staging.example.com" },
  prod: { domain: "example.com" },
};

const app = new App();

const envName = app.node.tryGetContext("env");
if (!Object.hasOwn(CONFIG, envName)) {
  throw new Error(`Unknown env "${envName}"`);
}
const config = CONFIG[envName as keyof typeof CONFIG];

const env = {
  account: process.env.CDK_DEFAULT_ACCOUNT,
  region: process.env.CDK_DEFAULT_REGION,
};

new Stack(app, "ApiComputeStack", { env, description: `API for ${config.domain}` });

Tags.of(app).add("x:env", envName);

app.synth();
```

The stack ID is `ApiComputeStack` in every account; only `config` varies by environment.

Deploying — the environment is chosen at dispatch, and each GitHub Environment holds that account's role and region:

```yaml
on:
  workflow_dispatch:
    inputs:
      env:
        type: choice
        options: [staging, prod]

jobs:
  checks:
    uses: ./.github/workflows/build.yml

  deploy:
    needs: [checks]
    runs-on: ubuntu-latest
    environment: ${{ inputs.env }}
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v5
      - uses: aws-actions/configure-aws-credentials@v6
        with:
          role-to-assume: ${{ secrets.AWS_GITHUB_ROLE }}
          aws-region: ${{ vars.AWS_REGION }}
      - run: pnpm cdk deploy --all -c env=${{ inputs.env }} --require-approval never
```

The `prod` GitHub Environment's deployment branch policy allows only `main`, and the `olatile-prod` role's trust policy accepts only that environment, so a production deploy from any other branch fails at "Assume Role".

Not supported — environments separated by suffix inside one account:

```typescript
// Do not do this — every stack, and often every resource name, has to carry
// the environment, and staging shares production's quotas and blast radius.
new ComputeStack(app, `ApiComputeStack-${envName}`, { env });
```

### Consequences

* Good, because a non-production failure, quota exhaustion, or runaway cost cannot affect production
* Good, because per-environment cost is visible from the account alone
* Good, because stack IDs and resource names carry no environment logic, and the same app deploys unchanged everywhere
* Good, because everything that differs between environments is reviewable in one config object
* Good, because deploy credentials are short-lived and scoped to one account
* Bad, because every new environment needs a new account, an OIDC role, and a GitHub Environment before its first deploy
* Bad, because resources genuinely shared across environments (e.g. a parent DNS zone) need cross-account delegation rather than a direct reference
* Neutral, because a developer may still deploy to a non-production account from their machine via their own SSO profile, but never to `prod`

### Confirmation

Code review must flag stack IDs or resource names that embed an environment name, hardcoded account IDs used to select the target account, environment conditionals inside stacks instead of in the app's config object, and deploy workflows that use long-lived access keys or can deploy `prod` from a branch other than `main`. Synth must fail when `env` is missing or unknown.

## Pros and Cons of the Options

### Account per environment

* Good, because hard isolation of blast radius, quotas, IAM, and cost at the account boundary
* Good, because identical stack IDs across environments with no naming suffixes
* Bad, because more accounts, roles, and CI environments to create and maintain

### One account, environment suffixes

* Good, because fewer accounts to manage
* Bad, because environments share quotas, so staging load can throttle production
* Bad, because every stack ID and many resource names must carry the environment to avoid collisions
* Bad, because per-environment cost depends entirely on tag hygiene

## More Information

* Related: [ADR-AWS-0001 — Use CDK](ADR-AWS-0001-use-cdk.md)
* Related: [ADR-AWS-0002 — Use CDK Nag](ADR-AWS-0002-use-cdk-nag.md)
* Related: [ADR-AWS-0003 — Tag All Resources](ADR-AWS-0003-tag-all-resources.md)
* Reference implementation: `infra/` and `.github/workflows/deploy.yml` in xcklein/olatile-api
* [AWS — Organizing your AWS environment using multiple accounts](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html)
