# Amazon Bedrock AgentCore Data Plane (bedrock-agentcore) Threat Model

[Threat Summary](#threat-summary) · [Threats](#threats) · [Mitigations](#mitigations) · [Implementations](#implementations) · [Appendix](#appendix)

## Overview

Amazon Bedrock AgentCore is an agentic platform for building, deploying, and operating AI agents at scale. It works with popular open-source agent frameworks and with foundation models hosted in or outside Amazon Bedrock. It helps teams avoid building and managing specialized agent infrastructure.

24 threats and 15 mitigations across 21 resource types and 67 API operations. 7 of 38 mitigation goals are covered, implemented by 24 implementations.

Source service model: [bedrock-agentcore](https://github.com/aws/api-models-aws/blob/3d89dd029ebd13b775ab5d22e5075399f20b35e4/models/bedrock-agentcore/service/2024-02-28/bedrock-agentcore-2024-02-28.json)

---

## Threat Summary

### [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.BatchEvaluation.1](#threatbedrock-agentcorebatchevaluation1) | IAM principal, with IAM permissions to create an Amazon Bedrock AgentCore batch evaluation, can encrypt the batch evaluation's stored artifacts, such as per-session ground truth, evaluation results, and insight results, with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1) |
| [Threat.BEDROCK-AGENTCORE.Recommendation.1](#threatbedrock-agentcorerecommendation1) | IAM principal, with IAM permissions to create an Amazon Bedrock AgentCore recommendation, can encrypt the recommendation's input data and AI-generated results with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1) |

### [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.AgentRuntime.2](#threatbedrock-agentcoreagentruntime2) | IAM principal, with IAM permissions to use an Amazon Bedrock AgentCore agent runtime, can interact with the agent runtime without a record of their activities being persisted by application and trace log delivery. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2) |
| [Threat.BEDROCK-AGENTCORE.CodeInterpreter.1](#threatbedrock-agentcorecodeinterpreter1) | IAM principal, with IAM permissions to use an Amazon Bedrock AgentCore code interpreter, can interact with the code interpreter without a record of their activities being persisted by application and trace log delivery to CloudWatch Logs. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1) |
| [Threat.BEDROCK-AGENTCORE.Memory.2](#threatbedrock-agentcorememory2) | IAM principal, with IAM permissions to use an Amazon Bedrock AgentCore memory, can interact with the memory without a record of their activities being persisted by application and trace log delivery. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2) |

### [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.1](#threatbedrock-agentcore1) | Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon Bedrock AgentCore service API calls. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2) |

### [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.AbTest.2](#threatbedrock-agentcoreabtest2) | Untrusted IAM principal, that has been granted access to my Amazon Bedrock AgentCore A/B test's IAM execution role through its trust policy, can assume my execution role and act with its permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1) |

### [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.BatchEvaluation.3](#threatbedrock-agentcorebatchevaluation3) | IAM principal, with IAM permissions to create an Amazon Bedrock AgentCore batch evaluation, can encrypt its output artifacts with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail. | [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1) |
| [Threat.BEDROCK-AGENTCORE.Recommendation.3](#threatbedrock-agentcorerecommendation3) | IAM principal, with IAM permissions to create an Amazon Bedrock AgentCore recommendation, can encrypt its configuration and result with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail. | [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1) |

### [Logging disablement](../potential-threats.md#logging-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.AgentRuntime.3](#threatbedrock-agentcoreagentruntime3) | IAM principal, with IAM permissions to modify an Amazon Bedrock AgentCore agent runtime, can disable application and trace log delivery for the agent runtime. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2) |
| [Threat.BEDROCK-AGENTCORE.CodeInterpreter.2](#threatbedrock-agentcorecodeinterpreter2) | IAM principal, with IAM permissions to modify an Amazon Bedrock AgentCore code interpreter, can disable application and trace log delivery for the code interpreter. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1) |
| [Threat.BEDROCK-AGENTCORE.Memory.3](#threatbedrock-agentcorememory3) | IAM principal, with IAM permissions to modify an Amazon Bedrock AgentCore memory, can disable application and trace log delivery for the memory. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2) |

### [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.AbTest.1](#threatbedrock-agentcoreabtest1) | IAM principal, with IAM permissions to create or modify an Amazon Bedrock AgentCore A/B test, can configure the A/B test without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.AbTest.2](#mitigationbedrock-agentcoreabtest2) |
| [Threat.BEDROCK-AGENTCORE.BatchEvaluation.2](#threatbedrock-agentcorebatchevaluation2) | IAM principal, with IAM permissions to create or modify an Amazon Bedrock AgentCore batch evaluation, can configure the batch evaluation without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.2](#mitigationbedrock-agentcorebatchevaluation2) |
| [Threat.BEDROCK-AGENTCORE.Recommendation.2](#threatbedrock-agentcorerecommendation2) | IAM principal, with IAM permissions to create or modify an Amazon Bedrock AgentCore recommendation, can configure the recommendation without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.BEDROCK-AGENTCORE.Recommendation.2](#mitigationbedrock-agentcorerecommendation2) |

### [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.2](#threatbedrock-agentcore2) | IAM principal, with IAM permissions on untrusted Amazon Bedrock AgentCore resources, can interact with those untrusted resources from a governed VPC network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1) |

### [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.AgentRuntime.1](#threatbedrock-agentcoreagentruntime1) | My IAM principal, with IAM permissions on an untrusted Amazon Bedrock AgentCore agent runtime, can interact with the untrusted agent runtime, including sending data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.1](#mitigationbedrock-agentcoreagentruntime1) |
| [Threat.BEDROCK-AGENTCORE.Event.1](#threatbedrock-agentcoreevent1) | My IAM principal, with IAM permissions on an untrusted Amazon Bedrock AgentCore memory, can interact with events in the untrusted memory, including sending data to them or receiving data from them. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.Event.1](#mitigationbedrock-agentcoreevent1) |
| [Threat.BEDROCK-AGENTCORE.Memory.1](#threatbedrock-agentcorememory1) | My IAM principal, with IAM permissions on an untrusted Amazon Bedrock AgentCore memory, can interact with the untrusted memory, including sending data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.Memory.1](#mitigationbedrock-agentcorememory1) |
| [Threat.BEDROCK-AGENTCORE.MemoryRecord.1](#threatbedrock-agentcorememoryrecord1) | My IAM principal, with IAM permissions on an untrusted Amazon Bedrock AgentCore memory, can interact with records in the untrusted memory, including sending data to them or receiving data from them. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1) |

### [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.BatchEvaluation.5](#threatbedrock-agentcorebatchevaluation5) | IAM principal, with unintended IAM permissions to read evaluation results and insight results from an Amazon Bedrock AgentCore batch evaluation that has been encrypted with an AWS owned KMS key, can read evaluation results and insight results in the batch evaluation. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1) |
| [Threat.BEDROCK-AGENTCORE.Recommendation.5](#threatbedrock-agentcorerecommendation5) | IAM principal, with unintended IAM permissions to read system prompts, tool descriptions, and trace-derived optimization results from an Amazon Bedrock AgentCore recommendation that has been encrypted with an AWS owned KMS key, can read system prompts, tool descriptions, and trace-derived optimization results in the recommendation. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1) |

### [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.BEDROCK-AGENTCORE.BatchEvaluation.4](#threatbedrock-agentcorebatchevaluation4) | IAM principal, with unintended IAM permissions to read evaluation and insight results from an Amazon Bedrock AgentCore batch evaluation that has not been encrypted at rest with a KMS customer managed key, can read plaintext evaluation and insight results in the batch evaluation. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1) |
| [Threat.BEDROCK-AGENTCORE.Recommendation.4](#threatbedrock-agentcorerecommendation4) | IAM principal, with unintended IAM permissions to read configuration and result data from an Amazon Bedrock AgentCore recommendation that has not been encrypted at rest with a KMS customer managed key, can read plaintext configuration and result data in the recommendation. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1) |

---

## Threats

Each scoping statement below describes the exposure the threat's mapped mitigations address. It assumes those mitigations are not applied and makes no claim about your resources.

### Threat.BEDROCK-AGENTCORE.1

Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon Bedrock AgentCore service API calls.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)): Amazon Bedrock AgentCore is reachable from your VPC through an interface VPC endpoint (powered by AWS PrivateLink). An IAM principal from outside your organization can use that network path to make Amazon Bedrock AgentCore API calls with its own credentials. That access enables transfer of data from your VPC to Amazon Bedrock AgentCore memories, agent runtimes, and code interpreters in an AWS account outside your organization.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:CreateEvent` | `$.memoryId`, `$.payload[*].blob`, `$.payload[*].conversational.content.text` |
| `bedrock-agentcore:InvokeAgentRuntime` | `$.accountId`, `$.agentRuntimeArn`, `$.payload` |
| `bedrock-agentcore:InvokeCodeInterpreter` | `$.arguments.content[*].text`, `$.codeInterpreterIdentifier`, `$.name` |
| `bedrock-agentcore:RetrieveMemoryRecords` | `$.memoryId`, `$.searchCriteria.searchQuery` |

### Threat.BEDROCK-AGENTCORE.2

IAM principal, with IAM permissions on untrusted Amazon Bedrock AgentCore resources, can interact with those untrusted resources from a governed VPC network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)): Amazon Bedrock AgentCore data plane API calls from your VPCs can target agent runtimes, memories, and other Amazon Bedrock AgentCore resources in another account. With permissions on such a resource, an IAM principal on your network can send your data to an agent runtime or write it into a memory as events and records.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:BatchCreateMemoryRecords` | `$.memoryId`, `$.records[*].content.text` |
| `bedrock-agentcore:CreateEvent` | `$.memoryId`, `$.payload[*].conversational.content.text` |
| `bedrock-agentcore:InvokeAgentRuntime` | `$.accountId`, `$.agentRuntimeArn`, `$.payload` |
| `bedrock-agentcore:RetrieveMemoryRecords` | `$.memoryId` |

### Threat.BEDROCK-AGENTCORE.AbTest.1

IAM principal, with IAM permissions to create or modify an Amazon Bedrock AgentCore A/B test, can configure the A/B test without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.AbTest.2](#mitigationbedrock-agentcoreabtest2)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon Bedrock AgentCore A/B test can omit the required tags, because `$.tags{key}` is optional on `CreateABTest`. An A/B test left without those tags still splits live traffic between variants and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:CreateABTest` | `$.tags{key}` |
| `bedrock-agentcore-control:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.BEDROCK-AGENTCORE.AbTest.2

Untrusted IAM principal, that has been granted access to my Amazon Bedrock AgentCore A/B test's IAM execution role through its trust policy, can assume my execution role and act with its permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1199](https://attack.mitre.org/techniques/T1199/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon Bedrock AgentCore A/B test designates an IAM execution role through `$.roleArn`, and you author its trust policy. That trust policy can allow identities outside your organization to assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:CreateABTest` | `$.roleArn` |
| `bedrock-agentcore:UpdateABTest` | `$.roleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.BEDROCK-AGENTCORE.AgentRuntime.1

My IAM principal, with IAM permissions on an untrusted Amazon Bedrock AgentCore agent runtime, can interact with the untrusted agent runtime, including sending data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.1](#mitigationbedrock-agentcoreagentruntime1)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): An agent runtime in an account outside your organization can grant your principals access through resource-based policies on both the runtime and its endpoint. Your principals can then call `InvokeAgentRuntime` to send data in `$.payload`, or call `InvokeAgentRuntimeCommand` to run shell commands in the runtime's session container. Both calls exchange data with an environment that the runtime's owner controls, outside your governance.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:InvokeAgentRuntime` | `$.accountId`, `$.agentRuntimeArn`, `$.payload`, `$.qualifier` |
| `bedrock-agentcore:InvokeAgentRuntimeCommand` | `$.accountId`, `$.agentRuntimeArn`, `$.body.command` |

### Threat.BEDROCK-AGENTCORE.AgentRuntime.2

IAM principal, with IAM permissions to use an Amazon Bedrock AgentCore agent runtime, can interact with the agent runtime without a record of their activities being persisted by application and trace log delivery.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When application and trace log delivery are not configured on your Amazon Bedrock AgentCore agent runtime, no per-invocation application log records or `InvokeAgentRuntime` spans are written by application and trace log delivery. An IAM principal with permissions to use the agent runtime can send request payloads to the agent runtime by invoking it, leaving no record of those request payloads or the calls that sent them persisted by application and trace log delivery.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:InvokeAgentRuntime` | `$.X-Amzn-Bedrock-AgentCore-Runtime-Session-Id`, `$.payload`, `$.qualifier` |

### Threat.BEDROCK-AGENTCORE.AgentRuntime.3

IAM principal, with IAM permissions to modify an Amazon Bedrock AgentCore agent runtime, can disable application and trace log delivery for the agent runtime.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to modify your Amazon Bedrock AgentCore agent runtime can use CloudWatch Logs permissions to call `DeleteDelivery`, then `DeleteDeliverySource`. Removing a delivery stops the log type it carries, the opt-in per-invocation application log records or the `InvokeAgentRuntime` spans, from reaching CloudWatch Logs while your agent runtime keeps serving `InvokeAgentRuntime` and `InvokeAgentRuntimeCommand` calls.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:DeleteDelivery` | `$.id` |
| `cloudwatch-logs:DeleteDeliverySource` | `$.name` |

### Threat.BEDROCK-AGENTCORE.BatchEvaluation.1

IAM principal, with IAM permissions to create an Amazon Bedrock AgentCore batch evaluation, can encrypt the batch evaluation's stored artifacts, such as per-session ground truth, evaluation results, and insight results, with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal able to start a batch evaluation chooses the KMS key that encrypts the batch evaluation's stored artifacts by setting `$.kmsKeyArn`. If that key material sits outside your control, its holder can later revoke or delete it and leave those artifacts unreadable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartBatchEvaluation` | `$.kmsKeyArn` |

### Threat.BEDROCK-AGENTCORE.BatchEvaluation.2

IAM principal, with IAM permissions to create or modify an Amazon Bedrock AgentCore batch evaluation, can configure the batch evaluation without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.2](#mitigationbedrock-agentcorebatchevaluation2)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon Bedrock AgentCore batch evaluation can omit the required tags, because `$.tags{key}` on `StartBatchEvaluation` is optional. A batch evaluation left without those tags still holds its evaluation results and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartBatchEvaluation` | `$.tags{key}` |

### Threat.BEDROCK-AGENTCORE.BatchEvaluation.3

IAM principal, with IAM permissions to create an Amazon Bedrock AgentCore batch evaluation, can encrypt its output artifacts with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal able to create your Amazon Bedrock AgentCore batch evaluation chooses whether the AWS owned key protects its output artifacts at rest. You cannot govern that key's policy or lifecycle, and the service's use of it produces no CloudTrail events in your account.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartBatchEvaluation` | `$.kmsKeyArn` |

### Threat.BEDROCK-AGENTCORE.BatchEvaluation.4

IAM principal, with unintended IAM permissions to read evaluation and insight results from an Amazon Bedrock AgentCore batch evaluation that has not been encrypted at rest with a KMS customer managed key, can read plaintext evaluation and insight results in the batch evaluation.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1)

**Frameworks:** MITRE ATT&CK [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)): Without a customer managed key, your batch evaluation's output artifacts, including evaluation and insight results, are encrypted under an AWS owned key that you cannot manage or audit. An IAM principal holding the permission to call `GetBatchEvaluation` can read those results in plaintext.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:GetBatchEvaluation` | `$.batchEvaluationId` |
| `bedrock-agentcore:ListBatchEvaluations` | *(any parameter)* |
| `bedrock-agentcore:StartBatchEvaluation` | `$.kmsKeyArn` |

### Threat.BEDROCK-AGENTCORE.BatchEvaluation.5

IAM principal, with unintended IAM permissions to read evaluation results and insight results from an Amazon Bedrock AgentCore batch evaluation that has been encrypted with an AWS owned KMS key, can read evaluation results and insight results in the batch evaluation.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)): By default, your Amazon Bedrock AgentCore batch evaluation's evaluation results and insight results are encrypted at rest with an AWS owned key. That key type gives you no key policy to govern, no audit record of key use, and no control over the key lifecycle, so an IAM principal granted broader permissions than intended can retrieve them in plaintext.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:GetBatchEvaluation` | `$.batchEvaluationId` |

### Threat.BEDROCK-AGENTCORE.CodeInterpreter.1

IAM principal, with IAM permissions to use an Amazon Bedrock AgentCore code interpreter, can interact with the code interpreter without a record of their activities being persisted by application and trace log delivery to CloudWatch Logs.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When application and trace log delivery are not configured on your Amazon Bedrock AgentCore code interpreter, no application logs or spans are written by application and trace log delivery to CloudWatch Logs. An IAM principal with permissions to use the code interpreter can run code, run shell commands, and access files, leaving no record of the code, commands, or file contents, or of the calls that ran them, persisted by application and trace log delivery.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:InvokeCodeInterpreter` | `$.arguments.code`, `$.arguments.command`, `$.arguments.content[*].path`, `$.arguments.paths[*]`, `$.name`, `$.x-amzn-code-interpreter-session-id` |

### Threat.BEDROCK-AGENTCORE.CodeInterpreter.2

IAM principal, with IAM permissions to modify an Amazon Bedrock AgentCore code interpreter, can disable application and trace log delivery for the code interpreter.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to modify your Amazon Bedrock AgentCore code interpreter can use CloudWatch Logs permissions to delete its log delivery with `DeleteDelivery` or overwrite its delivery source with `PutDeliverySource`. Either change stops the opt-in application logs or spans the affected delivery carries from reaching CloudWatch Logs while code execution, shell commands, and file operations in its sessions continue without those records.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:DeleteDelivery` | `$.id` |
| `cloudwatch-logs:DeleteDeliverySource` | `$.name` |
| `cloudwatch-logs:PutDeliverySource` | `$.logType`, `$.name`, `$.resourceArn` |

### Threat.BEDROCK-AGENTCORE.Event.1

My IAM principal, with IAM permissions on an untrusted Amazon Bedrock AgentCore memory, can interact with events in the untrusted memory, including sending data to them or receiving data from them.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Event.1](#mitigationbedrock-agentcoreevent1)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): An Amazon Bedrock AgentCore memory in an account outside your organization can grant your principals access through its resource-based policy. Your principals with IAM permissions on that memory can then call `CreateEvent` to write conversation text, JSON data, or binary content into storage that account controls. If that policy also grants read access, they can call `GetEvent` to retrieve events from that memory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:CreateEvent` | `$.memoryId`, `$.payload[*].blob`, `$.payload[*].conversational.content.text`, `$.payload[*].json.content` |
| `bedrock-agentcore:GetEvent` | `$.eventId`, `$.memoryId` |

### Threat.BEDROCK-AGENTCORE.Memory.1

My IAM principal, with IAM permissions on an untrusted Amazon Bedrock AgentCore memory, can interact with the untrusted memory, including sending data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Memory.1](#mitigationbedrock-agentcorememory1)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): An Amazon Bedrock AgentCore memory in an account outside your organization can grant your principals access through its resource-based policy. Your principals with IAM permissions on that memory can then call `IngestData` to send it conversational text in `$.source.inline.payload[*].conversational.content.text` or JSON in `$.source.inline.payload[*].json.content`.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:IngestData` | `$.memoryId`, `$.source.inline.payload[*].conversational.content.text`, `$.source.inline.payload[*].json.content` |

### Threat.BEDROCK-AGENTCORE.Memory.2

IAM principal, with IAM permissions to use an Amazon Bedrock AgentCore memory, can interact with the memory without a record of their activities being persisted by application and trace log delivery.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When application and trace log delivery are not configured on your Amazon Bedrock AgentCore memory, no application logs or spans are written by log delivery. An IAM principal with permissions to use the memory can interact with it, leaving no record of the long-term memory extraction and consolidation those interactions trigger, or of the calls themselves, persisted by log delivery.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:IngestData` | `$.actorId`, `$.extractionConfig.namespaceVariables{key}`, `$.sessionId`, `$.source.inline.payload[*].conversational.content.text`, `$.source.inline.payload[*].json.content` |
| `bedrock-agentcore:StartMemoryExtractionJob` | `$.extractionJob.jobId` |

### Threat.BEDROCK-AGENTCORE.Memory.3

IAM principal, with IAM permissions to modify an Amazon Bedrock AgentCore memory, can disable application and trace log delivery for the memory.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to modify your Amazon Bedrock AgentCore memory can call `DeleteDelivery` with the `$.id` of a log delivery for the memory, then call `DeleteDeliverySource` with the delivery source's `$.name`. That change stops the memory's opt-in application logs or spans, whichever delivery it removes, from reaching CloudWatch Logs.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:DeleteDelivery` | `$.id` |
| `cloudwatch-logs:DeleteDeliverySource` | `$.name` |

### Threat.BEDROCK-AGENTCORE.MemoryRecord.1

My IAM principal, with IAM permissions on an untrusted Amazon Bedrock AgentCore memory, can interact with records in the untrusted memory, including sending data to them or receiving data from them.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): An Amazon Bedrock AgentCore memory in an account outside your organization can grant your principals access through a resource-based policy. Your principals whose own permissions also cover that memory can then write your data into its records by calling `BatchCreateMemoryRecords` or `BatchUpdateMemoryRecords`. They can also read record content from that memory by calling `GetMemoryRecord` or `RetrieveMemoryRecords`.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:BatchCreateMemoryRecords` | `$.memoryId`, `$.records[*].content.text` |
| `bedrock-agentcore:BatchUpdateMemoryRecords` | `$.memoryId`, `$.records[*].content.text` |
| `bedrock-agentcore:GetMemoryRecord` | `$.memoryId` |
| `bedrock-agentcore:RetrieveMemoryRecords` | `$.memoryId` |

### Threat.BEDROCK-AGENTCORE.Recommendation.1

IAM principal, with IAM permissions to create an Amazon Bedrock AgentCore recommendation, can encrypt the recommendation's input data and AI-generated results with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal able to start a recommendation chooses the KMS key that encrypts your recommendation data by setting `$.kmsKeyArn`. If that key material sits outside your control, its holder can later revoke or delete it and leave that recommendation data unreadable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartRecommendation` | `$.kmsKeyArn` |

### Threat.BEDROCK-AGENTCORE.Recommendation.2

IAM principal, with IAM permissions to create or modify an Amazon Bedrock AgentCore recommendation, can configure the recommendation without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Recommendation.2](#mitigationbedrock-agentcorerecommendation2)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon Bedrock AgentCore recommendation can omit the required tags, because `$.tags{key}` on `StartRecommendation` is optional. A recommendation left without those tags still persists until you delete it and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartRecommendation` | `$.tags{key}` |

### Threat.BEDROCK-AGENTCORE.Recommendation.3

IAM principal, with IAM permissions to create an Amazon Bedrock AgentCore recommendation, can encrypt its configuration and result with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal able to create your Amazon Bedrock AgentCore recommendation chooses whether the AWS owned key protects its configuration and result at rest. You cannot govern that key's policy or lifecycle, and the service's use of it produces no CloudTrail events in your account.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartRecommendation` | `$.kmsKeyArn` |

### Threat.BEDROCK-AGENTCORE.Recommendation.4

IAM principal, with unintended IAM permissions to read configuration and result data from an Amazon Bedrock AgentCore recommendation that has not been encrypted at rest with a KMS customer managed key, can read plaintext configuration and result data in the recommendation.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)): Without a customer managed key, your recommendation's configuration and result, including trace content, are encrypted under an AWS owned key that you cannot restrict, audit or disable. An IAM principal holding the permission to call `GetRecommendation` can read the configuration and result in plaintext.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:GetRecommendation` | `$.recommendationId` |
| `bedrock-agentcore:StartRecommendation` | `$.kmsKeyArn` |

### Threat.BEDROCK-AGENTCORE.Recommendation.5

IAM principal, with unintended IAM permissions to read system prompts, tool descriptions, and trace-derived optimization results from an Amazon Bedrock AgentCore recommendation that has been encrypted with an AWS owned KMS key, can read system prompts, tool descriptions, and trace-derived optimization results in the recommendation.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Scoping** (for [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)): By default, your Amazon Bedrock AgentCore recommendation's system prompts, tool descriptions, and trace-derived optimization results are encrypted at rest with an AWS owned key. That key type gives you no key policy to govern, no audit record of key use, and no control over the key lifecycle, so an IAM principal granted broader permissions than intended can retrieve them in plaintext.

**Request parameters**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:GetRecommendation` | `$.recommendationId` |

---

## Mitigations

Each scoping statement below names the capability the mitigation goal governs on the entity and the objective of governing it. Each mitigation's implementations describe how to govern or monitor that capability.

### Mitigation.BEDROCK-AGENTCORE.1

Amazon Bedrock AgentCore service API calls on a governed VPC network should only be made against trusted resources or AWS service-owned resources

**Goal:** [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)

**Addresses:** [Threat.BEDROCK-AGENTCORE.2](#threatbedrock-agentcore2)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.1](#implementationbedrock-agentcore1), [Implementation.BEDROCK-AGENTCORE.2](#implementationbedrock-agentcore2)

**Scoping** (for [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)): Amazon Bedrock AgentCore API calls from your VPCs can target resources in another account. Restricting those calls to resources in your organization enforces the resource perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.BEDROCK-AGENTCORE.2

Amazon Bedrock AgentCore service API calls on a governed VPC network should only be made by trusted identities or AWS services

**Goal:** [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)

**Addresses:** [Threat.BEDROCK-AGENTCORE.1](#threatbedrock-agentcore1)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.3](#implementationbedrock-agentcore3), [Implementation.BEDROCK-AGENTCORE.4](#implementationbedrock-agentcore4)

**Scoping** (for [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)): Amazon Bedrock AgentCore API calls from your VPCs can be made by identities outside your organization. Restricting those calls to principals in your organization enforces the identity perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.BEDROCK-AGENTCORE.AbTest.1

An Amazon Bedrock AgentCore A/B test's execution role should be assumed only by Amazon Bedrock AgentCore acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.BEDROCK-AGENTCORE.AbTest.2](#threatbedrock-agentcoreabtest2)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.AbTest.1](#implementationbedrock-agentcoreabtest1), [Implementation.BEDROCK-AGENTCORE.AbTest.2](#implementationbedrock-agentcoreabtest2), [Implementation.BEDROCK-AGENTCORE.AbTest.3](#implementationbedrock-agentcoreabtest3)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): The `$.roleArn` request parameter sets or replaces your A/B test's IAM execution role, and that role's trust policy specifies which principals can assume it. Allowing only Amazon Bedrock AgentCore, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:CreateABTest` | `$.roleArn` |
| `bedrock-agentcore:UpdateABTest` | `$.roleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:GetABTest` | `$.abTestId` |

### Mitigation.BEDROCK-AGENTCORE.AbTest.2

An Amazon Bedrock AgentCore A/B test should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.BEDROCK-AGENTCORE.AbTest.1](#threatbedrock-agentcoreabtest1)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.AbTest.4](#implementationbedrock-agentcoreabtest4)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): When you call `CreateABTest`, you can tag the A/B test through the `$.tags{key}` request parameter, a key-to-value map of up to 50 tags. Owner and purpose tags help track ownership and the business function each A/B test supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:CreateABTest` | `$.tags{key}` |

### Mitigation.BEDROCK-AGENTCORE.AgentRuntime.1

IAM principals should be restricted to accessing only trusted Amazon Bedrock AgentCore agent runtime resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.BEDROCK-AGENTCORE.AgentRuntime.1](#threatbedrock-agentcoreagentruntime1)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.AgentRuntime.1](#implementationbedrock-agentcoreagentruntime1)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): Resource-based policies on an agent runtime and its endpoint in another account can grant your principals access to that runtime. That access includes invoking the hosted agent or tool, running commands in runtime sessions, and stopping those sessions. The resource perimeter helps restrict your identities to resources your organization owns.

**Configure via**

| Operation | Parameters |
|---|---|
| `organizations:AttachPolicy` | `$.PolicyId`, `$.TargetId` |
| `organizations:CreatePolicy` | `$.Content`, `$.Type` |
| `organizations:UpdatePolicy` | `$.Content`, `$.PolicyId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `organizations:DescribePolicy` | `$.PolicyId` |
| `organizations:ListPolicies` | `$.Filter` |
| `organizations:ListPoliciesForTarget` | `$.Filter`, `$.TargetId` |
| `organizations:ListTargetsForPolicy` | `$.PolicyId` |

### Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2

An Amazon Bedrock AgentCore agent runtime should be configured to send application and trace logs to CloudWatch Logs using vended logs delivery

**Goal:** [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2)

**Addresses:** [Threat.BEDROCK-AGENTCORE.AgentRuntime.2](#threatbedrock-agentcoreagentruntime2), [Threat.BEDROCK-AGENTCORE.AgentRuntime.3](#threatbedrock-agentcoreagentruntime3)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.AgentRuntime.2](#implementationbedrock-agentcoreagentruntime2), [Implementation.BEDROCK-AGENTCORE.AgentRuntime.3](#implementationbedrock-agentcoreagentruntime3)

**Scoping** (for [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2)): Amazon Bedrock AgentCore produces application logs for each invocation of your agent runtime, recording the operation, request ID, session ID, trace IDs and request payload, and trace spans for each `InvokeAgentRuntime` call, recording the agent runtime, endpoint, session ID, latency and error type. Sending those logs to CloudWatch Logs using vended logs delivery provides security-relevant visibility.

**Configure via**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:CreateDelivery` | `$.deliveryDestinationArn`, `$.deliverySourceName` |
| `cloudwatch-logs:PutDeliveryDestination` | `$.deliveryDestinationConfiguration.destinationResourceArn`, `$.deliveryDestinationType` |
| `cloudwatch-logs:PutDeliverySource` | `$.logType` |

**Verify via**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:GetDelivery` | `$.id` |
| `cloudwatch-logs:GetDeliveryDestination` | `$.name` |
| `cloudwatch-logs:GetDeliverySource` | `$.name` |

### Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1

An Amazon Bedrock AgentCore batch evaluation should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.BEDROCK-AGENTCORE.BatchEvaluation.1](#threatbedrock-agentcorebatchevaluation1), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.3](#threatbedrock-agentcorebatchevaluation3), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.4](#threatbedrock-agentcorebatchevaluation4), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.5](#threatbedrock-agentcorebatchevaluation5)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.BatchEvaluation.1](#implementationbedrock-agentcorebatchevaluation1), [Implementation.BEDROCK-AGENTCORE.BatchEvaluation.2](#implementationbedrock-agentcorebatchevaluation2)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): When you call `StartBatchEvaluation`, the `$.kmsKeyArn` request parameter specifies the key that encrypts your batch evaluation's stored artifacts. Using a customer managed key instead of the default AWS owned key gives you full control over the key's lifecycle and permissions.

**Configure via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartBatchEvaluation` | `$.kmsKeyArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:GetBatchEvaluation` | `$.batchEvaluationId` |
| `bedrock-agentcore:ListBatchEvaluations` | *(any parameter)* |

### Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.2

An Amazon Bedrock AgentCore batch evaluation should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.BEDROCK-AGENTCORE.BatchEvaluation.2](#threatbedrock-agentcorebatchevaluation2)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.BatchEvaluation.3](#implementationbedrock-agentcorebatchevaluation3)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): When you start a batch evaluation with `StartBatchEvaluation`, you tag it through the `$.tags{key}` request parameter, a key-to-value map of up to 50 tags. Owner and purpose tags help track ownership and the business function each batch evaluation supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartBatchEvaluation` | `$.tags{key}` |

### Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1

An Amazon Bedrock AgentCore code interpreter should be configured to send application and trace logs to CloudWatch Logs using vended logs delivery

**Goal:** [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2)

**Addresses:** [Threat.BEDROCK-AGENTCORE.CodeInterpreter.1](#threatbedrock-agentcorecodeinterpreter1), [Threat.BEDROCK-AGENTCORE.CodeInterpreter.2](#threatbedrock-agentcorecodeinterpreter2)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.CodeInterpreter.1](#implementationbedrock-agentcorecodeinterpreter1), [Implementation.BEDROCK-AGENTCORE.CodeInterpreter.2](#implementationbedrock-agentcorecodeinterpreter2)

**Scoping** (for [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2)): Amazon Bedrock AgentCore produces application logs for each `InvokeCodeInterpreter` call to your code interpreter, recording its session identifier and request and response payloads, and trace spans for each `StartCodeInterpreterSession`, `InvokeCodeInterpreter` and `StopCodeInterpreterSession` call, recording the code interpreter, tool session ID, latency and error type. Sending those logs to CloudWatch Logs using vended logs delivery provides security-relevant visibility.

**Configure via**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:CreateDelivery` | `$.deliveryDestinationArn`, `$.deliverySourceName` |
| `cloudwatch-logs:PutDeliveryDestination` | `$.deliveryDestinationConfiguration.destinationResourceArn`, `$.deliveryDestinationType` |
| `cloudwatch-logs:PutDeliverySource` | `$.logType` |

**Verify via**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:GetDelivery` | `$.id` |
| `cloudwatch-logs:GetDeliveryDestination` | `$.name` |
| `cloudwatch-logs:GetDeliverySource` | `$.name` |

### Mitigation.BEDROCK-AGENTCORE.Event.1

IAM principals should be restricted to accessing only events in trusted Amazon Bedrock AgentCore memories

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.BEDROCK-AGENTCORE.Event.1](#threatbedrock-agentcoreevent1)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.Event.1](#implementationbedrock-agentcoreevent1)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): A resource-based policy on an Amazon Bedrock AgentCore memory in another account can grant your principals access to write, read, list and delete that memory's events. The resource perimeter helps restrict your identities to resources your organization owns.

**Configure via**

| Operation | Parameters |
|---|---|
| `organizations:AttachPolicy` | `$.PolicyId`, `$.TargetId` |
| `organizations:CreatePolicy` | `$.Content`, `$.Type` |
| `organizations:UpdatePolicy` | `$.Content`, `$.PolicyId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `organizations:DescribePolicy` | `$.PolicyId` |
| `organizations:ListPolicies` | `$.Filter` |
| `organizations:ListPoliciesForTarget` | `$.Filter`, `$.TargetId` |
| `organizations:ListTargetsForPolicy` | `$.PolicyId` |

### Mitigation.BEDROCK-AGENTCORE.Memory.1

IAM principals should be restricted to accessing only trusted Amazon Bedrock AgentCore memory resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.BEDROCK-AGENTCORE.Memory.1](#threatbedrock-agentcorememory1)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.Memory.1](#implementationbedrock-agentcorememory1)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): A resource-based policy on an Amazon Bedrock AgentCore memory in another account can grant your principals data plane access to that memory. Each `IngestData`, `ListActors`, `ListSessions`, `ListMemoryExtractionJobs`, and `StartMemoryExtractionJob` request names the memory it targets. The resource perimeter helps restrict your identities to resources your organization owns.

**Configure via**

| Operation | Parameters |
|---|---|
| `organizations:AttachPolicy` | `$.PolicyId`, `$.TargetId` |
| `organizations:CreatePolicy` | `$.Content`, `$.Type` |
| `organizations:UpdatePolicy` | `$.Content`, `$.PolicyId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `organizations:DescribePolicy` | `$.PolicyId` |
| `organizations:ListPolicies` | `$.Filter` |
| `organizations:ListPoliciesForTarget` | `$.Filter`, `$.TargetId` |
| `organizations:ListTargetsForPolicy` | `$.PolicyId` |

### Mitigation.BEDROCK-AGENTCORE.Memory.2

An Amazon Bedrock AgentCore memory should be configured to send application and trace logs to CloudWatch Logs using vended logs delivery

**Goal:** [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2)

**Addresses:** [Threat.BEDROCK-AGENTCORE.Memory.2](#threatbedrock-agentcorememory2), [Threat.BEDROCK-AGENTCORE.Memory.3](#threatbedrock-agentcorememory3)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.Memory.2](#implementationbedrock-agentcorememory2), [Implementation.BEDROCK-AGENTCORE.Memory.3](#implementationbedrock-agentcorememory3)

**Scoping** (for [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2)): Amazon Bedrock AgentCore produces application logs that record your memory's deletion and its long-term memory extraction and consolidation workflows with actor and session identifiers, and trace spans that record `CreateEvent`, `GetEvent`, `ListEvents`, `DeleteEvent`, `RetrieveMemoryRecords` and `ListMemoryRecords` calls with memory, session, event, actor or namespace identifiers. Sending those logs to CloudWatch Logs using vended logs delivery provides security-relevant visibility.

**Configure via**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:CreateDelivery` | `$.deliveryDestinationArn`, `$.deliverySourceName` |
| `cloudwatch-logs:PutDeliveryDestination` | `$.deliveryDestinationConfiguration.destinationResourceArn`, `$.deliveryDestinationType` |
| `cloudwatch-logs:PutDeliverySource` | `$.logType` |

**Verify via**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:GetDelivery` | `$.id` |
| `cloudwatch-logs:GetDeliveryDestination` | `$.name` |
| `cloudwatch-logs:GetDeliverySource` | `$.name` |

### Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1

IAM principals should be restricted to accessing only records in trusted Amazon Bedrock AgentCore memories

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.BEDROCK-AGENTCORE.MemoryRecord.1](#threatbedrock-agentcorememoryrecord1)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.MemoryRecord.1](#implementationbedrock-agentcorememoryrecord1)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): A resource-based policy on an Amazon Bedrock AgentCore memory in another account can grant your principals access to its long-term memory records. That access can include calls that write, update, delete, read, list, or semantically search those records. The resource perimeter helps restrict your identities to resources your organization owns.

**Configure via**

| Operation | Parameters |
|---|---|
| `organizations:AttachPolicy` | `$.PolicyId`, `$.TargetId` |
| `organizations:CreatePolicy` | `$.Content`, `$.Type` |
| `organizations:UpdatePolicy` | `$.Content`, `$.PolicyId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `organizations:DescribePolicy` | `$.PolicyId` |
| `organizations:ListPolicies` | `$.Filter` |
| `organizations:ListPoliciesForTarget` | `$.Filter`, `$.TargetId` |
| `organizations:ListTargetsForPolicy` | `$.PolicyId` |

### Mitigation.BEDROCK-AGENTCORE.Recommendation.1

An Amazon Bedrock AgentCore recommendation should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.BEDROCK-AGENTCORE.Recommendation.1](#threatbedrock-agentcorerecommendation1), [Threat.BEDROCK-AGENTCORE.Recommendation.3](#threatbedrock-agentcorerecommendation3), [Threat.BEDROCK-AGENTCORE.Recommendation.4](#threatbedrock-agentcorerecommendation4), [Threat.BEDROCK-AGENTCORE.Recommendation.5](#threatbedrock-agentcorerecommendation5)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.Recommendation.1](#implementationbedrock-agentcorerecommendation1), [Implementation.BEDROCK-AGENTCORE.Recommendation.2](#implementationbedrock-agentcorerecommendation2)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): When you start a recommendation, the `$.kmsKeyArn` request parameter specifies an AWS KMS customer managed key that encrypts the recommendation's configuration and result. Using a customer managed key for this encryption gives you control over the key's lifecycle and permissions.

**Configure via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartRecommendation` | `$.kmsKeyArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:GetRecommendation` | `$.recommendationId` |

### Mitigation.BEDROCK-AGENTCORE.Recommendation.2

An Amazon Bedrock AgentCore recommendation should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.BEDROCK-AGENTCORE.Recommendation.2](#threatbedrock-agentcorerecommendation2)

**Implemented by:** [Implementation.BEDROCK-AGENTCORE.Recommendation.3](#implementationbedrock-agentcorerecommendation3)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag a recommendation when you call `StartRecommendation`, through the `$.tags{key}` request parameter, a key-to-value map of up to 50 tags. Owner and purpose tags help track ownership and the business function each recommendation supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `bedrock-agentcore:StartRecommendation` | `$.name`, `$.tags{key}` |

---

## Implementations

### Implementation.BEDROCK-AGENTCORE.1

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1)): This implementation guidance recommends using your VPC endpoint policy to govern Amazon Bedrock AgentCore API calls made through the endpoint with the `aws:ResourceOrgID` condition key. As a result, calls to Amazon Bedrock AgentCore resources outside your organization are restricted, while AWS services acting on your behalf (`aws:PrincipalIsAWSService`) remain exempt.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.BEDROCK-AGENTCORE.2

*CloudTrail network activity event — Detective*

Monitors the API calls below in CloudTrail.

- `bedrock-agentcore:BatchCreateMemoryRecords`
- `bedrock-agentcore:BatchDeleteMemoryRecords`
- `bedrock-agentcore:CreateEvent`
- `bedrock-agentcore:InvokeAgentRuntime`
- `bedrock-agentcore:RetrieveMemoryRecords`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1)): This implementation guidance recommends recording Amazon Bedrock AgentCore API calls made through your VPC endpoints, which are logged with an `eventSource` of `bedrock-agentcore.amazonaws.com`. Each event shows the account and principal that made the call, in `userIdentity`, and which operation they called. A call that violates your VPC endpoint policy carries an `errorCode` of `VpceAccessDenied`, helping you detect attempts to reach untrusted resources.

### Implementation.BEDROCK-AGENTCORE.3

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2)): This implementation guidance recommends governing Amazon Bedrock AgentCore API calls with the `aws:PrincipalOrgID` condition key in your VPC endpoint policy, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, identities outside your organization are restricted from making Amazon Bedrock AgentCore API calls through your network.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.BEDROCK-AGENTCORE.4

*CloudTrail network activity event — Detective*

Monitors the API calls below in CloudTrail.

- `bedrock-agentcore:BatchDeleteMemoryRecords`
- `bedrock-agentcore:CreateEvent`
- `bedrock-agentcore:InvokeAgentRuntime`
- `bedrock-agentcore:InvokeCodeInterpreter`
- `bedrock-agentcore:RetrieveMemoryRecords`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2)): This implementation guidance recommends recording Amazon Bedrock AgentCore API calls made through your VPC endpoints, which are logged with an `eventSource` of `bedrock-agentcore.amazonaws.com`. Each event shows the account and principal that made the call, in `userIdentity`, and which operation they called, helping you detect calls made by untrusted identities. A call that violates your VPC endpoint policy carries an `errorCode` of `VpceAccessDenied`, which you can filter on.

### Implementation.BEDROCK-AGENTCORE.AbTest.1

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore:ListABTests`; check `$.abTests[*].abTestId`
2. Call `bedrock-agentcore:GetABTest` with `$.abTestId`; check `$.roleArn`
3. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
5. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1)): This implementation guidance recommends reading `$.roleArn`, your A/B test's execution role, then evaluating `$.Role.AssumeRolePolicyDocument`, that role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.BEDROCK-AGENTCORE.AbTest.2

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `bedrock-agentcore.amazonaws.com` | `sts:AssumeRole` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon Bedrock AgentCore A/B test's execution role, `$.roleArn`, to the `bedrock-agentcore.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon Bedrock AgentCore from assuming the role on behalf of A/B tests outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.BEDROCK-AGENTCORE.AbTest.3

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon Bedrock AgentCore A/B test's execution role, `$.roleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.BEDROCK-AGENTCORE.AbTest.4

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore:ListABTests`; check `$.abTests[*].abTestArn`
2. Call `bedrock-agentcore-control:ListTagsForResource` with `$.resourceArn`; check `$.tags{key}`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.AbTest.2](#mitigationbedrock-agentcoreabtest2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.AbTest.2](#mitigationbedrock-agentcoreabtest2)): This implementation guidance recommends reading `$.tags{key}` on each Amazon Bedrock AgentCore A/B test listed through `$.abTests[*].abTestArn` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each A/B test to an accountable owner and the business function it supports.

### Implementation.BEDROCK-AGENTCORE.AgentRuntime.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.1](#mitigationbedrock-agentcoreagentruntime1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.1](#mitigationbedrock-agentcoreagentruntime1)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern Amazon Bedrock AgentCore API calls. As a result, your identities are restricted from accessing agent runtimes outside your organization, except through OAuth bearer-token calls made without IAM credentials.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.BEDROCK-AGENTCORE.AgentRuntime.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteDelivery` — `$.id`
- `cloudwatch-logs:DeleteDeliveryDestination` — `$.name`
- `cloudwatch-logs:DeleteDeliverySource` — `$.name`
- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteResourcePolicy` — `$.policyName`, `$.resourceArn`
- `cloudwatch-logs:PutDeliveryDestination` — `$.deliveryDestinationConfiguration.destinationResourceArn`, `$.name`
- `cloudwatch-logs:PutDeliverySource` — `$.name`, `$.resourceArn`
- `cloudwatch-logs:PutResourcePolicy` — `$.policyName`, `$.resourceArn`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `cloudwatch-logs:UpdateDeliveryConfiguration` — `$.id`, `$.recordFields[*]`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2)): This implementation guidance recommends recording the CloudWatch Logs operations that can disable or weaken the vended logs delivery for your Amazon Bedrock AgentCore agent runtime. Recording calls such as `DeleteDelivery`, `DeleteDeliverySource`, and `PutRetentionPolicy`, together with the calling identity, helps you detect and attribute those changes.

### Implementation.BEDROCK-AGENTCORE.AgentRuntime.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore-control:ListAgentRuntimes`; check `$.agentRuntimes[*].agentRuntimeArn`
2. Call `cloudwatch-logs:DescribeDeliverySources` with `$.resourceArns[*]`; check `$.deliverySources[*].resourceArns[*]`, `$.deliverySources[*].logType` = `APPLICATION_LOGS`, `$.deliverySources[*].name`
3. Call `cloudwatch-logs:DescribeDeliveries` with `$.deliverySourceNames[*]`; check `$.deliveries[*].deliveryDestinationType` = `CWL`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2)): This implementation guidance recommends flagging an Amazon Bedrock AgentCore agent runtime with no vended logs delivery to CloudWatch Logs, read from `$.deliveries[*].deliveryDestinationType`, the delivery's destination type. Surfacing that gap helps you detect agent runtime invocation activity that is not being recorded for review.

### Implementation.BEDROCK-AGENTCORE.BatchEvaluation.1

*CloudTrail data event — Detective*

Monitors the API calls below in CloudTrail.

- `bedrock-agentcore:GetBatchEvaluation`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1)): This implementation guidance recommends recording `GetBatchEvaluation` calls against your Amazon Bedrock AgentCore batch evaluations, along with the identity that made each call. Recording those calls helps you detect and investigate authorized-but-unexpected access to the data protected by your customer managed key.

**Reference:** Enforcing a customer managed key governs which key protects the data. It does not record which permitted principals read that data. The data events this guide monitors record those reads. CloudTrail does not log data events by default, and they are high volume. To scope log volume and cost, add advanced event selectors to the trail or event data store. Filter `resources.ARN` to the governed resources, using `Equals` for exact ARNs or `StartsWith` for a naming prefix. Filter `eventName` to the operations this guide monitors. A `readOnly` filter selects by CloudTrail's read or write classification, not by this guide's operation set, so do not rely on it alone. A trail or event data store accepts at most 500 values across all advanced event selector conditions.
- [Filtering data events by using advanced event selectors](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/filtering-data-events.html)
- [Logging data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)
- [Quotas in AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/WhatIsCloudTrail-Limits.html)

### Implementation.BEDROCK-AGENTCORE.BatchEvaluation.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore:ListBatchEvaluations`; check `$.batchEvaluations[*].kmsKeyArn`
2. Call `kms:DescribeKey` with `$.KeyId`; check `$.KeyMetadata.KeyManager` = `CUSTOMER`, `$.KeyMetadata.AWSAccountId`, `$.KeyMetadata.Origin`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1)): This implementation guidance recommends reading `$.batchEvaluations[*].kmsKeyArn`, the key that encrypts a batch evaluation's stored output artifacts, and resolving `$.KeyMetadata.KeyManager`, `$.KeyMetadata.AWSAccountId`, and `$.KeyMetadata.Origin` for that key. Flagging batch evaluations whose key is not a customer managed key in your account, or whose key material is imported or held in an external key store, helps you detect where you do not control the key policy, lifecycle, or key material.

