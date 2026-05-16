---
status: accepted
date: 2026-05-16
tags: [aws, cdk, security, compliance]
---
# Use cdk-nag

## Directive

Every CDK app entry point must apply `AwsSolutionsChecks` via `Aspects.of(app)`. The CDK synth step must fail on unsuppressed violations. All suppressions must include a `reason`.

## Context and Problem Statement

CDK stacks can be synthesised and deployed with security misconfigurations that violate AWS best practices and compliance requirements. These issues are often invisible during development and only surfaced post-deployment through security audits or incidents. cdk-nag is a CDK aspect that statically analyses stacks at synth time against rule packs based on established security frameworks, catching violations before deployment.

## Decision Drivers

* Security misconfigurations must be caught before infrastructure is deployed
* Compliance with AWS best practices must be enforced automatically, not manually audited
* Security checks must integrate into the existing CDK workflow with minimal friction
* All CDK stacks must be covered consistently

## Considered Options

* cdk-nag
* AWS Config rules (post-deployment)
* Manual CDK code review
* Checkov

## Decision Outcome

Chosen option: "cdk-nag", because it integrates directly into the CDK synth process, catching violations at development time before any deployment occurs.

### Examples

Apply cdk-nag to a stack in `bin/app.ts`:

```typescript
import * as cdk from 'aws-cdk-lib';
import { AwsSolutionsChecks } from 'cdk-nag';
import { Aspects } from 'aws-cdk-lib';
import { MyStack } from '../lib/my-stack';

const app = new cdk.App();
const stack = new MyStack(app, 'MyStack');

Aspects.of(app).add(new AwsSolutionsChecks({ verbose: true }));
```

Suppress a rule where a violation is intentional and accepted:

```typescript
import { NagSuppressions } from 'cdk-nag';

NagSuppressions.addResourceSuppressions(bucket, [
  {
    id: 'AwsSolutions-S1',
    reason: 'Server access logging not required for this ephemeral bucket.',
  },
]);
```

`cdk synth` will fail if unsuppressed violations are present.

### Consequences

* Good, because violations are caught at synth time — before any deployment occurs
* Good, because AwsSolutionsChecks covers a broad set of AWS security best practices out of the box
* Good, because suppressions are explicit and documented in code, creating an audit trail for accepted risks
* Good, because integrates seamlessly with existing CDK workflows and CI pipelines
* Bad, because some rules produce false positives that require suppression, adding noise in early adoption
* Bad, because teams must learn which rules apply to their context and how to write valid suppressions

### Confirmation

`AwsSolutionsChecks` must be applied via `Aspects.of(app)` in every CDK app entry point. CI runs `cdk synth` and fails the build on unsuppressed violations.

## Pros and Cons of the Options

### cdk-nag

* Good, because catches violations at synth time, before deployment
* Good, because integrates natively with CDK as an Aspect
* Good, because suppressions are code-level and auditable
* Good, because multiple rule packs available (AWS Solutions, HIPAA, PCI, NIST)
* Bad, because some rules require suppression for legitimate use cases
* Bad, because only covers CDK-managed infrastructure

### AWS Config rules (post-deployment)

* Good, because monitors live infrastructure continuously
* Good, because catches drift from desired state after deployment
* Bad, because violations are only detected after resources are deployed
* Bad, because remediation requires a redeployment cycle
* Bad, because adds cost for Config rule evaluations

### Manual CDK code review

* Good, because no tooling required
* Bad, because inconsistent — dependent on reviewer knowledge and attention
* Bad, because does not scale across teams or stacks
* Bad, because provides no audit trail

### Checkov

* Good, because broad multi-cloud support beyond AWS
* Good, because supports CloudFormation templates output by CDK synth
* Bad, because operates on synthesised CloudFormation, not CDK constructs — errors are harder to trace back to source
* Bad, because requires a separate tool outside the CDK workflow

## More Information

* [cdk-nag on GitHub](https://github.com/cdklabs/cdk-nag)
* [AWS Solutions rule pack](https://github.com/cdklabs/cdk-nag/blob/main/RULES.md)
* Related: [ADR-AWS-0001 — Use CDK](ADR-AWS-0001-use-cdk.md)
