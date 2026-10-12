# Detailed threat model design

Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.  SPDX-License-Identifier: CC-BY-4.0

## Threat model structure

`<service>.json` is a service threat model with full metadata, operations, threats, mitigations, and control implementations (each service's report is linked from the [release index](../threat-models-for-aws-services/README.md); its JSON lives under `data/services/`, listed in `manifest.json`). It has the following schema, described in more depth in the [threat model schema](../threat-models-for-aws-services/schema/threat-model.schema.json):

```json
{
  "metadata": { ... },
  "operations": [ ... ],
  "entities": [ ... ],
  "mitigations": [ ... ],
  "threats": [ ... ],
  "implementations": [ ... ]
}
```

**Metadata** - Top-level service information including the service name, identifiers (IAM namespace, CloudFormation prefix, AWS Config resource type prefix), a description of the service's capabilities, common use cases, and relationships to other AWS service namespaces.

**Operations** – AWS service API operations derived from [AWS service Smithy models](https://github.com/aws/api-models-aws) and analyzed in the threat model.

The `analyses` field on each operation records which types of analysis were performed for that operation. An operation may carry more than one analysis type.

- `DOCUMENTATION` – the operation was analyzed using the service's public documentation and its service model. Every operation in a threat model carries this value.
- `DATA_PERIMETER` – the operation was also covered by an assessment of the service against [data perimeter](https://aws.amazon.com/identity/data-perimeters-on-aws/) control objectives. If this assessment produced an observation relevant to the threat model scope, that observation is published as a supplemental context entry of type `DATA_PERIMETER` on the affected threat or implementation.

**Entities** – A set of resources that can be created in or retrieved from an AWS service using service APIs, derived from the [Service Authorization Reference](https://docs.aws.amazon.com/service-authorization/latest/reference/reference.html), and service operations that can create, modify, or act on them.

**Mitigations** – A set of mitigations identified for service resources, drawn from the mitigation goals catalog ([Markdown](../threat-models-for-aws-services/reports/mitigation-goals.md), [JSON](../threat-models-for-aws-services/data/mitigation-goals.json)). Each mitigation is a specific configuration of an AWS service resource that addresses one of the common security and governance objectives.

Mitigations are control objectives that can be achieved by enforcing specific values or constraints on AWS service API request parameters.

Mitigations include references to the specific service APIs and parameters that govern the configuration of a service resource relevant to the mitigation, as well as APIs for assessing whether the secure configuration is in place.

Mitigations are mapped to threats to inform security and governance decisions related to an AWS service resource and to provide context for related control intents. Each mitigation may map to multiple control implementations that help achieve it.

**Threats** - A set of threats, drawn from the potential threats catalog ([Markdown](../threat-models-for-aws-services/reports/potential-threats.md), [JSON](../threat-models-for-aws-services/data/potential-threats.json)), that are relevant to the identified mitigations. Each threat is a structured statement about an IAM principal or threat actor’s interaction with an AWS service resource that may have unwanted impacts on the security of a system.

Each threat is a conditional statement, not an assertion about your environment. It has three parts:

- `source` – the actor: an IAM principal, or a threat actor acting outside IAM authorization.
- `prerequisites` – what must be true for the action to be available, such as a permission the actor holds or a configuration the resource carries.
- `threatAction` – what the actor can then do.

The action states what becomes possible once the prerequisites hold. It does not claim that your resources are in that state, that every attempt succeeds, or that no other control stands in the way.

A threat does not necessarily enumerate every condition required for it to materialize. The resource conditions it names are the ones governed by the mitigations it maps to. Other conditions and qualifying factors may lower its likelihood or make it inapplicable to you. For example, a threat about uncontrolled internet egress through public-subnet routing maps to the mitigation for placing the resource in a private subnet. Security group rules also govern internet reachability, but they fall under a separate mitigation. So the threat does not claim that subnet placement is the only condition. Read each threat together with the mitigations mapped to it, then decide whether the threat applies to your use case and risk context.

Threats include references to the specific service APIs and parameters that can be misused for the threat to be realized. Each threat is mapped to an impact from the threat impacts catalog ([Markdown](../threat-models-for-aws-services/reports/threat-impacts.md), [JSON](../threat-models-for-aws-services/data/threat-impacts.json)). Each impact names its MITRE ATT&CK counterpart as framework mappings: the tactic it corresponds to, or, for an impact that is an Impact-tactic technique, both the Impact tactic and the technique.

The absence of a threat in a model does not mean the threat does not exist - it may simply be outside the current assessment scope, which is centered on identifying relevant mitigations and then mapping them to associated threats.

**Implementations** – A set of implementations that help achieve identified mitigations and provide ongoing governance for your AWS environment. Each implementation provides guidance for implementing a technical control with a specific control behavior. Whenever available, implementations reference managed controls. In other cases, they contain the information required to build a custom control, such as resource configuration properties or deployment parameters to monitor or govern.

There are four types of control behaviors:

- Preventive - A preventive control helps prevent actions that lead to deviations from identified mitigations. Preventive controls are implemented using [service control policies (SCPs)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html), [resource control policies (RCPs)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_rcps.html), [resource-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html), [IAM role trust policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html#id_roles_terms-and-concepts), and [declarative policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_declarative_policies.html).
- Detective - A detective control detects actions or resource configurations that may lead to deviations from identified mitigations. Detective controls are implemented using AWS CloudTrail [management events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-management-events-with-cloudtrail.html), [data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html), [network activity events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-network-events-with-cloudtrail.html), and [AWS Config rules](https://docs.aws.amazon.com/config/latest/developerguide/evaluate-config.html).
- Proactive - A proactive control scans resources before they are provisioned and helps ensure they comply with identified mitigations. Proactive controls are implemented using [AWS CloudFormation Hooks](https://docs.aws.amazon.com/cloudformation-cli/latest/hooks-userguide/what-is-cloudformation-hooks.html).
- Directive - Guidance that directs implementing an administrative control (policy or process), a technical solution, or other technical instructions to achieve the mitigation.

### Implementation mechanisms

Each implementation names its mechanism in `implementationType`. For a custom control, `implementationGuide` contains the details needed to build it. For a managed control, `managedImplementation` identifies the [AWS Control Catalog](https://docs.aws.amazon.com/controlcatalog/latest/userguide/what-is-controlcatalog.html) control instead. The mechanisms, grouped by control behavior, are:

**Preventive**

- `AWS::Organizations::Policy::SERVICE_CONTROL_POLICY` – A service control policy is an AWS Organizations policy that sets the maximum available permissions for IAM principals in member accounts. It governs which API actions those principals can call and under which request conditions. The `authorizationPolicy` guide contains the IAM actions, resources, and condition keys for each statement of the resulting policy.
- `AWS::Organizations::Policy::RESOURCE_CONTROL_POLICY` – A resource control policy is an AWS Organizations policy that sets the maximum available permissions for resources in member accounts. It governs who can access those resources and under which request conditions. The `authorizationPolicy` guide contains the IAM actions, resources, and condition keys for each statement of the resulting policy.
- `AWS::*::*::RESOURCE_POLICY` – A resource-based policy is attached to the resource itself, such as a bucket, key, queue, or VPC endpoint. It governs which principals can perform which actions on that resource, including principals in other accounts. The `authorizationPolicy` guide contains the IAM actions, resources, and condition keys for each statement of the resulting policy. `governedResourceType` names the resource type the policy attaches to.
- `AWS::IAM::Role::TRUST_POLICY` – An IAM role trust policy is the resource-based policy attached to a role. It governs which principals can assume the role. The `roleTrustPolicy` guide contains the principal, IAM actions, and condition keys for each statement of the resulting policy.
- `AWS::Organizations::Policy::DECLARATIVE_POLICY` – A declarative policy is an AWS Organizations policy that declares and enforces a configuration baseline for an AWS service across member accounts. The service enforces it in its control plane rather than through API authorization, so the baseline holds as the service adds features or APIs. The `declarativePolicy` guide contains the policy type and the JSONPaths of the attributes the mitigation governs.

**Detective**

- `AWS::Config::ConfigRule` – An AWS Config rule evaluates whether the recorded configuration of your resources meets a condition. The `configRule` guide contains the rule type (periodic or change-triggered), the configuration item properties to evaluate, and any API operations to call. API operations are ordered first call to last, each with the request parameters to send and the response properties to check.
- `AWS::CloudTrail::Trail::MANAGEMENT_EVENT` – A CloudTrail management event records a control plane operation performed on a resource, such as creating or reconfiguring it. Trails and event data stores log management events by default. The `cloudtrailEvent` guide contains the operations to monitor and any request parameters that narrow the match. These are typically the operations that can weaken or remove the mitigation.
- `AWS::CloudTrail::Trail::DATA_EVENT` – A CloudTrail data event records a data plane operation performed on or within a resource, such as reading an object or invoking a function. Data events are not logged by default, are enabled per resource type, and are available only for resource types CloudTrail supports. The `cloudtrailEvent` guide contains the operations to monitor, typically the data plane operations the mapped threats describe.
- `AWS::CloudTrail::Trail::NETWORK_ACTIVITY_EVENT` – A CloudTrail network activity event records an API call made to a service through a VPC endpoint, including the identity that made the call. Network activity events are not logged by default and are enabled per service, for the services that support them. The `cloudtrailEvent` guide contains the operations to monitor at the endpoint.

**Proactive**

- `AWS::CloudFormation::Type::HOOK` – An AWS CloudFormation Hook inspects a resource's configuration before CloudFormation provisions it and can fail the operation. A hook implemented with a Lambda function runs your code, which can call describe operations to resolve configuration parameters of resources the template references. The `cloudformationHook` guide contains the CloudFormation resource type and the property paths to evaluate. Where a referenced resource must be resolved, it also contains the chain of API operations to call with the response properties to check.

**Directive**

- `AWS::*::*::GUIDANCE` – Guidance is a recommendation that no policy or rule enforces, such as an administrative process or an architectural choice. The `guidance` guide contains the recommendation with supporting documentation links.

The implementations assume that certain foundational capabilities are already in place in your environment. Detective controls assume that AWS CloudTrail is enabled with the appropriate event type (management, data, or network activity events) and that you have the ability to discover and assess the configuration of your resources. Organization-level preventive controls assume an AWS Organizations structure aligned with your security objectives. Proactive controls assume that resources are provisioned through infrastructure as code. CloudFormation hooks are only effective when resources are deployed through CloudFormation or the AWS Cloud Control API, not directly through a service API or the console. Pair them with detective and preventive implementations, which apply regardless of how a change was made, or provision resources exclusively through CloudFormation or the AWS Cloud Control API.

A mitigation may link to multiple control implementations across different control behaviors. Review the implementations mapped to each mitigation and select those that align with your control strategy and operating model - you may apply more than one control behavior to strengthen coverage. If a managed control is not available or you prefer to use a different implementation mechanism, build it using the provided implementation guidance and validate it in a non-production environment (such as a test account or OU) before enforcing it broadly. Confirm that preventive controls block only intended actions, detective controls surface true deviations without excessive noise, and proactive controls stop non-compliant resources at provisioning time. Once the control behaves as expected, progressively roll it out to broader scopes - monitoring for unintended impact at each stage - until it is fully deployed.

## Supplemental context

Threats, mitigations, and implementations carry short explanatory statements alongside their structured data, in the `threatContext`, `mitigationContext`, and `implementationContext` fields respectively. Each entry has a `type` that classifies the statement:

- `SCOPING` – A mitigation's scoping statement names the capability the mitigation goal governs on the entity and the objective of governing it. A threat's scoping statement describes the exposure the threat's mapped mitigations address; it assumes those mitigations are not applied and makes no claim about your resources. An implementation's scoping statement explains how the implementation achieves a mitigation; an implementation carries one scoping statement per mitigation it implements. Each entry carries the id of the mitigation goal, potential threat, or mitigation its statement derives from (`mitigationGoalId`, `potentialThreatId`, `mitigationId`).
- `DATA_PERIMETER` – observations and guidance derived from the service entity's analysis against data perimeter control objectives.
- `REFERENCE` – general guidance and documentation links for this class of mitigation, threat, or implementation, shared across services rather than specific to a particular object within a service threat model.

Managed control implementations carry no supplemental context; their explanatory text is in the `title` and `description` of `managedImplementation`.