### Implementation.BEDROCK-AGENTCORE.BatchEvaluation.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore:ListBatchEvaluations`; check `$.batchEvaluations[*].batchEvaluationArn`
2. Call `resource-groups-tagging-api:GetResources` with `$.ResourceARNList[*]`; check `$.ResourceTagMappingList[*].ResourceARN`, `$.ResourceTagMappingList[*].Tags[*].Key`, `$.ResourceTagMappingList[*].Tags[*].Value`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.2](#mitigationbedrock-agentcorebatchevaluation2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.2](#mitigationbedrock-agentcorebatchevaluation2)): This implementation guidance recommends reading `$.ResourceTagMappingList[*].Tags[*].Key` on each Amazon Bedrock AgentCore batch evaluation listed through `$.batchEvaluations[*].batchEvaluationArn` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon Bedrock AgentCore batch evaluation to an accountable owner and the business function it supports.

### Implementation.BEDROCK-AGENTCORE.CodeInterpreter.1

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteDelivery` — `$.id`
- `cloudwatch-logs:DeleteDeliveryDestination` — `$.name`
- `cloudwatch-logs:DeleteDeliveryDestinationPolicy` — `$.deliveryDestinationName`
- `cloudwatch-logs:DeleteDeliverySource` — `$.name`
- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteResourcePolicy` — `$.policyName`, `$.resourceArn`
- `cloudwatch-logs:PutDeliveryDestination` — `$.deliveryDestinationConfiguration.destinationResourceArn`
- `cloudwatch-logs:PutDeliverySource` — `$.logType`, `$.resourceArn`
- `cloudwatch-logs:PutResourcePolicy` — `$.policyName`, `$.resourceArn`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `cloudwatch-logs:UpdateDeliveryConfiguration` — `$.id`, `$.recordFields[*]`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1)): This implementation guidance recommends recording the CloudWatch Logs operations that can disable or weaken the vended logs delivery for your Amazon Bedrock AgentCore code interpreter. Recording calls such as `DeleteDelivery`, `PutDeliverySource`, and `PutRetentionPolicy`, together with the calling identity, helps you detect and attribute those changes.

