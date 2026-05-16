---
status: accepted
date: 2026-05-16
tags: [aws, cdk, tagging, governance, cost]
---
# Tag All Resources

## Directive

All AWS resources must be tagged with `x:repo`, `x:service`, and `x:env` using `cdk.Tags.of(this)` at the stack level. Additional tags may be added as needed.

## Context and Problem Statement

AWS accounts accumulate resources across environments, teams, and services. Without consistent tagging, it is impossible to attribute costs, identify resource ownership, or enforce lifecycle policies programmatically. Tags are the foundational mechanism for governance, cost allocation, and operational visibility in AWS.

## Decision Drivers

* Cloud costs must be attributable to specific services, teams, or environments
* Resource ownership must be identifiable without inspecting deployment history
* Lifecycle policies (e.g., automated cleanup of ephemeral environments) must be enforceable via tags
* Compliance and audit requirements may require environment and classification metadata on resources
* Custom tags must be visually distinct from AWS-generated tags and group together alphabetically in the AWS console

## Considered Options

* Mandatory tags enforced via CDK and cdk-nag
* Tags applied per resource manually
* No tagging standard

## Decision Outcome

Chosen option: "Mandatory tags enforced via CDK and cdk-nag", because tags are applied consistently at the stack level through CDK's `Tags` API and violations are caught at synth time by cdk-nag, preventing untagged resources from being deployed.

### Examples

Apply mandatory tags to all resources in a stack via the CDK `Tags` API:

```typescript
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

export class MyStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    cdk.Tags.of(this).add('x:repo', 'my-repo');
    cdk.Tags.of(this).add('x:service', 'payments');
    cdk.Tags.of(this).add('x:env', 'production');
  }
}
```

Required tags for all resources:

All custom tag keys are prefixed with `x:`. The prefix serves two purposes: it groups custom tags together alphabetically in the AWS console (sorting after AWS-generated tags), and it makes custom tags visually distinct from tags applied by AWS services.

Required tags:

| Tag | Description | Example |
|---|---|---|
| `x:repo` | Source repository for this stack | `my-repo`, `platform-infra` |
| `x:service` | The service or application this resource belongs to | `payments`, `auth`, `data-pipeline` |
| `x:env` | Deployment environment | `production`, `staging`, `development` |

Additional tags (optional, as needed):

| Tag | Description | Example |
|---|---|---|
| `x:owner` | Team or individual responsible for the resource | `platform-team`, `data-eng` |
| `x:managed-by` | Deployment mechanism | `cdk` |

### Consequences

* Good, because cost allocation reports can be filtered and grouped by `Service` and `Environment`
* Good, because resource ownership is immediately identifiable without consulting deployment history
* Good, because CDK applies tags to the entire stack hierarchy — no per-resource annotation required
* Good, because cdk-nag can enforce required tag presence at synth time
* Bad, because some AWS resource types do not support tagging — these must be explicitly documented as exceptions
* Bad, because tag values must be kept consistent across stacks — drift in naming (e.g., `prod` vs `production`) reduces usefulness

### Confirmation

cdk-nag's `AwsSolutionsChecks` flags untagged resources. Required tags are applied via `cdk.Tags.of(this)` at the stack level in every CDK stack. Tag key names and allowed values are enforced via a shared CDK construct library.

## Pros and Cons of the Options

### Mandatory tags enforced via CDK and cdk-nag

* Good, because tags are applied once at the stack level — not per resource
* Good, because violations are caught at synth time before deployment
* Good, because the tag schema is version-controlled and auditable
* Bad, because requires a shared construct or convention for tag key names and values

### Tags applied per resource manually

* Good, because maximum flexibility per resource
* Bad, because inconsistent — different engineers apply different keys and values
* Bad, because easy to forget on individual resources
* Bad, because not enforceable without additional tooling

### No tagging standard

* Good, because zero overhead
* Bad, because cost attribution is impossible
* Bad, because resource ownership becomes opaque over time
* Bad, because lifecycle automation cannot be implemented reliably

## More Information

* [AWS Tagging Best Practices](https://docs.aws.amazon.com/general/latest/gr/aws_tagging.html)
* Related: [ADR-AWS-0001 — Use CDK](ADR-AWS-0001-use-cdk.md)
* Related: [ADR-AWS-0002 — Use cdk-nag](ADR-AWS-0002-use-cdk-nag.md)
