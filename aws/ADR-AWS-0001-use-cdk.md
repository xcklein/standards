---
status: accepted
date: 2026-05-16
tags: [aws, infrastructure, cdk]
---
# Use CDK

## Directive

All infrastructure must be defined and deployed using AWS CDK. Infrastructure must not be created or modified manually via the AWS console or CLI.

## Context and Problem Statement

CDK enables all infrastructure to be audited via version control. It also provides a means of enforcing standards through shared constructs and libraries, and grouping resources into logical stacks.

## Decision Drivers

* All infrastructure changes must be auditable and version-controlled
* Standards and guardrails must be enforceable across teams
* Resources should be organized into logical, reusable stacks
* Infrastructure should be expressed in a general-purpose programming language to enable abstraction and reuse

## Considered Options

* AWS CDK
* Terraform
* AWS CloudFormation (raw)

## Decision Outcome

Chosen option: "AWS CDK", because it is the first-class AWS infrastructure framework.

### Consequences

* Good, because all infrastructure changes are version-controlled and auditable
* Good, because standards can be enforced through shared constructs and libraries
* Good, because resources are grouped into logical stacks, improving visibility and manageability
* Good, because a general-purpose language enables abstraction, reuse, and testing of infrastructure
* Bad, because CDK adds a synthesis step before deployment
* Bad, because developers unfamiliar with the CDK language face a steeper learning curve than declarative tools like Terraform

### Confirmation

IAM permissions are restricted to prevent infrastructure changes outside of CDK deployments.

## Pros and Cons of the Options

### AWS CDK

* Good, because infrastructure is expressed in a general-purpose language enabling abstraction and reuse
* Good, because first-class AWS support with constructs for every service
* Good, because enables sharing of standards through construct libraries
* Bad, because requires a synthesis step before deployment
* Bad, because steeper learning curve for developers unfamiliar with the chosen language

### Terraform

* Good, because infrastructure is expressed in declarative HCL, approachable for ops-focused teams
* Good, because not AWS-specific — supports multi-cloud and hybrid environments
* Good, because mature ecosystem with broad provider support
* Bad, because limited ability to enforce organisation-specific standards programmatically
* Bad, because not a first-class AWS tool — lags behind new service support

### AWS CloudFormation (raw)

* Good, because no additional tooling required
* Good, because native AWS deployment engine (CDK synthesises to it anyway)
* Bad, because verbose YAML/JSON with no abstraction or reuse primitives
* Bad, because impossible to enforce standards beyond basic resource policies