### Implementation.BEDROCK-AGENTCORE.CodeInterpreter.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore-control:ListCodeInterpreters` with `$.type` = `CUSTOM`; check `$.codeInterpreterSummaries[*].codeInterpreterArn`
2. Call `cloudwatch-logs:DescribeDeliverySources` with `$.resourceArns[*]`; check `$.deliverySources[*].name`, `$.deliverySources[*].logType` = `APPLICATION_LOGS`
3. Call `cloudwatch-logs:DescribeDeliveries` with `$.deliverySourceNames[*]`; check `$.deliveries[*].deliveryDestinationType` = `CWL`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1)): This implementation guidance recommends flagging an Amazon Bedrock AgentCore code interpreter with no vended logs delivery to CloudWatch Logs, read from `$.deliveries[*].deliveryDestinationType`, the delivery's destination type. Surfacing that gap helps you detect code interpreter invocation activity that is not being recorded for review.

### Implementation.BEDROCK-AGENTCORE.Event.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.BEDROCK-AGENTCORE.Event.1](#mitigationbedrock-agentcoreevent1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.Event.1](#mitigationbedrock-agentcoreevent1)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern Amazon Bedrock AgentCore API calls. As a result, your identities are restricted from accessing memory events outside your organization, when IAM evaluates the event request against the calling identity.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.BEDROCK-AGENTCORE.Memory.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.BEDROCK-AGENTCORE.Memory.1](#mitigationbedrock-agentcorememory1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.Memory.1](#mitigationbedrock-agentcorememory1)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern Amazon Bedrock AgentCore API calls. As a result, your identities are restricted from accessing memory resources outside your organization, when IAM evaluates their requests.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.BEDROCK-AGENTCORE.Memory.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteDelivery` — `$.id`
- `cloudwatch-logs:DeleteDeliveryDestination` — `$.name`
- `cloudwatch-logs:DeleteDeliverySource` — `$.name`
- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteResourcePolicy` — `$.policyName`, `$.resourceArn`
- `cloudwatch-logs:PutDeliveryDestination` — `$.deliveryDestinationConfiguration.destinationResourceArn`
- `cloudwatch-logs:PutDeliverySource` — `$.resourceArn`
- `cloudwatch-logs:PutResourcePolicy` — `$.policyName`, `$.resourceArn`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `cloudwatch-logs:UpdateDeliveryConfiguration` — `$.id`, `$.recordFields[*]`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2)): This implementation guidance recommends recording the CloudWatch Logs operations that can disable or weaken the vended logs delivery for your Amazon Bedrock AgentCore memory. Recording calls such as `DeleteDelivery`, `PutDeliverySource`, and `PutRetentionPolicy`, together with the calling identity, helps you detect and attribute those changes.

### Implementation.BEDROCK-AGENTCORE.Memory.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore-control:ListMemories`; check `$.memories[*].arn`
2. Call `cloudwatch-logs:DescribeDeliverySources` with `$.resourceArns[*]`; check `$.deliverySources[*].resourceArns[*]`, `$.deliverySources[*].logType` = `APPLICATION_LOGS`, `$.deliverySources[*].name`
3. Call `cloudwatch-logs:DescribeDeliveries` with `$.deliverySourceNames[*]`; check `$.deliveries[*].deliverySourceName`, `$.deliveries[*].deliveryDestinationType` = `CWL`, `$.deliveries[*].deliveryDestinationArn`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2)): This implementation guidance recommends flagging an Amazon Bedrock AgentCore memory with no vended logs delivery to CloudWatch Logs, read from `$.deliveries[*].deliveryDestinationType`, the delivery's destination type. Surfacing that gap helps you detect memory deletion, extraction, and consolidation activity that is not being recorded for review.

### Implementation.BEDROCK-AGENTCORE.MemoryRecord.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern Amazon Bedrock AgentCore API calls. As a result, your identities are restricted from accessing records in Amazon Bedrock AgentCore memories outside your organization, when IAM evaluates the request under your identity.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.BEDROCK-AGENTCORE.Recommendation.1

*CloudTrail data event — Detective*

Monitors the API calls below in CloudTrail.

- `bedrock-agentcore:GetRecommendation` — `$.recommendationId`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1)): This implementation guidance recommends recording `GetRecommendation` calls against your Amazon Bedrock AgentCore recommendations, along with the identity that made each call. Recording those calls helps you detect and investigate authorized-but-unexpected access to the data protected by your customer managed key.

**Reference:** Enforcing a customer managed key governs which key protects the data. It does not record which permitted principals read that data. The data events this guide monitors record those reads. CloudTrail does not log data events by default, and they are high volume. To scope log volume and cost, add advanced event selectors to the trail or event data store. Filter `resources.ARN` to the governed resources, using `Equals` for exact ARNs or `StartsWith` for a naming prefix. Filter `eventName` to the operations this guide monitors. A `readOnly` filter selects by CloudTrail's read or write classification, not by this guide's operation set, so do not rely on it alone. A trail or event data store accepts at most 500 values across all advanced event selector conditions.
- [Filtering data events by using advanced event selectors](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/filtering-data-events.html)
- [Logging data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)
- [Quotas in AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/WhatIsCloudTrail-Limits.html)

### Implementation.BEDROCK-AGENTCORE.Recommendation.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore:ListRecommendations`; check `$.recommendationSummaries[*].recommendationId`
2. Call `bedrock-agentcore:GetRecommendation` with `$.recommendationId`; check `$.kmsKeyArn`
3. Call `kms:DescribeKey` with `$.KeyId`; check `$.KeyMetadata.KeyManager` = `CUSTOMER`, `$.KeyMetadata.AWSAccountId`, `$.KeyMetadata.Origin`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1)): This implementation guidance recommends reading `$.kmsKeyArn`, the key that encrypts the recommendation's configuration and result, and resolving `$.KeyMetadata.KeyManager`, `$.KeyMetadata.AWSAccountId`, and `$.KeyMetadata.Origin` for that key. Flagging Amazon Bedrock AgentCore recommendations whose key is not a customer managed key in your account, or whose key material is imported or held in an external key store, helps you detect where you do not control the key policy, lifecycle, or key material.

### Implementation.BEDROCK-AGENTCORE.Recommendation.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `bedrock-agentcore:ListRecommendations`; check `$.recommendationSummaries[*].recommendationArn`
2. Call `bedrock-agentcore-control:ListTagsForResource` with `$.resourceArn`; check `$.tags{key}`

**Implements:** [Mitigation.BEDROCK-AGENTCORE.Recommendation.2](#mitigationbedrock-agentcorerecommendation2)

**Scoping** (for [Mitigation.BEDROCK-AGENTCORE.Recommendation.2](#mitigationbedrock-agentcorerecommendation2)): This implementation guidance recommends reading `$.tags{key}` on each Amazon Bedrock AgentCore recommendation listed through `$.recommendationSummaries[*].recommendationArn` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each recommendation to an accountable owner and the business function it supports.

---

## Appendix

### Resources

#### AbTest

*JOB_OR_WORKFLOW*

An A/B test in Amazon Bedrock AgentCore compares two versions of an agent using live production traffic. It routes incoming sessions to a control or treatment variant through an AgentCore Gateway, without changes to agent code.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:ab-test/${ABTestId}`
- 5 operations, 2 threats, 2 mitigations

#### AgentRuntime

*COMPUTE_RESOURCE*

An agent runtime hosts AI agent or tool code in a secure, serverless environment built for agentic workloads. It works with any agent framework or foundation model. It also supports common agent communication protocols such as MCP and A2A.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:runtime/${RuntimeId}`
- CloudFormation: `AWS::BedrockAgentCore::Runtime`
- AWS Config: `AWS::BedrockAgentCore::Runtime`
- 4 operations, 3 threats, 2 mitigations

#### BatchEvaluation

*JOB_OR_WORKFLOW*

A batch evaluation is an asynchronous job that scores many recorded agent sessions in a single run. It reads agent telemetry from Amazon CloudWatch Logs, or reuses sessions already sampled by an online evaluation configuration. It then applies a chosen set of evaluators to each session it discovers.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:batch-evaluate/${BatchEvaluationId}`
- 5 operations, 5 threats, 2 mitigations

#### BrowserProfile

*DATA_STORE*

A browser profile stores persistent browser state, such as cookies and local storage, captured from an Amazon Bedrock AgentCore Browser session. You can load the saved state into later browser sessions so they start where an earlier session left off.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:browser-profile/${BrowserProfileId}`
- CloudFormation: `AWS::BedrockAgentCore::BrowserProfile`
- 1 operations, 0 threats, 0 mitigations

#### BrowserSession

*COMPUTE_RESOURCE*

A browser session is an isolated, managed Chromium browser that an AI agent uses to navigate websites, fill in forms, and extract information. Each session runs in its own dedicated sandbox with separate compute, memory, and file system, and is torn down when it ends.

- 6 operations, 0 threats, 0 mitigations

#### CapacityProvider

*CONFIGURATION*

A capacity provider defines the Amazon EC2 infrastructure that hosts Amazon Bedrock AgentCore agents on the Instances compute type. It specifies the operating system, allowed instance types, VPC placement, storage volumes, lifecycle limits, and the IAM role Amazon Bedrock AgentCore uses to manage instances.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:capacity-provider/${CapacityProviderId}`
- CloudFormation: `AWS::BedrockAgentCore::CapacityProvider`
- 1 operations, 0 threats, 0 mitigations

#### CodeInterpreter

*COMPUTE_RESOURCE*

A code interpreter gives AI agents a managed, sandboxed environment for writing and running code. Agents use it to perform precise calculations, analyze structured data, process files, and create visualizations that reasoning alone cannot reliably produce.

- IAM resource ARNs: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:code-interpreter-custom/${CodeInterpreterId}`, `arn:${Partition}:bedrock-agentcore:${Region}:aws:code-interpreter/${CodeInterpreterId}`
- CloudFormation: `AWS::BedrockAgentCore::CodeInterpreter`, `AWS::BedrockAgentCore::CodeInterpreterCustom`
- AWS Config: `AWS::BedrockAgentCore::CodeInterpreterCustom`
- 1 operations, 2 threats, 1 mitigations

#### CodeInterpreterSession

*COMPUTE_RESOURCE*

A code interpreter session is an isolated, short-lived sandbox where an AI agent writes and runs code. Each session starts from a code interpreter and inherits its network access and permission settings.

- 4 operations, 0 threats, 0 mitigations

#### Evaluator

*AI_RESOURCE*

An evaluator scores the quality of an AI agent's behavior using the telemetry the agent emits. It examines complete conversations, individual request-and-response turns, or single tool calls. It returns a score, a label, and a written explanation.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:evaluator/${EvaluatorId}`
- CloudFormation: `AWS::BedrockAgentCore::Evaluator`
- AWS Config: `AWS::BedrockAgentCore::Evaluator`
- 1 operations, 0 threats, 0 mitigations

#### Event

*DATA_STORE*

An event is a single, timestamped record of an interaction stored in the short-term memory of an AgentCore Memory resource. Agents use events to capture conversation turns, tool results, activity data, and binary content such as documents or saved agent state.

- 4 operations, 1 threats, 1 mitigations

#### Harness

*AI_RESOURCE*

An Amazon Bedrock AgentCore harness is a managed agent loop that turns a declarative agent definition into a running agent. You declare the model, instructions, tools, skills, memory, and execution limits. The service then handles orchestration, tool execution, context management, and streamed responses.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:harness/${HarnessId}`
- CloudFormation: `AWS::BedrockAgentCore::Harness`
- 1 operations, 0 threats, 0 mitigations

#### Memory

*DATA_STORE*

An Amazon Bedrock AgentCore memory is a fully managed store that gives AI agents persistent context across interactions. It keeps raw conversation history so an agent can resume a session without users repeating themselves. It can also distill conversations into durable insights, such as user preferences, facts, and summaries, for reuse in later sessions.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:memory/${MemoryId}`
- CloudFormation: `AWS::BedrockAgentCore::Memory`
- AWS Config: `AWS::BedrockAgentCore::Memory`
- 5 operations, 3 threats, 2 mitigations

#### MemoryRecord

*DATA_STORE*

A memory record is a structured unit of long-term knowledge that an AgentCore Memory keeps on behalf of an AI agent. Each record captures a distilled insight, such as a user preference, fact, or conversation summary, that the agent can recall in later sessions.

- 7 operations, 1 threats, 1 mitigations

#### PaymentInstrument

*CRYPTOGRAPHIC_MATERIAL*

A payment instrument represents an end user's embedded crypto wallet that an AI agent uses to pay for APIs, tools, and web content. Each instrument is tied to one end user and one blockchain network, such as Ethereum-compatible chains or Solana. It links the wallet to the user's identity at the wallet provider, such as an email address, phone number, or social sign-in.

- 6 operations, 0 threats, 0 mitigations

#### PaymentSession

*ACCESS_CONTROL*

A payment session is a time-bounded spending context for an AI agent acting on behalf of one end user. Each session belongs to a payment manager and carries an expiry time and an optional maximum spend amount. It also tracks how much of that budget remains as the agent pays for APIs, tools, and content.

- 5 operations, 0 threats, 0 mitigations

#### Recommendation

*JOB_OR_WORKFLOW*

A recommendation analyzes an agent's recorded session traces and proposes an improved system prompt or clearer tool descriptions. It helps teams move from manual trace review and prompt rewriting to evidence-based configuration changes.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:recommendation/${RecommendationId}`
- 4 operations, 5 threats, 2 mitigations

#### Registry

*RESOURCE_GROUPING*

A registry is a governed catalog for publishing and discovering agentic resources across an organization. Records describe agents, tools, MCP servers, and skills, capturing what each resource does and how to reach it.

- IAM resource ARN: `arn:${Partition}:bedrock-agentcore:${Region}:${Account}:registry/${RegistryId}`
- 1 operations, 0 threats, 0 mitigations

#### ResourceApiKey

*CRYPTOGRAPHIC_MATERIAL*

A resource API key is the stored secret an agent uses to authenticate to external services that rely on key-based access. AgentCore Identity keeps the key in encrypted, access-controlled storage and releases it to authorized agent workloads at runtime.

- 1 operations, 0 threats, 0 mitigations

#### ResourceOauth2Token

*CRYPTOGRAPHIC_MATERIAL*

A resource OAuth2 token is an OAuth 2.0 access token that lets an agent call an external API, tool, or MCP server. AgentCore Identity brokers the OAuth flow with the configured authorization server and keeps the resulting tokens in its token vault. This design helps keep client secrets and long-term credentials out of agent code.

- 2 operations, 0 threats, 0 mitigations

#### ResourcePaymentToken

*CRYPTOGRAPHIC_MATERIAL*

A resource payment token is a vendor-specific authentication credential that AgentCore Identity generates on demand for agent payment workflows. It is derived from developer credentials held in a payment credential provider, so agent code does not need to store those keys.

- 1 operations, 0 threats, 0 mitigations

#### WorkloadAccessToken

*CRYPTOGRAPHIC_MATERIAL*

A workload access token is a short-lived, AWS-signed credential that represents an agent's workload identity within Amazon Bedrock AgentCore. When an agent acts for an end user, the token also carries that user's identity, binding the two together.

- 3 operations, 0 threats, 0 mitigations

### Operations

| Operation | Analyses | Resources | Mitigations |
|---|---|---|---|
| `BatchCreateMemoryRecords` | DOCUMENTATION, DATA_PERIMETER | [MemoryRecord](#memoryrecord) | [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1), [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1) |
| `BatchDeleteMemoryRecords` | DOCUMENTATION, DATA_PERIMETER | [MemoryRecord](#memoryrecord) |  |
| `BatchUpdateMemoryRecords` | DOCUMENTATION, DATA_PERIMETER | [MemoryRecord](#memoryrecord) | [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1) |
| `CompleteResourceTokenAuth` | DOCUMENTATION | [ResourceOauth2Token](#resourceoauth2token) |  |
| `CreateABTest` | DOCUMENTATION | [AbTest](#abtest) | [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1), [Mitigation.BEDROCK-AGENTCORE.AbTest.2](#mitigationbedrock-agentcoreabtest2) |
| `CreateEvent` | DOCUMENTATION, DATA_PERIMETER | [Event](#event) | [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1), [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2), [Mitigation.BEDROCK-AGENTCORE.Event.1](#mitigationbedrock-agentcoreevent1) |
| `CreatePaymentInstrument` | DOCUMENTATION | [PaymentInstrument](#paymentinstrument) |  |
| `CreatePaymentSession` | DOCUMENTATION | [PaymentSession](#paymentsession) |  |
| `DeleteABTest` | DOCUMENTATION | [AbTest](#abtest) |  |
| `DeleteBatchEvaluation` | DOCUMENTATION | [BatchEvaluation](#batchevaluation) |  |
| `DeleteCapacityProviderSession` | DOCUMENTATION | [CapacityProvider](#capacityprovider) |  |
| `DeleteEvent` | DOCUMENTATION, DATA_PERIMETER | [Event](#event) |  |
| `DeleteMemoryRecord` | DOCUMENTATION, DATA_PERIMETER | [MemoryRecord](#memoryrecord) |  |
| `DeletePaymentInstrument` | DOCUMENTATION | [PaymentInstrument](#paymentinstrument) |  |
| `DeletePaymentSession` | DOCUMENTATION | [PaymentSession](#paymentsession) |  |
| `DeleteRecommendation` | DOCUMENTATION | [Recommendation](#recommendation) |  |
| `Evaluate` | DOCUMENTATION | [Evaluator](#evaluator) |  |
| `GetABTest` | DOCUMENTATION | [AbTest](#abtest) | [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1) |
| `GetAgentCard` | DOCUMENTATION, DATA_PERIMETER | [AgentRuntime](#agentruntime) |  |
| `GetBatchEvaluation` | DOCUMENTATION | [BatchEvaluation](#batchevaluation) | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1) |
| `GetBrowserSession` | DOCUMENTATION, DATA_PERIMETER | [BrowserSession](#browsersession) |  |
| `GetCodeInterpreterSession` | DOCUMENTATION, DATA_PERIMETER | [CodeInterpreterSession](#codeinterpretersession) |  |
| `GetEvent` | DOCUMENTATION, DATA_PERIMETER | [Event](#event) | [Mitigation.BEDROCK-AGENTCORE.Event.1](#mitigationbedrock-agentcoreevent1) |
| `GetMemoryRecord` | DOCUMENTATION, DATA_PERIMETER | [MemoryRecord](#memoryrecord) | [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1) |
| `GetPaymentInstrument` | DOCUMENTATION | [PaymentInstrument](#paymentinstrument) |  |
| `GetPaymentInstrumentBalance` | DOCUMENTATION | [PaymentInstrument](#paymentinstrument) |  |
| `GetPaymentSession` | DOCUMENTATION | [PaymentSession](#paymentsession) |  |
| `GetRecommendation` | DOCUMENTATION | [Recommendation](#recommendation) | [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1) |
| `GetResourceApiKey` | DOCUMENTATION | [ResourceApiKey](#resourceapikey) |  |
| `GetResourceOauth2Token` | DOCUMENTATION | [ResourceOauth2Token](#resourceoauth2token) |  |
| `GetResourcePaymentToken` | DOCUMENTATION | [ResourcePaymentToken](#resourcepaymenttoken) |  |
| `GetWorkloadAccessToken` | DOCUMENTATION, DATA_PERIMETER | [WorkloadAccessToken](#workloadaccesstoken) |  |
| `GetWorkloadAccessTokenForJWT` | DOCUMENTATION | [WorkloadAccessToken](#workloadaccesstoken) |  |
| `GetWorkloadAccessTokenForUserId` | DOCUMENTATION, DATA_PERIMETER | [WorkloadAccessToken](#workloadaccesstoken) |  |
| `IngestData` | DOCUMENTATION | [Memory](#memory) | [Mitigation.BEDROCK-AGENTCORE.Memory.1](#mitigationbedrock-agentcorememory1), [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2) |
| `InvokeAgentRuntime` | DOCUMENTATION, DATA_PERIMETER | [AgentRuntime](#agentruntime) | [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1), [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2), [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.1](#mitigationbedrock-agentcoreagentruntime1), [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2) |
| `InvokeAgentRuntimeCommand` | DOCUMENTATION | [AgentRuntime](#agentruntime) | [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.1](#mitigationbedrock-agentcoreagentruntime1) |
| `InvokeBrowser` | DOCUMENTATION | [BrowserSession](#browsersession) |  |
| `InvokeCodeInterpreter` | DOCUMENTATION, DATA_PERIMETER | [CodeInterpreter](#codeinterpreter) | [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2), [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1) |
| `InvokeHarness` | DOCUMENTATION | [Harness](#harness) |  |
| `ListABTests` | DOCUMENTATION | [AbTest](#abtest) |  |
| `ListActors` | DOCUMENTATION, DATA_PERIMETER | [Memory](#memory) |  |
| `ListBatchEvaluations` | DOCUMENTATION | [BatchEvaluation](#batchevaluation) | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1) |
| `ListBrowserSessions` | DOCUMENTATION, DATA_PERIMETER | [BrowserSession](#browsersession) |  |
| `ListCodeInterpreterSessions` | DOCUMENTATION, DATA_PERIMETER | [CodeInterpreterSession](#codeinterpretersession) |  |
| `ListEvents` | DOCUMENTATION, DATA_PERIMETER | [Event](#event) |  |
| `ListMemoryExtractionJobs` | DOCUMENTATION | [Memory](#memory) |  |
| `ListMemoryRecords` | DOCUMENTATION, DATA_PERIMETER | [MemoryRecord](#memoryrecord) |  |
| `ListPaymentInstruments` | DOCUMENTATION | [PaymentInstrument](#paymentinstrument) |  |
| `ListPaymentSessions` | DOCUMENTATION | [PaymentSession](#paymentsession) |  |
| `ListRecommendations` | DOCUMENTATION | [Recommendation](#recommendation) |  |
| `ListSessions` | DOCUMENTATION, DATA_PERIMETER | [Memory](#memory) |  |
| `ProcessPayment` | DOCUMENTATION | [PaymentInstrument](#paymentinstrument), [PaymentSession](#paymentsession) |  |
| `RetrieveMemoryRecords` | DOCUMENTATION, DATA_PERIMETER | [MemoryRecord](#memoryrecord) | [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1), [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2), [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1) |
| `SaveBrowserSessionProfile` | DOCUMENTATION | [BrowserProfile](#browserprofile) |  |
| `SearchRegistryRecords` | DOCUMENTATION | [Registry](#registry) |  |
| `StartBatchEvaluation` | DOCUMENTATION | [BatchEvaluation](#batchevaluation) | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1), [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.2](#mitigationbedrock-agentcorebatchevaluation2) |
| `StartBrowserSession` | DOCUMENTATION, DATA_PERIMETER | [BrowserSession](#browsersession) |  |
| `StartCodeInterpreterSession` | DOCUMENTATION, DATA_PERIMETER | [CodeInterpreterSession](#codeinterpretersession) |  |
| `StartMemoryExtractionJob` | DOCUMENTATION | [Memory](#memory) | [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2) |
| `StartRecommendation` | DOCUMENTATION | [Recommendation](#recommendation) | [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1), [Mitigation.BEDROCK-AGENTCORE.Recommendation.2](#mitigationbedrock-agentcorerecommendation2) |
| `StopBatchEvaluation` | DOCUMENTATION | [BatchEvaluation](#batchevaluation) |  |
| `StopBrowserSession` | DOCUMENTATION, DATA_PERIMETER | [BrowserSession](#browsersession) |  |
| `StopCodeInterpreterSession` | DOCUMENTATION, DATA_PERIMETER | [CodeInterpreterSession](#codeinterpretersession) |  |
| `StopRuntimeSession` | DOCUMENTATION, DATA_PERIMETER | [AgentRuntime](#agentruntime) |  |
| `UpdateABTest` | DOCUMENTATION | [AbTest](#abtest) | [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1) |
| `UpdateBrowserStream` | DOCUMENTATION, DATA_PERIMETER | [BrowserSession](#browsersession) |  |

Mitigations attach to an operation directly (the operation appears in the mitigation's request parameters) or through a threat whose request parameters include the operation.

### Goal Coverage (7 of 38)

| Mitigation Goal | Covered | Mitigations |
|---|---|---|
| [AI Services Opt-Out Policy](../mitigation-goals.md#ai-services-opt-out-policy) | — |  |
| [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup) | — |  |
| [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native) | — |  |
| [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery) | — |  |
| [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2) | yes | [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.2](#mitigationbedrock-agentcoreagentruntime2), [Mitigation.BEDROCK-AGENTCORE.CodeInterpreter.1](#mitigationbedrock-agentcorecodeinterpreter1), [Mitigation.BEDROCK-AGENTCORE.Memory.2](#mitigationbedrock-agentcorememory2) |
| [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest) | — |  |
| [Cross-Region Replication](../mitigation-goals.md#cross-region-replication) | — |  |
| [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies) | — |  |
| [Customer Code Execution - Govern Code Content](../mitigation-goals.md#customer-code-execution---govern-code-content) | — |  |
| [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage) | — |  |
| [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection) | — |  |
| [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics) | — |  |
| [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics) | — |  |
| [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy) | — |  |
| [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature) | — |  |
| [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication) | — |  |
| [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network) | yes | [Mitigation.BEDROCK-AGENTCORE.2](#mitigationbedrock-agentcore2) |
| [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources) | — |  |
| [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption) | yes | [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.1](#mitigationbedrock-agentcorebatchevaluation1), [Mitigation.BEDROCK-AGENTCORE.Recommendation.1](#mitigationbedrock-agentcorerecommendation1) |
| [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment) | — |  |
| [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment) | — |  |
| [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks) | — |  |
| [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks) | — |  |
| [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement) | — |  |
| [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets) | — |  |
| [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources) | yes | [Mitigation.BEDROCK-AGENTCORE.AgentRuntime.1](#mitigationbedrock-agentcoreagentruntime1), [Mitigation.BEDROCK-AGENTCORE.Event.1](#mitigationbedrock-agentcoreevent1), [Mitigation.BEDROCK-AGENTCORE.Memory.1](#mitigationbedrock-agentcorememory1), [Mitigation.BEDROCK-AGENTCORE.MemoryRecord.1](#mitigationbedrock-agentcorememoryrecord1) |
| [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network) | yes | [Mitigation.BEDROCK-AGENTCORE.1](#mitigationbedrock-agentcore1) |
| [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition) | — |  |
| [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention) | — |  |
| [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose) | yes | [Mitigation.BEDROCK-AGENTCORE.AbTest.2](#mitigationbedrock-agentcoreabtest2), [Mitigation.BEDROCK-AGENTCORE.BatchEvaluation.2](#mitigationbedrock-agentcorebatchevaluation2), [Mitigation.BEDROCK-AGENTCORE.Recommendation.2](#mitigationbedrock-agentcorerecommendation2) |
| [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role) | yes | [Mitigation.BEDROCK-AGENTCORE.AbTest.1](#mitigationbedrock-agentcoreabtest1) |
| [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access) | — |  |
| [Service Principal Presigned URL](../mitigation-goals.md#service-principal-presigned-url) | — |  |
| [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration) | — |  |
| [Turn on Block Public Access (BPA)](../mitigation-goals.md#turn-on-block-public-access-bpa) | — |  |
| [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention) | — |  |
| [Use Nitro instances](../mitigation-goals.md#use-nitro-instances) | — |  |
| [VPC Deployment](../mitigation-goals.md#vpc-deployment) | — |  |

### Impact Rollup

| Impact | Threats |
|---|---|
| [Data Collection](../threat-impacts.md#data-collection) | [Threat.BEDROCK-AGENTCORE.1](#threatbedrock-agentcore1), [Threat.BEDROCK-AGENTCORE.2](#threatbedrock-agentcore2), [Threat.BEDROCK-AGENTCORE.AbTest.2](#threatbedrock-agentcoreabtest2), [Threat.BEDROCK-AGENTCORE.AgentRuntime.1](#threatbedrock-agentcoreagentruntime1), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.3](#threatbedrock-agentcorebatchevaluation3), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.4](#threatbedrock-agentcorebatchevaluation4), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.5](#threatbedrock-agentcorebatchevaluation5), [Threat.BEDROCK-AGENTCORE.Event.1](#threatbedrock-agentcoreevent1), [Threat.BEDROCK-AGENTCORE.Memory.1](#threatbedrock-agentcorememory1), [Threat.BEDROCK-AGENTCORE.MemoryRecord.1](#threatbedrock-agentcorememoryrecord1), [Threat.BEDROCK-AGENTCORE.Recommendation.3](#threatbedrock-agentcorerecommendation3), [Threat.BEDROCK-AGENTCORE.Recommendation.4](#threatbedrock-agentcorerecommendation4), [Threat.BEDROCK-AGENTCORE.Recommendation.5](#threatbedrock-agentcorerecommendation5) |
| [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Threat.BEDROCK-AGENTCORE.BatchEvaluation.1](#threatbedrock-agentcorebatchevaluation1), [Threat.BEDROCK-AGENTCORE.Recommendation.1](#threatbedrock-agentcorerecommendation1) |
| [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Threat.BEDROCK-AGENTCORE.1](#threatbedrock-agentcore1), [Threat.BEDROCK-AGENTCORE.2](#threatbedrock-agentcore2), [Threat.BEDROCK-AGENTCORE.AbTest.2](#threatbedrock-agentcoreabtest2), [Threat.BEDROCK-AGENTCORE.AgentRuntime.1](#threatbedrock-agentcoreagentruntime1), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.4](#threatbedrock-agentcorebatchevaluation4), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.5](#threatbedrock-agentcorebatchevaluation5), [Threat.BEDROCK-AGENTCORE.Event.1](#threatbedrock-agentcoreevent1), [Threat.BEDROCK-AGENTCORE.Memory.1](#threatbedrock-agentcorememory1), [Threat.BEDROCK-AGENTCORE.MemoryRecord.1](#threatbedrock-agentcorememoryrecord1), [Threat.BEDROCK-AGENTCORE.Recommendation.4](#threatbedrock-agentcorerecommendation4), [Threat.BEDROCK-AGENTCORE.Recommendation.5](#threatbedrock-agentcorerecommendation5) |
| [Defense Evasion](../threat-impacts.md#defense-evasion) | [Threat.BEDROCK-AGENTCORE.AbTest.1](#threatbedrock-agentcoreabtest1), [Threat.BEDROCK-AGENTCORE.AgentRuntime.2](#threatbedrock-agentcoreagentruntime2), [Threat.BEDROCK-AGENTCORE.AgentRuntime.3](#threatbedrock-agentcoreagentruntime3), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.2](#threatbedrock-agentcorebatchevaluation2), [Threat.BEDROCK-AGENTCORE.BatchEvaluation.3](#threatbedrock-agentcorebatchevaluation3), [Threat.BEDROCK-AGENTCORE.CodeInterpreter.1](#threatbedrock-agentcorecodeinterpreter1), [Threat.BEDROCK-AGENTCORE.CodeInterpreter.2](#threatbedrock-agentcorecodeinterpreter2), [Threat.BEDROCK-AGENTCORE.Memory.2](#threatbedrock-agentcorememory2), [Threat.BEDROCK-AGENTCORE.Memory.3](#threatbedrock-agentcorememory3), [Threat.BEDROCK-AGENTCORE.Recommendation.2](#threatbedrock-agentcorerecommendation2), [Threat.BEDROCK-AGENTCORE.Recommendation.3](#threatbedrock-agentcorerecommendation3) |
| [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Threat.BEDROCK-AGENTCORE.BatchEvaluation.1](#threatbedrock-agentcorebatchevaluation1), [Threat.BEDROCK-AGENTCORE.Recommendation.1](#threatbedrock-agentcorerecommendation1) |
| [Initial Access](../threat-impacts.md#initial-access) | [Threat.BEDROCK-AGENTCORE.AbTest.2](#threatbedrock-agentcoreabtest2) |
