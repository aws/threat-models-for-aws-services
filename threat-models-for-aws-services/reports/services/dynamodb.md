# Amazon DynamoDB (dynamodb) Threat Model

[Threat Summary](#threat-summary) · [Threats](#threats) · [Mitigations](#mitigations) · [Implementations](#implementations) · [Appendix](#appendix)

## Overview

Amazon DynamoDB is a serverless, fully managed NoSQL database service. It supports both key-value and document data models and delivers consistent single-digit millisecond response times. Storage and throughput scale automatically with demand, with no servers to provision, patch, or maintain, and no maintenance windows or version upgrades.

28 threats and 24 mitigations across 4 resource types and 58 API operations. 16 of 38 mitigation goals are covered, implemented by 90 implementations.

Source service model: [dynamodb](https://github.com/aws/api-models-aws/blob/a0767ac42e279dfc8b311564c33436f8abfd3031/models/dynamodb/service/2012-08-10/dynamodb-2012-08-10.json)

---

## Threat Summary

### [Cross-service confused deputy problem through resource-based policies](../potential-threats.md#cross-service-confused-deputy-problem-through-resource-based-policies)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.17](#threatdynamodbtable17) | Untrusted IAM principal, with access to configure an AWS service that has been granted access to an Amazon DynamoDB table through its resource-based policy, may be able to cause that AWS service to interact with the table using the principal's data or context. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9) |

### [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Stream.7](#threatdynamodbstream7) | Threat actor, with network adjacency to the connection path between clients and an Amazon DynamoDB stream that has not been configured to encrypt data in transit, can capture stream network traffic and read data in transit that has not been encrypted. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5) |
| [Threat.DYNAMODB.Table.18](#threatdynamodbtable18) | Threat actor, with network adjacency to the connection path between clients and an Amazon DynamoDB table that has not been configured to encrypt data in transit, can capture table network traffic and read data in transit that has not been encrypted, including item attribute values, table and index names, and partition and sort key values. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10) |

### [Data inaccessibility through resource-based policy lockout](../potential-threats.md#data-inaccessibility-through-resource-based-policy-lockout)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Stream.1](#threatdynamodbstream1) | IAM principal, with IAM permissions to modify a resource-based policy governing an Amazon DynamoDB stream, can make the stream inaccessible by modifying the stream resource-based policy. | [Data Destruction](../threat-impacts.md#data-destruction), [Service Stop](../threat-impacts.md#service-stop) | [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7) |
| [Threat.DYNAMODB.Table.2](#threatdynamodbtable2) | IAM principal, with IAM permissions to modify a resource-based policy governing an Amazon DynamoDB table, can make the table inaccessible by modifying the table resource-based policy. | [Account Access Removal](../threat-impacts.md#account-access-removal), [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12) |

### [Data loss through encryption key deletion](../potential-threats.md#data-loss-through-encryption-key-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.3](#threatdynamodbtable3) | IAM principal, with IAM permissions to delete the AWS KMS customer managed key used by an Amazon DynamoDB table, can delete the AWS KMS customer managed key used by the table, which drives the table to `INACCESSIBLE_ENCRYPTION_CREDENTIALS`, blocks all reads and writes, and archives the table so that its items, secondary indexes, stream and backups can no longer be accessed. | [Data Destruction](../threat-impacts.md#data-destruction), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7) |

### [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.5](#threatdynamodbtable5) | IAM principal, with IAM permissions to create or modify data in an Amazon DynamoDB table, can encrypt the items and attributes stored in the table with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7) |

### [Data loss through resource deletion](../potential-threats.md#data-loss-through-resource-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.1](#threatdynamodbtable1) | IAM principal, with IAM permissions to delete an Amazon DynamoDB table, can delete the table and the items that the table contains. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14) |

### [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.9](#threatdynamodbtable9) | IAM principal, with IAM permissions to create or modify an Amazon DynamoDB table that supports CloudWatch Contributor Insights, can create or configure the table without CloudWatch Contributor Insights enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |

### [Direct data deletion](../potential-threats.md#direct-data-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.14](#threatdynamodbtable14) | IAM principal, with IAM permissions to delete items from an Amazon DynamoDB table, can delete items from the table while the table itself remains intact and continues serving requests. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4) |

### [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.1](#threatdynamodb1) | Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon DynamoDB service API calls. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.2](#mitigationdynamodb2) |

### [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Stream.6](#threatdynamodbstream6) | Untrusted IAM principal, that has been granted access to my Amazon DynamoDB stream through its resource-based policy, can use my stream. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4) |
| [Threat.DYNAMODB.Table.13](#threatdynamodbtable13) | Untrusted IAM principal, that has been granted access to my Amazon DynamoDB table through its resource-based policy, can use my table. | [Data Collection](../threat-impacts.md#data-collection), [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8) |

### [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.7](#threatdynamodbtable7) | IAM principal, with IAM permissions to create or modify an Amazon DynamoDB table, can encrypt the table with an AWS owned or AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5) |

### [Monitoring disablement](../potential-threats.md#monitoring-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.4](#threatdynamodbtable4) | IAM principal, with IAM permissions to modify an Amazon DynamoDB table's CloudWatch Contributor Insights, can disable CloudWatch Contributor Insights for the table, deleting the per-key insight rules and stopping publication of the `ConsumedThroughputUnits` and `ThrottleCount` metrics. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |

### [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Stream.5](#threatdynamodbstream5) | IAM principal, with IAM permissions to read or modify my Amazon DynamoDB stream and access to an unexpected network, can interact with my stream from an unexpected network. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1) |
| [Threat.DYNAMODB.Table.11](#threatdynamodbtable11) | IAM principal, with IAM permissions on my Amazon DynamoDB table and access to an unexpected network, can interact with my table from an unexpected network. | [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1) |

### [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Stream.2](#threatdynamodbstream2) | IAM principal, with IAM permissions to create or modify an Amazon DynamoDB stream, can configure the stream without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6) |
| [Threat.DYNAMODB.Table.6](#threatdynamodbtable6) | IAM principal, with IAM permissions to create or modify an Amazon DynamoDB table, can configure the table without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11) |

### [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.2](#threatdynamodb2) | IAM principal, with IAM permissions on untrusted Amazon DynamoDB resources, can interact with those untrusted resources from a governed VPC network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1) |

### [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Stream.4](#threatdynamodbstream4) | My IAM principal, with IAM permissions on an untrusted Amazon DynamoDB stream, can interact with the untrusted stream, including sending data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.Stream.2](#mitigationdynamodbstream2) |
| [Threat.DYNAMODB.Table.10](#threatdynamodbtable10) | My IAM principal, with IAM permissions on an untrusted Amazon DynamoDB table, can interact with the untrusted table, including sending data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3) |

### [Service disruption through resource deletion](../potential-threats.md#service-disruption-through-resource-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.GlobalTable.1](#threatdynamodbglobaltable1) | IAM principal, with IAM permissions to modify or delete an Amazon DynamoDB global table, can delete the global table by removing all of its replicas, stopping cross-Region replication and removing multi-Region read and write availability. | [Service Stop](../threat-impacts.md#service-stop) | [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1) |

### [Unauthorized data access through AWS managed KMS key](../potential-threats.md#unauthorized-data-access-through-aws-managed-kms-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.15](#threatdynamodbtable15) | IAM principal, with unintended IAM permissions to read items from an Amazon DynamoDB table that is encrypted with an AWS managed KMS key in the same account as the principal, can read the items in the table. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5) |

### [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.16](#threatdynamodbtable16) | IAM principal, with unintended IAM permissions to read items from an Amazon DynamoDB table that has been encrypted with an AWS owned KMS key, can read items in the table in plaintext, because decryption is transparent to the caller and the AWS owned KMS key provides no additional authorization barrier. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5) |

### [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Table.12](#threatdynamodbtable12) | IAM principal, with unintended IAM permissions to read items from an Amazon DynamoDB table that has not been encrypted at rest with a KMS customer managed key, can read plaintext items in the table, including harvesting every attribute of every item through `Scan` or a PartiQL statement. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5) |

### [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DYNAMODB.Stream.3](#threatdynamodbstream3) | Threat actor, with a method for enumerating public Amazon DynamoDB streams, or prior knowledge of an existing public stream, can access a stream that has been configured to allow public access and read the item-level change records it retains for the parent table. | [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3) |
| [Threat.DYNAMODB.Table.8](#threatdynamodbtable8) | Threat actor, with a method for enumerating public Amazon DynamoDB tables, or prior knowledge of an existing public table, can access a table that has been configured to allow public access. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6) |

---

## Threats

Each scoping statement below describes the exposure the threat's mapped mitigations address. It assumes those mitigations are not applied and makes no claim about your resources.

### Threat.DYNAMODB.1

Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon DynamoDB service API calls.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.2](#mitigationdynamodb2)

**Frameworks:** MITRE ATT&CK [T1074.002](https://attack.mitre.org/techniques/T1074/002/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)): An IAM principal from outside your organization can reach the Amazon DynamoDB API from your VPC. That access enables transfer of data from your VPC to DynamoDB tables in an AWS account outside your organization.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:BatchGetItem` | `$.RequestItems{key}.Keys[*]{key}.S` |
| `dynamodb:DeleteItem` | `$.TableName` |
| `dynamodb:DeleteTable` | `$.TableName` |
| `dynamodb:ExecuteStatement` | `$.Statement` |
| `dynamodb:ExportTableToPointInTime` | `$.S3Bucket`, `$.S3BucketOwner`, `$.TableArn` |
| `dynamodb:GetItem` | `$.TableName` |
| `dynamodb:PutItem` | `$.Item{key}.S`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `dynamodb:Query` | `$.ProjectionExpression`, `$.TableName` |
| `dynamodb:Scan` | `$.Select`, `$.TableName` |
| `dynamodb:UpdateItem` | `$.TableName`, `$.UpdateExpression` |

### Threat.DYNAMODB.2

IAM principal, with IAM permissions on untrusted Amazon DynamoDB resources, can interact with those untrusted resources from a governed VPC network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.1](#mitigationdynamodb1)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)): An IAM principal in your VPC can access an Amazon DynamoDB table in an account outside your organization. That access enables reading data from that table or transferring data from your VPC into it.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:BatchGetItem` | *(any parameter)* |
| `dynamodb:BatchWriteItem` | *(any parameter)* |
| `dynamodb:DeleteItem` | `$.TableName` |
| `dynamodb:DeleteResourcePolicy` | `$.ResourceArn` |
| `dynamodb:DeleteTable` | `$.TableName` |
| `dynamodb:ExecuteStatement` | `$.Statement` |
| `dynamodb:ExportTableToPointInTime` | `$.S3Bucket`, `$.S3BucketOwner`, `$.TableArn` |
| `dynamodb:GetItem` | `$.TableName` |
| `dynamodb:ImportTable` | `$.S3BucketSource.S3Bucket`, `$.S3BucketSource.S3BucketOwner` |
| `dynamodb:PutItem` | `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `dynamodb:Query` | `$.TableName` |
| `dynamodb:Scan` | `$.TableName` |
| `dynamodb:UpdateTable` | `$.TableName` |

### Threat.DYNAMODB.GlobalTable.1

IAM principal, with IAM permissions to modify or delete an Amazon DynamoDB global table, can delete the global table by removing all of its replicas, stopping cross-Region replication and removing multi-Region read and write availability.

**Impacts:** [Service Stop](../threat-impacts.md#service-stop)

**Mitigations:** [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1)

**Frameworks:** MITRE ATT&CK [T1489](https://attack.mitre.org/techniques/T1489/)

**Scoping** (for [Service disruption through resource deletion](../potential-threats.md#service-disruption-through-resource-deletion)): An IAM principal with permissions to modify or delete an Amazon DynamoDB global table can remove all of its replica Regions. Removing every replica deletes the global table and stops cross-Region replication and multi-Region read and write availability.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:DeleteTable` | `$.TableName` |
| `dynamodb:UpdateGlobalTable` | `$.GlobalTableName`, `$.ReplicaUpdates[*].Delete.RegionName` |
| `iam:DeleteServiceLinkedRole` | `$.RoleName` |
| `kms:ScheduleKeyDeletion` | `$.KeyId` |

### Threat.DYNAMODB.Stream.1

IAM principal, with IAM permissions to modify a resource-based policy governing an Amazon DynamoDB stream, can make the stream inaccessible by modifying the stream resource-based policy.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction), [Service Stop](../threat-impacts.md#service-stop)

**Mitigations:** [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/), [T1489](https://attack.mitre.org/techniques/T1489/)

**Scoping** (for [Data inaccessibility through resource-based policy lockout](../potential-threats.md#data-inaccessibility-through-resource-based-policy-lockout)): An IAM principal with permissions to modify the stream resource-based policy can call `PutResourcePolicy` with a policy that blocks stream reads. That policy can also block later changes to itself, leaving the stored change log unreachable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn`, `$.x-amz-confirm-remove-self-resource-access` |

### Threat.DYNAMODB.Stream.2

IAM principal, with IAM permissions to create or modify an Amazon DynamoDB stream, can configure the stream without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): A principal that manages a DynamoDB stream can do so without applying the tags your organization requires for that stream. The stream may then fall outside your tag-based cost allocation and ownership tracking.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.StreamSpecification.StreamEnabled` |
| `dynamodb:TagResource` | `$.ResourceArn`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `dynamodb:UntagResource` | `$.ResourceArn`, `$.TagKeys[*]` |
| `dynamodb:UpdateTable` | `$.StreamSpecification.StreamEnabled`, `$.StreamSpecification.StreamViewType` |

### Threat.DYNAMODB.Stream.3

Threat actor, with a method for enumerating public Amazon DynamoDB streams, or prior knowledge of an existing public stream, can access a stream that has been configured to allow public access and read the item-level change records it retains for the parent table.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/)

**Scoping** (for [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)): The resource-based policy on your stream can grant its read actions, such as `dynamodb:GetRecords`, to principals outside your organization. Those principals can then read the item-level change records the stream retains, without holding any permission on the parent table.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb-streams:DescribeStream` | `$.StreamArn` |
| `dynamodb-streams:GetRecords` | `$.Limit`, `$.ShardIterator` |
| `dynamodb-streams:GetShardIterator` | `$.ShardId`, `$.ShardIteratorType`, `$.StreamArn` |

### Threat.DYNAMODB.Stream.4

My IAM principal, with IAM permissions on an untrusted Amazon DynamoDB stream, can interact with the untrusted stream, including sending data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.Stream.2](#mitigationdynamodbstream2)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): Amazon DynamoDB streams support cross-account access granted through their resource-based policies. An IAM principal in your organization can therefore read item-level change records from a stream owned by an account outside your organization.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:TagResource` | `$.ResourceArn`, `$.Tags[*].Value` |
| `dynamodb-streams:GetRecords` | `$.Limit`, `$.ShardIterator` |
| `dynamodb-streams:GetShardIterator` | `$.ShardIteratorType`, `$.StreamArn` |
| `lambda:CreateEventSourceMapping` | `$.EventSourceArn`, `$.StartingPosition` |

### Threat.DYNAMODB.Stream.5

IAM principal, with IAM permissions to read or modify my Amazon DynamoDB stream and access to an unexpected network, can interact with my stream from an unexpected network.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1537](https://attack.mitre.org/techniques/T1537/), [T1562.007](https://attack.mitre.org/techniques/T1562/007/)

**Scoping** (for [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)): An IAM principal with valid credentials can initiate requests against your Amazon DynamoDB stream from an unexpected network. From that network, a principal with permissions on the stream can read item-level change records, or rewrite or remove its resource-based policy.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:DeleteResourcePolicy` | `$.ResourceArn` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `dynamodb-streams:GetRecords` | `$.Limit`, `$.ShardIterator` |
| `dynamodb-streams:GetShardIterator` | `$.ShardIteratorType`, `$.StreamArn` |

### Threat.DYNAMODB.Stream.6

Untrusted IAM principal, that has been granted access to my Amazon DynamoDB stream through its resource-based policy, can use my stream.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)): A resource-based policy on your stream can grant read access to identities outside your organization. Those identities can then call `GetShardIterator` and `GetRecords` to read the item-level change records the stream retains for 24 hours.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:TagResource` | `$.ResourceArn`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `dynamodb:UntagResource` | `$.ResourceArn`, `$.TagKeys[*]` |
| `dynamodb-streams:GetRecords` | `$.Limit`, `$.ShardIterator` |
| `dynamodb-streams:GetShardIterator` | `$.ShardId`, `$.ShardIteratorType`, `$.StreamArn` |

### Threat.DYNAMODB.Stream.7

Threat actor, with network adjacency to the connection path between clients and an Amazon DynamoDB stream that has not been configured to encrypt data in transit, can capture stream network traffic and read data in transit that has not been encrypted.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Protocols:** `HTTP`

**Scoping** (for [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)): Clients consume your stream by calling `DescribeStream`, `GetShardIterator` and `GetRecords` across a network path. When a connection is not encrypted, a threat actor adjacent to that path can read those exchanges and the stream records they return.

### Threat.DYNAMODB.Table.1

IAM principal, with IAM permissions to delete an Amazon DynamoDB table, can delete the table and the items that the table contains.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Scoping** (for [Data loss through resource deletion](../potential-threats.md#data-loss-through-resource-deletion)): A single `DeleteTable` call removes the table and all of the items it holds. The caller needs only the table name in `$.TableName`, not an item-level permission such as `dynamodb:DeleteItem`.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudformation:DeleteStack` | `$.StackName` |
| `dynamodb:DeleteTable` | `$.TableName` |

### Threat.DYNAMODB.Table.2

IAM principal, with IAM permissions to modify a resource-based policy governing an Amazon DynamoDB table, can make the table inaccessible by modifying the table resource-based policy.

**Impacts:** [Account Access Removal](../threat-impacts.md#account-access-removal), [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/), [T1531](https://attack.mitre.org/techniques/T1531/)

**Scoping** (for [Data inaccessibility through resource-based policy lockout](../potential-threats.md#data-inaccessibility-through-resource-based-policy-lockout)): A principal with permission to call `PutResourcePolicy` can replace the resource-based policy on your table with one that blocks access for all principals. The table and its stored items remain in place, but legitimate principals can no longer reach them.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.ResourcePolicy`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn`, `$.x-amz-confirm-remove-self-resource-access` |

### Threat.DYNAMODB.Table.3

IAM principal, with IAM permissions to delete the AWS KMS customer managed key used by an Amazon DynamoDB table, can delete the AWS KMS customer managed key used by the table, which drives the table to `INACCESSIBLE_ENCRYPTION_CREDENTIALS`, blocks all reads and writes, and archives the table so that its items, secondary indexes, stream and backups can no longer be accessed.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/), [T1490](https://attack.mitre.org/techniques/T1490/)

**Scoping** (for [Data loss through encryption key deletion](../potential-threats.md#data-loss-through-encryption-key-deletion)): When your DynamoDB table uses a customer managed key, its items, secondary indexes, stream and backups all depend on that one key. An IAM principal with permissions to delete that key can render the table and its backups inaccessible.

**Request parameters**

| Operation | Parameters |
|---|---|
| `kms:DeleteImportedKeyMaterial` | `$.KeyId`, `$.KeyMaterialId` |
| `kms:DisconnectCustomKeyStore` | `$.CustomKeyStoreId` |
| `kms:ScheduleKeyDeletion` | `$.KeyId`, `$.PendingWindowInDays` |

### Threat.DYNAMODB.Table.4

IAM principal, with IAM permissions to modify an Amazon DynamoDB table's CloudWatch Contributor Insights, can disable CloudWatch Contributor Insights for the table, deleting the per-key insight rules and stopping publication of the `ConsumedThroughputUnits` and `ThrottleCount` metrics.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): CloudWatch Contributor Insights adds per-key metrics for your DynamoDB table beyond the default table-level metrics. A principal with permission to call `UpdateContributorInsights` can disable it, deleting the insight rules and stopping publication of those metrics.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:UpdateContributorInsights` | `$.ContributorInsightsAction`, `$.ContributorInsightsMode`, `$.IndexName`, `$.TableName` |

### Threat.DYNAMODB.Table.5

IAM principal, with IAM permissions to create or modify data in an Amazon DynamoDB table, can encrypt the items and attributes stored in the table with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/), [T1490](https://attack.mitre.org/techniques/T1490/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal calling `CreateTable` or `UpdateTable` can set `$.SSESpecification.KMSMasterKeyId` to a key with imported key material or an external key store. The holder of that key material can delete it or disconnect the external key store, leaving your table archived and unreadable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.SSESpecification.KMSMasterKeyId`, `$.TableName` |
| `dynamodb:ImportTable` | `$.TableCreationParameters.SSESpecification.KMSMasterKeyId`, `$.TableCreationParameters.TableName` |
| `dynamodb:RestoreTableFromBackup` | `$.SSESpecificationOverride.KMSMasterKeyId`, `$.TargetTableName` |
| `dynamodb:RestoreTableToPointInTime` | `$.SSESpecificationOverride.KMSMasterKeyId`, `$.TargetTableName` |
| `dynamodb:UpdateTable` | `$.ReplicaUpdates[*].Create.KMSMasterKeyId`, `$.ReplicaUpdates[*].Update.KMSMasterKeyId`, `$.SSESpecification.KMSMasterKeyId`, `$.TableName` |

### Threat.DYNAMODB.Table.6

IAM principal, with IAM permissions to create or modify an Amazon DynamoDB table, can configure the table without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal can create a table without the required tags, or remove those tags later by calling `UntagResource`. A table left without those tags may fall outside the ownership attribution processes that key on tags.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.TableName`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `dynamodb:ImportTable` | `$.TableCreationParameters.TableName` |
| `dynamodb:RestoreTableFromBackup` | `$.BackupArn`, `$.TargetTableName` |
| `dynamodb:RestoreTableToPointInTime` | `$.SourceTableArn`, `$.TargetTableName` |
| `dynamodb:TagResource` | `$.ResourceArn`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `dynamodb:UntagResource` | `$.ResourceArn`, `$.TagKeys[*]` |

### Threat.DYNAMODB.Table.7

IAM principal, with IAM permissions to create or modify an Amazon DynamoDB table, can encrypt the table with an AWS owned or AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal who can create or modify your table also selects the AWS KMS key type that protects it at rest. Both the default AWS owned key and the AWS managed key limit your governance of the key's policy and lifecycle.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.SSESpecification.Enabled`, `$.SSESpecification.KMSMasterKeyId`, `$.SSESpecification.SSEType`, `$.TableName` |
| `dynamodb:ImportTable` | `$.TableCreationParameters.SSESpecification.Enabled`, `$.TableCreationParameters.SSESpecification.KMSMasterKeyId`, `$.TableCreationParameters.SSESpecification.SSEType`, `$.TableCreationParameters.TableName` |
| `dynamodb:RestoreTableFromBackup` | `$.SSESpecificationOverride.Enabled`, `$.SSESpecificationOverride.KMSMasterKeyId`, `$.SSESpecificationOverride.SSEType`, `$.TargetTableName` |
| `dynamodb:RestoreTableToPointInTime` | `$.SSESpecificationOverride.Enabled`, `$.SSESpecificationOverride.KMSMasterKeyId`, `$.SSESpecificationOverride.SSEType`, `$.TargetTableName` |
| `dynamodb:UpdateTable` | `$.ReplicaUpdates[*].Create.KMSMasterKeyId`, `$.ReplicaUpdates[*].Update.KMSMasterKeyId`, `$.SSESpecification.Enabled`, `$.SSESpecification.KMSMasterKeyId`, `$.SSESpecification.SSEType`, `$.TableName` |

### Threat.DYNAMODB.Table.8

Threat actor, with a method for enumerating public Amazon DynamoDB tables, or prior knowledge of an existing public table, can access a table that has been configured to allow public access.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)): A DynamoDB table can be configured to allow access from any AWS account rather than a defined set of identities. A threat actor with knowledge of such a table can read your items with `dynamodb:Scan` or export them with `dynamodb:ExportTableToPointInTime`.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:BatchGetItem` | `$.RequestItems{key}.ProjectionExpression` |
| `dynamodb:DescribeTable` | `$.TableName` |
| `dynamodb:ExportTableToPointInTime` | `$.S3Bucket`, `$.S3BucketOwner`, `$.TableArn` |
| `dynamodb:GetItem` | `$.ProjectionExpression`, `$.TableName` |
| `dynamodb:ListTagsOfResource` | `$.ResourceArn` |
| `dynamodb:Query` | `$.KeyConditionExpression`, `$.TableName` |
| `dynamodb:Scan` | `$.IndexName`, `$.Segment`, `$.Select`, `$.TableName`, `$.TotalSegments` |
| `dynamodb:TransactGetItems` | `$.TransactItems[*].Get.TableName` |

### Threat.DYNAMODB.Table.9

IAM principal, with IAM permissions to create or modify an Amazon DynamoDB table that supports CloudWatch Contributor Insights, can create or configure the table without CloudWatch Contributor Insights enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): `CreateTable` exposes no Contributor Insights parameter, so a new DynamoDB table starts without per-partition-key `ConsumedThroughputUnits` and `ThrottleCount` metric series. Without that per-key detail, concentrated key access or whole-table enumeration through `Scan` blends into table-level aggregates.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch:DescribeInsightRules` | *(any parameter)* |
| `dynamodb:BatchGetItem` | `$.RequestItems{key}.Keys[*]{key}.S` |
| `dynamodb:BatchWriteItem` | `$.RequestItems{key}[*].DeleteRequest.Key{key}.S` |
| `dynamodb:CreateTable` | `$.TableName` |
| `dynamodb:DeleteItem` | `$.TableName` |
| `dynamodb:GetItem` | `$.Key{key}.S`, `$.TableName` |
| `dynamodb:ImportTable` | `$.TableCreationParameters.TableName` |
| `dynamodb:PutItem` | `$.TableName` |
| `dynamodb:Query` | `$.KeyConditionExpression`, `$.TableName` |
| `dynamodb:RestoreTableFromBackup` | `$.TargetTableName` |
| `dynamodb:RestoreTableToPointInTime` | `$.TargetTableName` |
| `dynamodb:Scan` | `$.Limit`, `$.Segment`, `$.TableName`, `$.TotalSegments` |
| `dynamodb:UpdateItem` | `$.TableName` |

### Threat.DYNAMODB.Table.10

My IAM principal, with IAM permissions on an untrusted Amazon DynamoDB table, can interact with the untrusted table, including sending data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): Amazon DynamoDB tables support cross-account access granted through their resource-based policies. An IAM principal in your organization can therefore read items from, or write items to, a table owned by an account outside your organization.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:DeleteItem` | `$.TableName` |
| `dynamodb:DeleteTable` | `$.TableName` |
| `dynamodb:ExportTableToPointInTime` | `$.S3Bucket`, `$.S3BucketOwner`, `$.TableArn` |
| `dynamodb:GetItem` | `$.TableName` |
| `dynamodb:ImportTable` | `$.S3BucketSource.S3Bucket`, `$.S3BucketSource.S3BucketOwner` |
| `dynamodb:PutItem` | `$.Item{key}.S`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `dynamodb:Query` | `$.TableName` |
| `dynamodb:Scan` | `$.TableName` |
| `dynamodb:UpdateItem` | `$.TableName` |
| `dynamodb:UpdateTable` | `$.TableName` |

### Threat.DYNAMODB.Table.11

IAM principal, with IAM permissions on my Amazon DynamoDB table and access to an unexpected network, can interact with my table from an unexpected network.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1485](https://attack.mitre.org/techniques/T1485/), [T1537](https://attack.mitre.org/techniques/T1537/), [T1556.009](https://attack.mitre.org/techniques/T1556/009/)

**Scoping** (for [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)): Amazon DynamoDB authorizes requests to your table on the signed IAM credential, not on the network a request comes from. An IAM principal with permissions on the table can read, modify, and export items from an unexpected network, or reconfigure and delete the table.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:DeleteItem` | `$.TableName` |
| `dynamodb:DeleteResourcePolicy` | `$.ResourceArn` |
| `dynamodb:DeleteTable` | `$.TableName` |
| `dynamodb:ExecuteStatement` | `$.Statement` |
| `dynamodb:ExportTableToPointInTime` | `$.S3Bucket`, `$.S3BucketOwner`, `$.TableArn` |
| `dynamodb:GetItem` | `$.TableName` |
| `dynamodb:PutItem` | `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `dynamodb:Query` | `$.TableName` |
| `dynamodb:Scan` | `$.TableName` |
| `dynamodb:UpdateItem` | `$.TableName`, `$.UpdateExpression` |
| `dynamodb:UpdateTable` | `$.TableName` |

### Threat.DYNAMODB.Table.12

IAM principal, with unintended IAM permissions to read items from an Amazon DynamoDB table that has not been encrypted at rest with a KMS customer managed key, can read plaintext items in the table, including harvesting every attribute of every item through `Scan` or a PartiQL statement.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/)

**Scoping** (for [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)): Without a customer managed key, your table's items are encrypted under an AWS owned key you cannot audit or disable. A principal with data-plane read permissions can call `Scan` or `ExecuteStatement` and read plaintext items.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:BatchExecuteStatement` | `$.Statements[*].Statement` |
| `dynamodb:BatchGetItem` | `$.RequestItems{key}.Keys[*]{key}.S`, `$.RequestItems{key}.ProjectionExpression` |
| `dynamodb:CreateTable` | `$.SSESpecification.Enabled`, `$.SSESpecification.KMSMasterKeyId`, `$.TableName` |
| `dynamodb:ExecuteStatement` | `$.Statement` |
| `dynamodb:GetItem` | `$.ConsistentRead`, `$.Key{key}.S`, `$.ProjectionExpression`, `$.TableName` |
| `dynamodb:Query` | `$.IndexName`, `$.KeyConditionExpression`, `$.ProjectionExpression`, `$.Select`, `$.TableName` |
| `dynamodb:Scan` | `$.IndexName`, `$.Segment`, `$.Select`, `$.TableName`, `$.TotalSegments` |
| `dynamodb:SearchVectors` | `$.IndexName`, `$.ProjectionExpression`, `$.TableName` |
| `dynamodb:TransactGetItems` | `$.TransactItems[*].Get.Key{key}.S`, `$.TransactItems[*].Get.ProjectionExpression`, `$.TransactItems[*].Get.TableName` |
| `dynamodb:UpdateTable` | `$.SSESpecification.Enabled`, `$.SSESpecification.KMSMasterKeyId` |

### Threat.DYNAMODB.Table.13

Untrusted IAM principal, that has been granted access to my Amazon DynamoDB table through its resource-based policy, can use my table.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1485](https://attack.mitre.org/techniques/T1485/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)): A resource-based policy on your Amazon DynamoDB table can grant DynamoDB actions on the table and its items to identities outside your organization. DynamoDB enforces such cross-account grants, allowing those identities to read, change, or delete your table data.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:DeleteItem` | `$.Key{key}.S`, `$.TableName` |
| `dynamodb:DeleteTable` | `$.TableName` |
| `dynamodb:GetItem` | `$.ProjectionExpression`, `$.TableName` |
| `dynamodb:PutItem` | `$.Item{key}.S`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `dynamodb:Query` | `$.KeyConditionExpression`, `$.TableName` |
| `dynamodb:Scan` | `$.Select`, `$.TableName` |
| `dynamodb:UpdateItem` | `$.TableName`, `$.UpdateExpression` |
| `dynamodb:UpdateTable` | `$.TableName` |

### Threat.DYNAMODB.Table.14

IAM principal, with IAM permissions to delete items from an Amazon DynamoDB table, can delete items from the table while the table itself remains intact and continues serving requests.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Scoping** (for [Direct data deletion](../potential-threats.md#direct-data-deletion)): Amazon DynamoDB exposes item-level delete operations on your table, including `DeleteItem`, `BatchWriteItem` and PartiQL `DELETE` statements. An IAM principal granted actions such as `dynamodb:DeleteItem` can delete items while the table remains intact and continues serving requests.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:BatchExecuteStatement` | `$.Statements[*].Statement` |
| `dynamodb:BatchWriteItem` | `$.RequestItems{key}[*].DeleteRequest.Key{key}.B`, `$.RequestItems{key}[*].DeleteRequest.Key{key}.N`, `$.RequestItems{key}[*].DeleteRequest.Key{key}.S` |
| `dynamodb:DeleteItem` | `$.ConditionExpression`, `$.Key{key}.B`, `$.Key{key}.N`, `$.Key{key}.S`, `$.TableName` |
| `dynamodb:ExecuteStatement` | `$.Statement` |
| `dynamodb:ExecuteTransaction` | `$.TransactStatements[*].Statement` |
| `dynamodb:TransactWriteItems` | `$.TransactItems[*].Delete.Key{key}.N`, `$.TransactItems[*].Delete.Key{key}.S`, `$.TransactItems[*].Delete.TableName` |
| `dynamodb:UpdateItem` | `$.AttributeUpdates{key}.Action`, `$.Key{key}.S`, `$.TableName`, `$.UpdateExpression` |

### Threat.DYNAMODB.Table.15

IAM principal, with unintended IAM permissions to read items from an Amazon DynamoDB table that is encrypted with an AWS managed KMS key in the same account as the principal, can read the items in the table.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/)

**Scoping** (for [Unauthorized data access through AWS managed KMS key](../potential-threats.md#unauthorized-data-access-through-aws-managed-kms-key)): Your table can be encrypted with the AWS managed key `aws/dynamodb`, whose key policy AWS establishes and you do not manage. IAM permissions for read actions such as `dynamodb:GetItem`, `dynamodb:Query`, and `dynamodb:Scan` then govern which principals in your account retrieve plaintext items.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:BatchExecuteStatement` | `$.Statements[*].Statement` |
| `dynamodb:BatchGetItem` | `$.RequestItems{key}.Keys[*]{key}.S`, `$.RequestItems{key}.ProjectionExpression` |
| `dynamodb:ExecuteStatement` | `$.Statement` |
| `dynamodb:GetItem` | `$.Key{key}.S`, `$.ProjectionExpression`, `$.TableName` |
| `dynamodb:Query` | `$.KeyConditionExpression`, `$.Select`, `$.TableName` |
| `dynamodb:Scan` | `$.IndexName`, `$.Select`, `$.TableName`, `$.TotalSegments` |
| `dynamodb:SearchVectors` | `$.IndexName`, `$.TableName`, `$.TopK` |
| `dynamodb:TransactGetItems` | `$.TransactItems[*].Get.Key{key}.S`, `$.TransactItems[*].Get.TableName` |

### Threat.DYNAMODB.Table.16

IAM principal, with unintended IAM permissions to read items from an Amazon DynamoDB table that has been encrypted with an AWS owned KMS key, can read items in the table in plaintext, because decryption is transparent to the caller and the AWS owned KMS key provides no additional authorization barrier.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/)

**Scoping** (for [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)): Your table stores items encrypted at rest with an AWS owned KMS key whenever no encryption type is specified. That key type gives you no key policy to govern, no audit record of key use, and no control over the key lifecycle.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:BatchExecuteStatement` | `$.Statements[*].ConsistentRead`, `$.Statements[*].Statement` |
| `dynamodb:BatchGetItem` | `$.RequestItems{key}.ConsistentRead`, `$.RequestItems{key}.Keys[*]{key}.S`, `$.RequestItems{key}.ProjectionExpression` |
| `dynamodb:ExecuteStatement` | `$.ConsistentRead`, `$.Limit`, `$.Statement` |
| `dynamodb:GetItem` | `$.ConsistentRead`, `$.Key{key}.S`, `$.ProjectionExpression`, `$.TableName` |
| `dynamodb:Query` | `$.FilterExpression`, `$.IndexName`, `$.KeyConditionExpression`, `$.ProjectionExpression`, `$.Select`, `$.TableName` |
| `dynamodb:Scan` | `$.IndexName`, `$.ProjectionExpression`, `$.Segment`, `$.Select`, `$.TableName`, `$.TotalSegments` |
| `dynamodb:SearchVectors` | `$.IndexName`, `$.ProjectionExpression`, `$.SearchVector[*].N`, `$.TableName`, `$.TopK` |
| `dynamodb:TransactGetItems` | `$.TransactItems[*].Get.Key{key}.S`, `$.TransactItems[*].Get.ProjectionExpression`, `$.TransactItems[*].Get.TableName` |

### Threat.DYNAMODB.Table.17

Untrusted IAM principal, with access to configure an AWS service that has been granted access to an Amazon DynamoDB table through its resource-based policy, may be able to cause that AWS service to interact with the table using the principal's data or context.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Privilege Escalation](../threat-impacts.md#privilege-escalation)

**Mitigations:** [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/), [T1548.005](https://attack.mitre.org/techniques/T1548/005/)

**Scoping** (for [Cross-service confused deputy problem through resource-based policies](../potential-threats.md#cross-service-confused-deputy-problem-through-resource-based-policies)): Your table's resource-based policy can grant an AWS service such as `redshift.amazonaws.com` or `glue.amazonaws.com` access to the table. An untrusted principal who can configure that service may cause it to interact with your table using the principal's own data or context.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.ResourcePolicy` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `glue:CreateIntegration` | `$.SourceArn`, `$.TargetArn` |
| `redshift:CreateIntegration` | `$.SourceArn`, `$.TargetArn` |

### Threat.DYNAMODB.Table.18

Threat actor, with network adjacency to the connection path between clients and an Amazon DynamoDB table that has not been configured to encrypt data in transit, can capture table network traffic and read data in transit that has not been encrypted, including item attribute values, table and index names, and partition and sort key values.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Protocols:** `HTTP`

**Scoping** (for [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)): If your table carries no encryption-in-transit configuration that requires client connections to be encrypted, a network-adjacent threat actor can capture unencrypted traffic to your table and read item attribute values, table and index names, and key values.

---

## Mitigations

Each scoping statement below names the capability the mitigation goal governs on the entity and the objective of governing it. Each mitigation's implementations describe how to govern or monitor that capability.

### Mitigation.DYNAMODB.1

Amazon DynamoDB service API calls on a governed VPC network should only be made against trusted resources or AWS service-owned resources

**Goal:** [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)

**Addresses:** [Threat.DYNAMODB.2](#threatdynamodb2)

**Implemented by:** [Implementation.DYNAMODB.1](#implementationdynamodb1)

**Scoping** (for [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)): You can call the Amazon DynamoDB API from your VPC through a gateway VPC endpoint or an interface VPC endpoint powered by AWS PrivateLink. Restricting those calls to resources in your organization enforces the resource perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.DYNAMODB.2

Amazon DynamoDB service API calls on a governed VPC network should only be made by trusted identities or AWS services

**Goal:** [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)

**Addresses:** [Threat.DYNAMODB.1](#threatdynamodb1)

**Implemented by:** [Implementation.DYNAMODB.2](#implementationdynamodb2)

**Scoping** (for [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)): You can call the Amazon DynamoDB API from your VPC through a gateway VPC endpoint or an interface VPC endpoint powered by AWS PrivateLink. Restricting those calls to principals in your organization enforces the identity perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.DYNAMODB.GlobalTable.1

An Amazon DynamoDB global table should be replicated to at least one other AWS Region

**Goal:** [Cross-Region Replication](../mitigation-goals.md#cross-region-replication)

**Addresses:** [Threat.DYNAMODB.GlobalTable.1](#threatdynamodbglobaltable1)

**Implemented by:** [Implementation.DYNAMODB.GlobalTable.1](#implementationdynamodbglobaltable1)

**Scoping** (for [Cross-Region Replication](../mitigation-goals.md#cross-region-replication)): A global table in Amazon DynamoDB is a set of replica tables in different AWS Regions that each hold the same data items. Writes to one replica are propagated automatically to the other replicas.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateGlobalTable` | `$.GlobalTableName`, `$.ReplicationGroup[*].RegionName` |
| `dynamodb:UpdateGlobalTable` | `$.GlobalTableName`, `$.ReplicaUpdates[*].Create.RegionName`, `$.ReplicaUpdates[*].Delete.RegionName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:DescribeGlobalTable` | `$.GlobalTableName` |
| `dynamodb:DescribeGlobalTableSettings` | `$.GlobalTableName` |
| `dynamodb:ListGlobalTables` | `$.RegionName` |

### Mitigation.DYNAMODB.Stream.1

An Amazon DynamoDB stream should be accessed only from expected networks

**Goal:** [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)

**Addresses:** [Threat.DYNAMODB.Stream.5](#threatdynamodbstream5)

**Implemented by:** [Implementation.DYNAMODB.Stream.1](#implementationdynamodbstream1), [Implementation.DYNAMODB.Stream.2](#implementationdynamodbstream2), [Implementation.DYNAMODB.Stream.3](#implementationdynamodbstream3), [Implementation.DYNAMODB.Stream.4](#implementationdynamodbstream4)

**Scoping** (for [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)): Requests to your Amazon DynamoDB stream can originate from any network, including unexpected networks, and are authorized separately from requests to its parent table. A network perimeter helps you confirm that your stream can be accessed only from expected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Stream.2

IAM principals should be restricted to accessing only trusted Amazon DynamoDB stream resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.DYNAMODB.Stream.4](#threatdynamodbstream4)

**Implemented by:** [Implementation.DYNAMODB.Stream.5](#implementationdynamodbstream5)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): A resource-based policy on an Amazon DynamoDB stream in another account can grant your identities read access to its change-data-capture records. A resource perimeter helps confirm that your identities read those records only from streams your organization owns.

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

### Mitigation.DYNAMODB.Stream.3

An Amazon DynamoDB stream's resource-based policy should limit permissions to a subset of IAM principals, rather than granting access to all IAM principals

**Goal:** [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)

**Addresses:** [Threat.DYNAMODB.Stream.3](#threatdynamodbstream3)

**Implemented by:** [Implementation.DYNAMODB.Stream.6](#implementationdynamodbstream6), [Implementation.DYNAMODB.Stream.7](#implementationdynamodbstream7), [Implementation.DYNAMODB.Stream.8](#implementationdynamodbstream8), [Implementation.DYNAMODB.Stream.9](#implementationdynamodbstream9), [Implementation.DYNAMODB.Stream.10](#implementationdynamodbstream10)

**Scoping** (for [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)): You can attach a resource-based policy to your stream, and its `$.Policy` document names the IAM principals allowed to read stream records. A wildcard principal in that document can extend that access to every IAM principal in an account or organization.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Stream.4

An Amazon DynamoDB stream should be accessed only by trusted identities or AWS services acting on behalf of trusted resources

**Goal:** [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)

**Addresses:** [Threat.DYNAMODB.Stream.6](#threatdynamodbstream6)

**Implemented by:** [Implementation.DYNAMODB.Stream.11](#implementationdynamodbstream11), [Implementation.DYNAMODB.Stream.12](#implementationdynamodbstream12), [Implementation.DYNAMODB.Stream.13](#implementationdynamodbstream13), [Implementation.DYNAMODB.Stream.14](#implementationdynamodbstream14), [Implementation.DYNAMODB.Stream.15](#implementationdynamodbstream15)

**Scoping** (for [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)): A DynamoDB stream supports its own resource-based policy, attached directly to the stream. The policy can extend access to accounts outside your organization.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:DeleteResourcePolicy` | `$.ResourceArn` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Stream.5

An Amazon DynamoDB stream should be governed by a policy that requires all connections to be encrypted in transit

**Goal:** [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy)

**Addresses:** [Threat.DYNAMODB.Stream.7](#threatdynamodbstream7)

**Implemented by:** [Implementation.DYNAMODB.Stream.2](#implementationdynamodbstream2), [Implementation.DYNAMODB.Stream.3](#implementationdynamodbstream3), [Implementation.DYNAMODB.Stream.16](#implementationdynamodbstream16), [Implementation.DYNAMODB.Stream.17](#implementationdynamodbstream17), [Implementation.DYNAMODB.Stream.18](#implementationdynamodbstream18), [Implementation.DYNAMODB.Stream.19](#implementationdynamodbstream19)

**Scoping** (for [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy)): The DynamoDB Streams endpoint accepts both HTTP and HTTPS requests, so requests for your stream records can arrive unencrypted. DynamoDB supports resource-based policies on streams.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Stream.6

An Amazon DynamoDB stream should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.DYNAMODB.Stream.2](#threatdynamodbstream2)

**Implemented by:** [Implementation.DYNAMODB.Stream.20](#implementationdynamodbstream20), [Implementation.DYNAMODB.Stream.21](#implementationdynamodbstream21), [Implementation.DYNAMODB.Stream.22](#implementationdynamodbstream22), [Implementation.DYNAMODB.Stream.23](#implementationdynamodbstream23)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): A DynamoDB stream is independently taggable through its own stream ARN, so you can record owner and purpose directly on the stream. Stream tags are not inherited from the parent table.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:TagResource` | `$.ResourceArn`, `$.Tags[*].Key`, `$.Tags[*].Value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:ListTagsOfResource` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Stream.7

An Amazon DynamoDB stream should be configured to disallow resource-based policy modifications that would remove all policy-management access for the stream

**Goal:** [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention)

**Addresses:** [Threat.DYNAMODB.Stream.1](#threatdynamodbstream1)

**Implemented by:** [Implementation.DYNAMODB.Stream.2](#implementationdynamodbstream2), [Implementation.DYNAMODB.Stream.3](#implementationdynamodbstream3), [Implementation.DYNAMODB.Stream.24](#implementationdynamodbstream24)

**Scoping** (for [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention)): A DynamoDB stream carries its own resource-based policy that can govern the `dynamodb:PutResourcePolicy` action needed to amend that policy. DynamoDB rejects a policy that removes your own policy-management access, unless the request explicitly confirms that removal.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn`, `$.x-amz-confirm-remove-self-resource-access` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.1

An Amazon DynamoDB table should be accessed only from expected networks

**Goal:** [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)

**Addresses:** [Threat.DYNAMODB.Table.11](#threatdynamodbtable11)

**Implemented by:** [Implementation.DYNAMODB.Table.1](#implementationdynamodbtable1), [Implementation.DYNAMODB.Table.2](#implementationdynamodbtable2), [Implementation.DYNAMODB.Table.3](#implementationdynamodbtable3), [Implementation.DYNAMODB.Table.4](#implementationdynamodbtable4)

**Scoping** (for [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)): Requests to your Amazon DynamoDB table can originate from any network, including unexpected networks. A network perimeter helps you confirm that your table can be accessed only from expected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.ResourcePolicy`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.2

An Amazon DynamoDB table should have point-in-time recovery enabled

**Goal:** [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native)

**Addresses:** [Threat.DYNAMODB.Table.1](#threatdynamodbtable1), [Threat.DYNAMODB.Table.14](#threatdynamodbtable14)

**Implemented by:** [Implementation.DYNAMODB.Table.5](#implementationdynamodbtable5), [Implementation.DYNAMODB.Table.6](#implementationdynamodbtable6), [Implementation.DYNAMODB.Table.7](#implementationdynamodbtable7), [Implementation.DYNAMODB.Table.8](#implementationdynamodbtable8), [ManagedControl.DYNAMODB.Table.1](#managedcontroldynamodbtable1), [ManagedControl.DYNAMODB.Table.2](#managedcontroldynamodbtable2)

**Scoping** (for [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native)): DynamoDB provides point-in-time recovery for the items a table persists, through a native backup capability on the table's own API. The recovery window is configurable from 1 to 35 days, with per-second restore granularity.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:UpdateContinuousBackups` | `$.PointInTimeRecoverySpecification.PointInTimeRecoveryEnabled`, `$.PointInTimeRecoverySpecification.RecoveryPeriodInDays`, `$.TableName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:DescribeContinuousBackups` | `$.TableName` |

### Mitigation.DYNAMODB.Table.3

IAM principals should be restricted to accessing only trusted Amazon DynamoDB table resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.DYNAMODB.Table.10](#threatdynamodbtable10)

**Implemented by:** [Implementation.DYNAMODB.Table.9](#implementationdynamodbtable9)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): A resource-based policy on an Amazon DynamoDB table in another account can grant your identities access to the table and its items. A resource perimeter helps confirm that your identities read and write items only in tables your organization owns.

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

### Mitigation.DYNAMODB.Table.4

An Amazon DynamoDB table should be protected by an AWS Backup backup plan

**Goal:** [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup)

**Addresses:** [Threat.DYNAMODB.Table.1](#threatdynamodbtable1), [Threat.DYNAMODB.Table.14](#threatdynamodbtable14)

**Implemented by:** [Implementation.DYNAMODB.Table.10](#implementationdynamodbtable10), [Implementation.DYNAMODB.Table.11](#implementationdynamodbtable11), [Implementation.DYNAMODB.Table.12](#implementationdynamodbtable12), [ManagedControl.DYNAMODB.Table.3](#managedcontroldynamodbtable3), [ManagedControl.DYNAMODB.Table.4](#managedcontroldynamodbtable4), [ManagedControl.DYNAMODB.Table.5](#managedcontroldynamodbtable5), [ManagedControl.DYNAMODB.Table.6](#managedcontroldynamodbtable6)

**Scoping** (for [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup)): Amazon DynamoDB is natively integrated with AWS Backup, which supports a table as an independently protectable resource type. A table is the primary store for your persisted data, and AWS Backup captures its contents in recovery points that you can restore to a new table.

**Configure via**

| Operation | Parameters |
|---|---|
| `backup:CreateBackupPlan` | `$.BackupPlan.BackupPlanName`, `$.BackupPlan.Rules[*].Lifecycle.DeleteAfterDays`, `$.BackupPlan.Rules[*].RuleName`, `$.BackupPlan.Rules[*].ScheduleExpression`, `$.BackupPlan.Rules[*].TargetBackupVaultName` |
| `backup:CreateBackupSelection` | `$.BackupPlanId`, `$.BackupSelection.Conditions.StringEquals[*].ConditionKey`, `$.BackupSelection.Conditions.StringEquals[*].ConditionValue`, `$.BackupSelection.IamRoleArn`, `$.BackupSelection.ListOfTags[*].ConditionKey`, `$.BackupSelection.ListOfTags[*].ConditionType`, `$.BackupSelection.ListOfTags[*].ConditionValue`, `$.BackupSelection.NotResources[*]`, `$.BackupSelection.Resources[*]`, `$.BackupSelection.SelectionName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `backup:DescribeProtectedResource` | `$.ResourceArn` |
| `backup:GetBackupPlan` | `$.BackupPlanId` |
| `backup:GetBackupSelection` | `$.BackupPlanId`, `$.SelectionId` |
| `backup:ListBackupPlans` | *(any parameter)* |
| `backup:ListBackupSelections` | `$.BackupPlanId` |
| `backup:ListProtectedResources` | *(any parameter)* |
| `backup:ListRecoveryPointsByResource` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.5

An Amazon DynamoDB table should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.DYNAMODB.Table.5](#threatdynamodbtable5), [Threat.DYNAMODB.Table.7](#threatdynamodbtable7), [Threat.DYNAMODB.Table.12](#threatdynamodbtable12), [Threat.DYNAMODB.Table.15](#threatdynamodbtable15), [Threat.DYNAMODB.Table.16](#threatdynamodbtable16)

**Implemented by:** [Implementation.DYNAMODB.Table.13](#implementationdynamodbtable13), [Implementation.DYNAMODB.Table.14](#implementationdynamodbtable14), [Implementation.DYNAMODB.Table.15](#implementationdynamodbtable15), [Implementation.DYNAMODB.Table.16](#implementationdynamodbtable16), [ManagedControl.DYNAMODB.Table.7](#managedcontroldynamodbtable7), [ManagedControl.DYNAMODB.Table.8](#managedcontroldynamodbtable8)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): An Amazon DynamoDB table lets you select a customer managed key for its server-side encryption at rest through `$.SSESpecification.KMSMasterKeyId`. Owning the key gives you control over its policy and lifecycle, audit of its use, and the ability to make table data unavailable.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.SSESpecification.KMSMasterKeyId`, `$.TableName` |
| `dynamodb:ImportTable` | `$.TableCreationParameters.SSESpecification.KMSMasterKeyId`, `$.TableCreationParameters.TableName` |
| `dynamodb:RestoreTableFromBackup` | `$.SSESpecificationOverride.KMSMasterKeyId`, `$.TargetTableName` |
| `dynamodb:RestoreTableToPointInTime` | `$.SSESpecificationOverride.KMSMasterKeyId`, `$.TargetTableName` |
| `dynamodb:UpdateTable` | `$.ReplicaUpdates[*].Create.KMSMasterKeyId`, `$.ReplicaUpdates[*].Update.KMSMasterKeyId`, `$.SSESpecification.KMSMasterKeyId`, `$.TableName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:DescribeTable` | `$.TableName` |

### Mitigation.DYNAMODB.Table.6

An Amazon DynamoDB table's resource policy should limit permissions to a subset of IAM principals, rather than granting access to all IAM principals

**Goal:** [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)

**Addresses:** [Threat.DYNAMODB.Table.8](#threatdynamodbtable8)

**Implemented by:** [Implementation.DYNAMODB.Table.17](#implementationdynamodbtable17), [Implementation.DYNAMODB.Table.18](#implementationdynamodbtable18), [Implementation.DYNAMODB.Table.19](#implementationdynamodbtable19), [Implementation.DYNAMODB.Table.20](#implementationdynamodbtable20), [Implementation.DYNAMODB.Table.21](#implementationdynamodbtable21)

**Scoping** (for [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)): Your DynamoDB table supports a resource-based policy that must specify a principal. A wildcard principal in that policy extends its DynamoDB permissions to every IAM principal.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.ResourcePolicy`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.7

An Amazon DynamoDB table should be replicated to at least one other AWS Region

**Goal:** [Cross-Region Replication](../mitigation-goals.md#cross-region-replication)

**Addresses:** [Threat.DYNAMODB.Table.1](#threatdynamodbtable1), [Threat.DYNAMODB.Table.3](#threatdynamodbtable3), [Threat.DYNAMODB.Table.5](#threatdynamodbtable5)

**Implemented by:** [Implementation.DYNAMODB.Table.22](#implementationdynamodbtable22), [Implementation.DYNAMODB.Table.23](#implementationdynamodbtable23)

**Scoping** (for [Cross-Region Replication](../mitigation-goals.md#cross-region-replication)): A DynamoDB table is a Regional resource that persists your items and attributes. Global tables provide AWS-managed replication of that table data to replicas in other AWS Regions.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.GlobalTableSettingsReplicationMode`, `$.GlobalTableSourceArn`, `$.TableName` |
| `dynamodb:UpdateTable` | `$.GlobalTableSettingsReplicationMode`, `$.GlobalTableWitnessUpdates[*].Create.RegionName`, `$.MultiRegionConsistency`, `$.ReplicaUpdates[*].Create.RegionName`, `$.TableName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:DescribeTable` | `$.TableName` |
| `dynamodb:DescribeTableReplicaAutoScaling` | `$.TableName` |

### Mitigation.DYNAMODB.Table.8

An Amazon DynamoDB table should be accessed only by trusted identities or AWS services acting on behalf of trusted resources

**Goal:** [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)

**Addresses:** [Threat.DYNAMODB.Table.13](#threatdynamodbtable13)

**Implemented by:** [Implementation.DYNAMODB.Table.24](#implementationdynamodbtable24), [Implementation.DYNAMODB.Table.25](#implementationdynamodbtable25), [Implementation.DYNAMODB.Table.26](#implementationdynamodbtable26), [Implementation.DYNAMODB.Table.27](#implementationdynamodbtable27), [Implementation.DYNAMODB.Table.28](#implementationdynamodbtable28)

**Scoping** (for [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)): A DynamoDB table supports a resource-based policy that grants permissions to perform actions on the table. That policy can grant those permissions to principals outside your organization, making the identity perimeter applicable to this table.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.ResourcePolicy`, `$.TableName` |
| `dynamodb:DeleteResourcePolicy` | `$.ResourceArn` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.9

An Amazon DynamoDB table should be accessed by an AWS service principal, such as the Amazon Redshift service principal, only when the service principal is acting on behalf of a trusted resource, AWS account, or organization

**Goal:** [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies)

**Addresses:** [Threat.DYNAMODB.Table.17](#threatdynamodbtable17)

**Implemented by:** [Implementation.DYNAMODB.Table.29](#implementationdynamodbtable29), [Implementation.DYNAMODB.Table.30](#implementationdynamodbtable30), [Implementation.DYNAMODB.Table.31](#implementationdynamodbtable31), [Implementation.DYNAMODB.Table.32](#implementationdynamodbtable32), [Implementation.DYNAMODB.Table.33](#implementationdynamodbtable33)

**Scoping** (for [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies)): A resource-based policy on your DynamoDB table can grant an AWS service principal, such as `redshift.amazonaws.com`, access to the table. Such a grant may raise the cross-service confused deputy problem, where that principal may be able to act on behalf of resources outside your organization.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.ResourcePolicy`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.10

An Amazon DynamoDB table should be governed by a policy that requires all connections to be encrypted in transit

**Goal:** [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy)

**Addresses:** [Threat.DYNAMODB.Table.18](#threatdynamodbtable18)

**Implemented by:** [Implementation.DYNAMODB.Table.2](#implementationdynamodbtable2), [Implementation.DYNAMODB.Table.3](#implementationdynamodbtable3), [Implementation.DYNAMODB.Table.34](#implementationdynamodbtable34), [Implementation.DYNAMODB.Table.35](#implementationdynamodbtable35), [Implementation.DYNAMODB.Table.36](#implementationdynamodbtable36), [Implementation.DYNAMODB.Table.37](#implementationdynamodbtable37)

**Scoping** (for [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy)): Amazon DynamoDB supports resource-based policies on tables. Encrypted connections are a client-side default rather than a table-side requirement, so a policy can require encryption in transit.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.ResourcePolicy`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.11

An Amazon DynamoDB table should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.DYNAMODB.Table.6](#threatdynamodbtable6)

**Implemented by:** [Implementation.DYNAMODB.Table.38](#implementationdynamodbtable38), [Implementation.DYNAMODB.Table.39](#implementationdynamodbtable39), [Implementation.DYNAMODB.Table.40](#implementationdynamodbtable40), [Implementation.DYNAMODB.Table.41](#implementationdynamodbtable41)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): Your Amazon DynamoDB table accepts tags at creation through the tag key and value parameters `$.Tags[*].Key` and `$.Tags[*].Value`. You can also tag the table after creation by calling `TagResource` against its own ARN.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.TableName`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `dynamodb:TagResource` | `$.ResourceArn`, `$.Tags[*].Key`, `$.Tags[*].Value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:ListTagsOfResource` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.12

An Amazon DynamoDB table should be configured to disallow table policy modifications that would remove all policy-management access for the table

**Goal:** [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention)

**Addresses:** [Threat.DYNAMODB.Table.2](#threatdynamodbtable2)

**Implemented by:** [Implementation.DYNAMODB.Table.42](#implementationdynamodbtable42), [Implementation.DYNAMODB.Table.43](#implementationdynamodbtable43), [Implementation.DYNAMODB.Table.44](#implementationdynamodbtable44)

**Scoping** (for [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention)): Your DynamoDB table supports a resource-based policy, known as a table policy, that governs the `dynamodb:PutResourcePolicy` and `dynamodb:DeleteResourcePolicy` actions on the table itself. A policy write can therefore leave no principal able to change the table policy, and `PutResourcePolicy` accepts `$.x-amz-confirm-remove-self-resource-access` to waive that self-access-removal guard.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.ResourcePolicy`, `$.TableName` |
| `dynamodb:PutResourcePolicy` | `$.Policy`, `$.ResourceArn`, `$.x-amz-confirm-remove-self-resource-access` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.DYNAMODB.Table.13

An Amazon DynamoDB table should have CloudWatch Contributor Insights enabled to collect detailed CloudWatch metrics

**Goal:** [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)

**Addresses:** [Threat.DYNAMODB.Table.4](#threatdynamodbtable4), [Threat.DYNAMODB.Table.9](#threatdynamodbtable9)

**Implemented by:** [Implementation.DYNAMODB.Table.45](#implementationdynamodbtable45), [Implementation.DYNAMODB.Table.46](#implementationdynamodbtable46), [Implementation.DYNAMODB.Table.47](#implementationdynamodbtable47), [Implementation.DYNAMODB.Table.48](#implementationdynamodbtable48)

**Scoping** (for [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)): Your DynamoDB table supports CloudWatch Contributor Insights, an optional monitoring feature that publishes detailed metrics to CloudWatch. You can configure the feature for each table you manage.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:UpdateContributorInsights` | `$.ContributorInsightsAction`, `$.ContributorInsightsMode`, `$.IndexName`, `$.TableName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:DescribeContributorInsights` | `$.IndexName`, `$.TableName` |
| `dynamodb:ListContributorInsights` | `$.TableName` |

### Mitigation.DYNAMODB.Table.14

An Amazon DynamoDB table should have deletion protection enabled

**Goal:** [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection)

**Addresses:** [Threat.DYNAMODB.Table.1](#threatdynamodbtable1)

**Implemented by:** [Implementation.DYNAMODB.Table.49](#implementationdynamodbtable49), [Implementation.DYNAMODB.Table.50](#implementationdynamodbtable50), [Implementation.DYNAMODB.Table.51](#implementationdynamodbtable51), [Implementation.DYNAMODB.Table.52](#implementationdynamodbtable52), [Implementation.DYNAMODB.Table.53](#implementationdynamodbtable53), [Implementation.DYNAMODB.Table.54](#implementationdynamodbtable54), [ManagedControl.DYNAMODB.Table.9](#managedcontroldynamodbtable9)

**Scoping** (for [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection)): Deleting an Amazon DynamoDB table cannot be undone, and data is recoverable only where backups or point-in-time recovery exist. The table supports a built-in deletion protection capability that helps protect it from unintended deletion.

**Configure via**

| Operation | Parameters |
|---|---|
| `dynamodb:CreateTable` | `$.DeletionProtectionEnabled`, `$.TableName` |
| `dynamodb:UpdateTable` | `$.DeletionProtectionEnabled`, `$.TableName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dynamodb:DescribeTable` | `$.TableName` |

---

## Implementations

### Implementation.DYNAMODB.1

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.1](#mitigationdynamodb1)

**Scoping** (for [Mitigation.DYNAMODB.1](#mitigationdynamodb1)): This implementation uses your VPC endpoint policy to govern Amazon DynamoDB API calls made through the endpoint with the `aws:ResourceOrgID` condition key. As a result, calls to DynamoDB resources outside your organization are restricted, while exempting AWS services that access them on your behalf (`aws:PrincipalIsAWSService`).

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.DYNAMODB.2

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.2](#mitigationdynamodb2)

**Scoping** (for [Mitigation.DYNAMODB.2](#mitigationdynamodb2)): This VPC endpoint policy restricts Amazon DynamoDB API calls with the `aws:PrincipalOrgID` condition key, while exempting AWS services that act on your behalf (`aws:PrincipalIsAWSService`). Requests that do not satisfy that condition are denied, so principals outside your organization cannot call the DynamoDB API.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.DYNAMODB.GlobalTable.1

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb:ListGlobalTables`; check `$.GlobalTables[*].GlobalTableName`, `$.GlobalTables[*].ReplicationGroup[*].RegionName`
2. Call `dynamodb:DescribeGlobalTable` with `$.GlobalTableName`; check `$.GlobalTableDescription.ReplicationGroup[*].RegionName`, `$.GlobalTableDescription.ReplicationGroup[*].ReplicaStatus`

**Implements:** [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1)

**Scoping** (for [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1)): This periodic check flags a global table whose `$.GlobalTableDescription.ReplicationGroup[*].RegionName` names only one AWS Region. It also reads `$.GlobalTableDescription.ReplicationGroup[*].ReplicaStatus`, so a replica that has stopped replicating is reported instead of counting toward your multi-Region resiliency.

### Implementation.DYNAMODB.Stream.1

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Stream` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/stream/${StreamLabel}` | `aws:PrincipalArn`, `aws:PrincipalIsAWSService`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:ViaAWSService`, `aws:VpceOrgID` (alternatives: `aws:SourceVpc`, `aws:SourceVpce`, `aws:VpceAccount`, `aws:VpceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1)

**Scoping** (for [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1)): This implementation guidance recommends applying the `aws:SourceIp` and `aws:VpceOrgID` condition keys in your stream's resource-based policy to govern Amazon DynamoDB API calls against it. These conditions help restrict stream reads (`GetRecords`) to expected networks, exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`, `aws:ViaAWSService`), tagged principals (`aws:PrincipalTag/dp:exclude:network`) and the administering principal (`aws:PrincipalArn`).

**Reference:** Confirm that resource-based policies grant access only to expected networks. As a defense-in-depth measure, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalIsAWSService` to exempt AWS services that access your resources with their service principals. Use `aws:PrincipalTag/tag-key` to exempt principals that must legitimately operate from outside your expected networks. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)

### Implementation.DYNAMODB.Stream.2

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1), [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5), [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7)

**Scoping** (for [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7)): Validates `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument` at create time, before the policy reaches your replica's stream. Rejecting a document that leaves no principal able to call `dynamodb:PutResourcePolicy` helps you keep the stream policy manageable and avoid an orphaned stream.

**Scoping** (for [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)): This implementation validates that the stream resource-based policy at `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument` uses the `aws:SecureTransport` condition key to deny requests made over unencrypted connections. Stack operations that declare a replica stream policy without that condition are denied.

**Scoping** (for [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1)): This hook validates `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument` before each replica stream is provisioned. Rejecting stream policies that do not satisfy conditions such as `aws:SourceIp`, `aws:SourceVpc`, `aws:SourceVpce`, or `aws:VpceOrgID` helps keep stream records reachable only from expected networks.

### Implementation.DYNAMODB.Stream.3

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.StreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1), [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5), [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7)

**Scoping** (for [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7)): This implementation validates the policy document at `$.StreamSpecification.ResourcePolicy.PolicyDocument` before it is attached to your table's stream. Rejecting a document that removes all `dynamodb:PutResourcePolicy` and `dynamodb:DeleteResourcePolicy` access helps keep the stream's policy manageable.

**Scoping** (for [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)): At create and update time, this implementation validates `$.StreamSpecification.ResourcePolicy.PolicyDocument`, the resource-based policy on your table's stream, for an `aws:SecureTransport` condition. Stack operations whose stream policy omits that condition are denied, helping restrict stream access to encrypted connections.

**Scoping** (for [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1)): This implementation validates `$.StreamSpecification.ResourcePolicy.PolicyDocument` at create and update time, confirming the stream policy carries conditions such as `aws:SourceIp`, `aws:SourceVpc`, `aws:SourceVpce`, or `aws:VpceOrgID`. A template lacking those conditions fails validation, helping keep your stream records reachable only from expected networks.

### Implementation.DYNAMODB.Stream.4

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:ViaAWSService`, `aws:VpceOrgID` (alternatives: `aws:SourceVpc`, `aws:SourceVpce`, `aws:VpceAccount`, `aws:VpceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1)

**Scoping** (for [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1)): This resource control policy (RCP) restricts Amazon DynamoDB stream access to expected networks with the `aws:SourceIp` and `aws:VpceOrgID` condition keys. Requests that do not satisfy those conditions are denied, except for AWS services acting on your behalf under `aws:PrincipalIsAWSService` and `aws:ViaAWSService`.

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalIsAWSService` to exempt AWS services that access your resources with their service principals. Use `aws:PrincipalTag/tag-key` to exempt principals that must legitimately operate from outside your expected networks. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [RCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/network_perimeter_vpceorgid_rcp.json)

### Implementation.DYNAMODB.Stream.5

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Stream.2](#mitigationdynamodbstream2)

**Scoping** (for [Mitigation.DYNAMODB.Stream.2](#mitigationdynamodbstream2)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern DynamoDB stream actions such as `dynamodb:GetRecords`. As a result, your identities are restricted to reading stream records only from trusted resources in your organization.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.DYNAMODB.Stream.6

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Stream` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:DescribeStream`, `dynamodb:GetRecords`, `dynamodb:GetShardIterator` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/stream/${StreamLabel}` | `aws:PrincipalAccount` |

**Implements:** [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)

**Scoping** (for [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)): This implementation governs `dynamodb:DescribeStream`, `dynamodb:GetShardIterator`, and `dynamodb:GetRecords` in the stream's resource-based policy (`$.Policy`) with the `aws:PrincipalAccount` condition key. As a result, principals outside the accounts you specify are restricted from reading the item-level change records your stream retains.

**Reference:** Confirm that resource-based policies grant access only to authorized identities. As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.DYNAMODB.Stream.7

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)

**Scoping** (for [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)): This hook checks `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument` for public access before CloudFormation applies a replica stream's policy. Stack operations carrying a policy that could grant public access are denied, so a wildcard principal is caught before it reaches that stream.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.DYNAMODB.Stream.8

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.StreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)

**Scoping** (for [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)): This hook checks the resource-based policy on your stream at `$.StreamSpecification.ResourcePolicy.PolicyDocument` for statements that can grant public access. Stack operations that fail this check are denied, helping you limit stream access to principals you name explicitly.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.DYNAMODB.Stream.9

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb-streams:ListStreams`; check `$.Streams[*].StreamArn`
2. Call `dynamodb:GetResourcePolicy` with `$.ResourceArn`; check `$.Policy`
3. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::DynamoDB::Stream`; check `$.result` = `PASS`

**Implements:** [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)

**Scoping** (for [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)): This implementation reads each stream's resource-based policy at `$.Policy` and calls IAM Access Analyzer `CheckNoPublicAccess` to evaluate it. Flagging streams whose public-access verdict at `$.result` is not `PASS` helps you find policies that extend access to all IAM principals.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.DYNAMODB.Stream.10

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `arn:aws:dynamodb:*:*:table/*/stream/*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)

**Scoping** (for [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3)): This implementation governs Amazon DynamoDB API calls against your streams with the `aws:PrincipalOrgID` condition key, while exempting AWS services that access them on your behalf (`aws:PrincipalIsAWSService`). As a result, principals outside your organization are restricted from accessing your streams, even when a stream's resource-based policy grants access to a wildcard principal.

**Reference:** To restrict access to trusted identities at scale, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)
- [RCP sample: restrict resource access to principals in the same organizational unit](https://github.com/aws-samples/resource-control-policy-examples/blob/main/Establish-intra-organization-boundaries/Deny-access-to-resources-in-an-organizational-unit%2Cexcept-for-principals-from-the-same-or-specified-organizational-unit.json)

### Implementation.DYNAMODB.Stream.11

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Stream` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/stream/${StreamLabel}` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `dynamodb:*` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/stream/${StreamLabel}` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)

**Scoping** (for [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)): This resource-based policy governs all Amazon DynamoDB API calls against your stream with the `aws:PrincipalOrgID` condition key, while exempting AWS services that access the stream on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. As a result, reads of the stream's item-level change records are restricted to principals in your organization and services acting on your behalf.

**Reference:** Confirm that resource-based policies grant access only to trusted identities and to services acting on your behalf.

As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`.

To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys.

Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.DYNAMODB.Stream.12

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)

**Scoping** (for [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)): This implementation validates `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument` before each replica stream is provisioned. Deployments whose stream policy would grant public access, grant untrusted principals stream actions such as `dynamodb:GetRecords`, or grant a service principal access without `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` do not succeed.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DYNAMODB.Stream.13

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.StreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)

**Scoping** (for [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)): This implementation validates `$.StreamSpecification.ResourcePolicy.PolicyDocument`, the resource-based policy on your table's stream, at create and update time. The hook keeps policies that allow public access or omit service principal source conditions from being provisioned.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DYNAMODB.Stream.14

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb-streams:ListStreams`; check `$.Streams[*].StreamArn`
2. Call `dynamodb:GetResourcePolicy` with `$.ResourceArn`; check `$.Policy`
3. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::DynamoDB::Stream`; check `$.result` = `PASS`
4. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result` = `PASS`
5. Call `accessanalyzer:CheckAccessNotGranted` with `$.policyDocument`, `$.policyType` = `RESOURCE_POLICY`, `$.access[*].actions[*]`; check `$.result` = `PASS`
6. Call `accessanalyzer:ValidatePolicy` with `$.policyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.findings[*].issueCode` = `RESTRICT_ACCESS_TO_SERVICE_PRINCIPAL`
7. Call `accessanalyzer:ListFindingsV2` with `$.analyzerArn`, `$.filter`; check `$.findings`

**Implements:** [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)

**Scoping** (for [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)): This implementation reads your stream's resource-based policy from `$.Policy` and evaluates it for public access and for specified untrusted access. Flagging such grants helps address the mitigation by surfacing stream access reachable by identities outside your zone of trust.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.

External access findings, generated by an external access analyzer, use automated reasoning. They identify resource-based policies that allow access from outside your zone of trust.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DYNAMODB.Stream.15

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `arn:aws:dynamodb:*:*:table/*/stream/*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `dynamodb:*` | `arn:aws:dynamodb:*:*:table/*/stream/*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)

**Scoping** (for [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4)): This implementation governs Amazon DynamoDB API calls against your streams with `aws:PrincipalOrgID` and `aws:SourceOrgID` condition keys, exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, principals outside your organization are restricted from cross-account `dynamodb:GetRecords` reads that a resource-based policy on the stream could otherwise permit.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.DYNAMODB.Stream.16

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Stream` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/stream/${StreamLabel}` | `aws:SecureTransport` |

**Implements:** [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)

**Scoping** (for [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)): This implementation restricts Amazon DynamoDB API calls against your stream with the `aws:SecureTransport` condition key. Requests that are not sent over an encrypted connection are denied, so stream records can be read only over a TLS-protected connection.

### Implementation.DYNAMODB.Stream.17

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb-streams:ListStreams`; check `$.Streams[*].StreamArn`
2. Call `dynamodb:GetResourcePolicy` with `$.ResourceArn`; check `$.Policy`

**Implements:** [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)

**Scoping** (for [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)): This implementation reads each stream's resource-based policy at `$.Policy` and flags streams whose policy does not condition stream access on `aws:SecureTransport`. Surfacing that gap helps you review which streams lack a policy-level requirement for encrypted connections.

### Implementation.DYNAMODB.Stream.18

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `*` | `aws:SecureTransport` |

**Implements:** [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)

**Scoping** (for [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)): This resource control policy (RCP) restricts Amazon DynamoDB API calls against your stream by using the `aws:SecureTransport` condition key. Requests that are not sent over TLS are denied, so access to your stream records requires encrypted transport.

**Reference:** To require encryption in transit, use the `aws:SecureTransport` condition key so that requests made over unencrypted connections are denied.
- [RCP sample: require HTTPS connections to your resources](https://github.com/aws-samples/resource-control-policy-examples/blob/main/Restrict-resource-access-patterns/Restrict-access-to-only-HTTPS-connections-to-your-resources.json)

### Implementation.DYNAMODB.Stream.19

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:DescribeStream`, `dynamodb:GetRecords`, `dynamodb:GetShardIterator`, `dynamodb:ListStreams` | `*` | `aws:SecureTransport` |

**Implements:** [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)

**Scoping** (for [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5)): This implementation restricts the stream's actions, such as `dynamodb:GetRecords`, with `aws:SecureTransport`. Stream requests from principals in your organization that are not made over TLS are denied.

### Implementation.DYNAMODB.Stream.20

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ReplicaStreamSpecification.Tags[*].Key`, `$.Replicas[*].ReplicaStreamSpecification.Tags[*].Value`

**Implements:** [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)

**Scoping** (for [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)): This hook validates that `$.Replicas[*].ReplicaStreamSpecification.Tags[*].Key` and `$.Replicas[*].ReplicaStreamSpecification.Tags[*].Value` declare owner and purpose tags on each replica stream at create time. Because stream tags are independent of table tags, this helps you attribute ownership of change-data-capture data that replica table tags do not cover.

### Implementation.DYNAMODB.Stream.21

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.StreamSpecification.Tags[*].Key`, `$.StreamSpecification.Tags[*].Value`

**Implements:** [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)

**Scoping** (for [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)): This implementation checks the tag keys and values at `$.StreamSpecification.Tags[*].Key` and `$.StreamSpecification.Tags[*].Value` when a table with a stream is created or updated. Requiring owner and purpose tags on those properties helps address the mitigation by making each stream traceable to a responsible team and business function.

### Implementation.DYNAMODB.Stream.22

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb-streams:ListStreams`; check `$.Streams[*].StreamArn`, `$.Streams[*].TableName`
2. Call `dynamodb:ListTagsOfResource` with `$.ResourceArn`; check `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)

**Scoping** (for [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)): This implementation reads the tag keys and values recorded on your Amazon DynamoDB stream (`$.Tags[*].Key` and `$.Tags[*].Value`). Flagging streams that lack your required owner and purpose tags helps you identify streams with no recorded owner or business function.

### Implementation.DYNAMODB.Stream.23

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:TagResource` | `arn:aws:dynamodb:*:*:table/*/stream/*` | `aws:RequestTag/owner` |
| `dynamodb:TagResource` | `arn:aws:dynamodb:*:*:table/*/stream/*` | `aws:RequestTag/purpose` |
| `dynamodb:TagResource` | `arn:aws:dynamodb:*:*:table/*/stream/*` | `aws:TagKeys` |
| `dynamodb:UntagResource` | `arn:aws:dynamodb:*:*:table/*/stream/*` | `aws:TagKeys` |

**Implements:** [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)

**Scoping** (for [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6)): This implementation restricts `dynamodb:TagResource` and `dynamodb:UntagResource` on your stream ARNs with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys. Requests that omit or remove the owner and purpose tag keys are denied, keeping ownership and business function traceable for each stream.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.DYNAMODB.Stream.24

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `dynamodb:DeleteResourcePolicy` — `$.ResourceArn`
- `dynamodb:PutResourcePolicy` — `$.Policy`, `$.ResourceArn`, `$.x-amz-confirm-remove-self-resource-access` = `true`

**Implements:** [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7)

**Scoping** (for [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7)): This implementation records `PutResourcePolicy` calls on your stream, capturing the policy document in `$.Policy` and the remove-self-resource-access confirmation in `$.x-amz-confirm-remove-self-resource-access`. Recording those calls helps you detect a policy change that removed all policy-management access, once the change has taken effect.

### Implementation.DYNAMODB.Table.1

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Table` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/index/${IndexName}` | `aws:PrincipalArn`, `aws:PrincipalIsAWSService`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:ViaAWSService`, `aws:VpceOrgID` (alternatives: `aws:SourceVpc`, `aws:SourceVpce`, `aws:VpceAccount`, `aws:VpceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1)

**Scoping** (for [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1)): This implementation restricts Amazon DynamoDB API calls to your table and its indexes with the `aws:SourceIp` and `aws:VpceOrgID` condition keys. Requests for item reads, writes, exports, and reconfiguration from unexpected networks are denied, while exempting AWS services that access them on your behalf (`aws:PrincipalIsAWSService`).

**Reference:** Confirm that resource-based policies grant access only to expected networks. As a defense-in-depth measure, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalIsAWSService` to exempt AWS services that access your resources with their service principals. Use `aws:PrincipalTag/tag-key` to exempt principals that must legitimately operate from outside your expected networks. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)

### Implementation.DYNAMODB.Table.2

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)

**Scoping** (for [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1)): This implementation validates that each replica policy document at `$.Replicas[*].ResourcePolicy.PolicyDocument` declares conditions such as `aws:SourceIp`, `aws:SourceVpc`, `aws:SourceVpce`, or `aws:VpceOrgID`. A template that omits these network conditions fails the check before the replica is provisioned.

**Scoping** (for [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)): This implementation validates at create time that each `$.Replicas[*].ResourcePolicy.PolicyDocument` on your global table conditions access on `aws:SecureTransport`. The check verifies the condition is present in the policy; it does not itself establish that requests made over unencrypted connections are denied.

### Implementation.DYNAMODB.Table.3

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)

**Scoping** (for [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1)): This implementation validates the `$.ResourcePolicy.PolicyDocument` your table declares before the table is provisioned. Requiring conditions such as `aws:SourceIp`, `aws:SourceVpc`, `aws:SourceVpce`, or `aws:VpceOrgID` on the table's DynamoDB actions helps restrict access to your expected networks.

**Scoping** (for [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)): This implementation requires the table resource-based policy in `$.ResourcePolicy.PolicyDocument` to deny requests where `aws:SecureTransport` is false. A template whose policy omits that statement is rejected, so the table is not provisioned without resource-level rejection of unencrypted requests.

### Implementation.DYNAMODB.Table.4

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:ViaAWSService`, `aws:VpceOrgID` (alternatives: `aws:SourceVpc`, `aws:SourceVpce`, `aws:VpceAccount`, `aws:VpceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1)

**Scoping** (for [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1)): This resource control policy (RCP) governs Amazon DynamoDB API calls against your tables with the `aws:SourceIp` and `aws:VpceOrgID` condition keys, while exempting AWS services that access them on your behalf (`aws:ViaAWSService`, `aws:PrincipalIsAWSService`) and principals you tag with `aws:PrincipalTag/dp:exclude:network`. As a result, your tables can be accessed only from expected networks, establishing a network perimeter.

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalIsAWSService` to exempt AWS services that access your resources with their service principals. Use `aws:PrincipalTag/tag-key` to exempt principals that must legitimately operate from outside your expected networks. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [RCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/network_perimeter_vpceorgid_rcp.json)

### Implementation.DYNAMODB.Table.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].PointInTimeRecoverySpecification.PointInTimeRecoveryEnabled`, `$.Replicas[*].PointInTimeRecoverySpecification.RecoveryPeriodInDays`

**Implements:** [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)

**Scoping** (for [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)): This implementation validates `$.Replicas[*].PointInTimeRecoverySpecification.PointInTimeRecoveryEnabled` at create time, so a global table replica cannot deploy with point-in-time recovery left off. Point-in-time recovery is not retroactive, so requiring it at deployment helps you restore within the window set by `$.Replicas[*].PointInTimeRecoverySpecification.RecoveryPeriodInDays`.

### Implementation.DYNAMODB.Table.6

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.PointInTimeRecoverySpecification.PointInTimeRecoveryEnabled`, `$.PointInTimeRecoverySpecification.RecoveryPeriodInDays`

**Implements:** [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)

**Scoping** (for [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)): This implementation validates the point-in-time recovery setting `$.PointInTimeRecoverySpecification.PointInTimeRecoveryEnabled` on table create and update operations, so a table is not provisioned without recovery points. Point-in-time recovery is not retroactive, so this check helps recovery coverage begin with your table.

### Implementation.DYNAMODB.Table.7

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `dynamodb:UpdateContinuousBackups` — `$.PointInTimeRecoverySpecification.PointInTimeRecoveryEnabled` = `false`, `$.PointInTimeRecoverySpecification.RecoveryPeriodInDays`, `$.TableName`

**Implements:** [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)

**Scoping** (for [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)): This implementation records `UpdateContinuousBackups` calls that set `$.PointInTimeRecoverySpecification.PointInTimeRecoveryEnabled` to false, or that shorten the recovery window through `$.PointInTimeRecoverySpecification.RecoveryPeriodInDays`. Surfacing the calling identity and the time of the change helps address the mitigation, because turning point-in-time recovery off discards recovery points you cannot recreate.

### Implementation.DYNAMODB.Table.8

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DynamoDB::Table`: `$.supplementaryConfiguration.ContinuousBackupsDescription.pointInTimeRecoveryDescription.pointInTimeRecoveryStatus`, `$.supplementaryConfiguration.ContinuousBackupsDescription.continuousBackupsStatus`, `$.configuration.tableStatus`

**Implements:** [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)

**Scoping** (for [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)): This implementation flags your Amazon DynamoDB tables whose point-in-time recovery status, recorded at `$.supplementaryConfiguration.ContinuousBackupsDescription.pointInTimeRecoveryDescription.pointInTimeRecoveryStatus`, is not `ENABLED`. That check addresses the mitigation by identifying tables where DynamoDB is not maintaining automatic recovery points for restore.

### Implementation.DYNAMODB.Table.9

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3)

**Scoping** (for [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern Amazon DynamoDB API calls on table resources. As a result, principals in your organization are restricted from accessing tables owned by accounts outside your organization.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.DYNAMODB.Table.10

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `backup:DeleteBackupPlan`
- `backup:DeleteBackupSelection`
- `backup:DeleteRecoveryPoint`
- `backup:DisassociateRecoveryPoint`
- `backup:StopBackupJob`

**Implements:** [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)

**Scoping** (for [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)): This implementation records `DeleteBackupPlan`, `DeleteBackupSelection`, `DeleteRecoveryPoint`, `DisassociateRecoveryPoint`, and `StopBackupJob`. Recording these calls helps address the mitigation by making removal of your table's backup plan protection detectable.

### Implementation.DYNAMODB.Table.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `backup:ListBackupPlans`; check `$.BackupPlansList[*].BackupPlanId`
2. Call `backup:ListBackupSelections` with `$.BackupPlanId`; check `$.BackupSelectionsList[*].SelectionId`
3. Call `backup:GetBackupSelection` with `$.BackupPlanId`, `$.SelectionId`; check `$.BackupSelection.Resources[*]`, `$.BackupSelection.NotResources[*]`, `$.BackupSelection.ListOfTags[*].ConditionKey`, `$.BackupSelection.ListOfTags[*].ConditionValue`, `$.BackupSelection.Conditions.StringEquals[*].ConditionKey`, `$.BackupSelection.Conditions.StringEquals[*].ConditionValue`, `$.BackupSelection.Conditions.StringLike[*].ConditionKey`, `$.BackupSelection.Conditions.StringLike[*].ConditionValue`, `$.BackupSelection.Conditions.StringNotEquals[*].ConditionKey`, `$.BackupSelection.Conditions.StringNotEquals[*].ConditionValue`, `$.BackupSelection.Conditions.StringNotLike[*].ConditionKey`, `$.BackupSelection.Conditions.StringNotLike[*].ConditionValue`

**Implements:** [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)

**Scoping** (for [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)): This periodic check compares your table against each backup selection's `$.BackupSelection.Resources[*]` and tag conditions. Flagging tables that no selection covers helps you detect data left without a centrally governed recovery point.

### Implementation.DYNAMODB.Table.12

*Declarative policy — Preventive*

Governs the `BACKUP_POLICY` declarative policy attributes below.

- `$.plans.*.selections.resources.*.resource_types`

**Implements:** [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)

**Scoping** (for [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)): This implementation declares `$.plans.*.selections.resources.*.resource_types`, the list of resource types a backup plan protects. Requiring the DynamoDB table type in that list assigns your tables to the plan for scheduled recovery points.

### Implementation.DYNAMODB.Table.13

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.SSESpecification.SSEEnabled`, `$.SSESpecification.SSEType`, `$.Replicas[*].SSESpecification.KMSMasterKeyId`

**Implements:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

**Scoping** (for [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)): Before your DynamoDB global table is created, this hook checks each replica's KMS key identifier (`$.Replicas[*].SSESpecification.KMSMasterKeyId`) by calling `DescribeKey`. Rejecting replicas without a customer managed key helps you keep control of key policy and lifecycle.

### Implementation.DYNAMODB.Table.14

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.SSESpecification.KMSMasterKeyId`, `$.SSESpecification.SSEEnabled`, `$.SSESpecification.SSEType`

**Implements:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

**Scoping** (for [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)): This hook validates `$.SSESpecification.KMSMasterKeyId`, the setting that names the KMS key encrypting your table at rest. A table that omits a customer managed key is rejected, which helps address the mitigation by keeping key policy and lifecycle in your control.

### Implementation.DYNAMODB.Table.15

*CloudTrail data event — Detective*

Monitors the API calls below in CloudTrail.

- `dynamodb:BatchExecuteStatement` — `$.Statements[*].Statement`
- `dynamodb:BatchGetItem`
- `dynamodb:ExecuteStatement` — `$.Statement`
- `dynamodb:GetItem` — `$.TableName`
- `dynamodb:Query` — `$.TableName`
- `dynamodb:Scan` — `$.TableName`
- `dynamodb:SearchVectors` — `$.TableName`
- `dynamodb:TransactGetItems` — `$.TransactItems[*].Get.TableName`

**Implements:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

**Scoping** (for [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)): This implementation records `GetItem`, `Query`, `Scan`, `BatchGetItem`, `TransactGetItems`, `ExecuteStatement`, `BatchExecuteStatement` and `SearchVectors` calls against your table, along with the identity that made each call. Reviewing those reads helps you detect and investigate an authorized-but-unexpected read of items protected by your customer managed key.

**Reference:** Enforcing a customer managed key governs which key protects the data. It does not record which permitted principals read that data. The data events this guide monitors record those reads. CloudTrail does not log data events by default, and they are high volume. To scope log volume and cost, add advanced event selectors to the trail or event data store. Filter `resources.ARN` to the governed resources, using `Equals` for exact ARNs or `StartsWith` for a naming prefix. Filter `eventName` to the operations this guide monitors. A `readOnly` filter selects by CloudTrail's read or write classification, not by this guide's operation set, so do not rely on it alone. A trail or event data store accepts at most 500 values across all advanced event selector conditions.
- [Filtering data events by using advanced event selectors](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/filtering-data-events.html)
- [Logging data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)
- [Quotas in AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/WhatIsCloudTrail-Limits.html)

### Implementation.DYNAMODB.Table.16

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DynamoDB::Table`: `$.configuration.ssedescription.kmsmasterKeyArn`, `$.configuration.ssedescription.ssetype`, `$.configuration.ssedescription.status`, `$.configuration.replicas[*].kmsmasterKeyId`

**Implements:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

**Scoping** (for [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)): This rule reads `$.configuration.ssedescription.kmsmasterKeyArn` on the table's configuration item, then resolves `$.KeyMetadata.KeyManager` for that key. Flagging tables not protected by a customer managed key helps you review where key lifecycle control and audit visibility are absent.

### Implementation.DYNAMODB.Table.17

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Table` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:BatchGetItem`, `dynamodb:BatchWriteItem`, `dynamodb:ConditionCheckItem`, `dynamodb:DeleteItem`, `dynamodb:DescribeTable`, `dynamodb:ExportTableToPointInTime`, `dynamodb:GetItem`, `dynamodb:ListTagsOfResource`, `dynamodb:PartiQLDelete`, `dynamodb:PartiQLInsert`, `dynamodb:PartiQLSelect`, `dynamodb:PartiQLUpdate`, `dynamodb:PutItem`, `dynamodb:Query`, `dynamodb:Scan`, `dynamodb:UpdateItem` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/index/${IndexName}` | `aws:PrincipalAccount` |

**Implements:** [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)

**Scoping** (for [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)): This implementation requires your table's resource-based policy to name the specific IAM principals it grants access to, rather than all IAM principals. As a result, principals outside that set are restricted from accessing your table through that policy.

**Reference:** Confirm that resource-based policies grant access only to authorized identities. As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.DYNAMODB.Table.18

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)

**Scoping** (for [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)): This implementation validates the resource-based policy you declare for each replica of your global table (`$.Replicas[*].ResourcePolicy.PolicyDocument`). Rejecting a policy that can grant public access helps you catch an unrestricted wildcard principal before the policy is attached.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.DYNAMODB.Table.19

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)

**Scoping** (for [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)): This implementation validates the resource-based policy your table declares in `$.ResourcePolicy.PolicyDocument` before the table is created. Table creation is denied when that policy grants access to all IAM principals, helping limit access to the principals you name.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.DYNAMODB.Table.20

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb:ListTables`; check `$.TableNames[*]`
2. Call `dynamodb:DescribeTable` with `$.TableName`; check `$.Table.TableArn`
3. Call `dynamodb:GetResourcePolicy` with `$.ResourceArn`; check `$.Policy`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::DynamoDB::Table`; check `$.result` = `PASS`

**Implements:** [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)

**Scoping** (for [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)): This AWS Config rule evaluates the resource-based policy your table records at `$.Policy` by calling `CheckNoPublicAccess`. Flagging tables whose public-access verdict at `$.result` is not `PASS` helps you identify resource-based policies that can grant public access.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.DYNAMODB.Table.21

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)

**Scoping** (for [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6)): This implementation governs Amazon DynamoDB API calls against your resources with the `aws:PrincipalOrgID` condition key, while exempting AWS services that access them on your behalf (`aws:PrincipalIsAWSService`). As a result, principals outside your organization are restricted from accessing your DynamoDB resources, even when a table's resource-based policy specifies a wildcard principal.

**Reference:** To restrict access to trusted identities at scale, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)
- [RCP sample: restrict resource access to principals in the same organizational unit](https://github.com/aws-samples/resource-control-policy-examples/blob/main/Establish-intra-organization-boundaries/Deny-access-to-resources-in-an-organizational-unit%2Cexcept-for-principals-from-the-same-or-specified-organizational-unit.json)

### Implementation.DYNAMODB.Table.22

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].Region`, `$.MultiRegionConsistency`, `$.GlobalTableWitnesses[*].Region`, `$.Replicas[*].GlobalTableSettingsReplicationMode`, `$.GlobalTableSourceArn`, `$.TableName`

**Implements:** [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7)

**Scoping** (for [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7)): At create time, this hook rejects a global table whose replica Region list, `$.Replicas[*].Region`, names only the stack Region. That additional Region addresses the mitigation, since DynamoDB then replicates your table data to another Region automatically.

### Implementation.DYNAMODB.Table.23

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DynamoDB::Table`: `$.configuration.replicas[*].regionName`, `$.configuration.replicas[*].replicaStatus`, `$.configuration.globalTableVersion`, `$.awsRegion`

**Implements:** [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7)

**Scoping** (for [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7)): This implementation flags a table whose `$.configuration.replicas[*].regionName` list names no Region other than `$.awsRegion`. Surfacing those tables helps you add a global table replica in a second AWS Region before a Regional event affects availability.

### Implementation.DYNAMODB.Table.24

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Table` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/index/${IndexName}` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `dynamodb:*` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/index/${IndexName}` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)

**Scoping** (for [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)): This resource-based policy governs Amazon DynamoDB API calls against your table with the `aws:PrincipalOrgID` condition key, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. As a result, table access is restricted to principals in your organization and to AWS services acting on behalf of your resources.

**Reference:** Confirm that resource-based policies grant access only to trusted identities and to services acting on your behalf.

As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`.

To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys.

Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.DYNAMODB.Table.25

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)

**Scoping** (for [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)): This implementation validates the resource-based policy in `$.Replicas[*].ResourcePolicy.PolicyDocument` before a global table replica is provisioned. Create and update operations are denied when that policy could grant public access, or access beyond the principals you designate as trusted.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DYNAMODB.Table.26

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.ResourcePolicy.PolicyDocument`, `$.StreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)

**Scoping** (for [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)): This implementation validates the table's resource-based policy at `$.ResourcePolicy.PolicyDocument` and the stream's resource-based policy at `$.StreamSpecification.ResourcePolicy.PolicyDocument`. Provisioning fails when either policy allows public access, grants permissions you specify as untrusted, or grants a service principal access without a source condition.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DYNAMODB.Table.27

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb:ListTables`; check `$.TableNames[*]`
2. Call `dynamodb:DescribeTable` with `$.TableName`; check `$.Table.TableArn`
3. Call `dynamodb:GetResourcePolicy` with `$.ResourceArn`; check `$.Policy`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::DynamoDB::Table`; check `$.result`
5. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
6. Call `accessanalyzer:CheckAccessNotGranted` with `$.policyDocument`, `$.policyType` = `RESOURCE_POLICY`, `$.access[*].actions[*]`; check `$.result`
7. Call `accessanalyzer:ValidatePolicy` with `$.policyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.findings[*].issueCode` = `RESTRICT_ACCESS_TO_SERVICE_PRINCIPAL`
8. Call `accessanalyzer:ListFindingsV2` with `$.analyzerArn`, `$.filter{key}.eq[*]`; check `$.findings[*].findingType` = `ExternalAccess`, `$.findings[*].status` = `ACTIVE`

**Implements:** [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)

**Scoping** (for [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)): This rule reads each table's resource-based policy as `$.Policy` and evaluates it with IAM Access Analyzer checks. Flagging policies that allow public access or access by identities outside your organization helps you detect table access beyond trusted identities.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.

External access findings, generated by an external access analyzer, use automated reasoning. They identify resource-based policies that allow access from outside your zone of trust.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DYNAMODB.Table.28

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `dynamodb:*` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)

**Scoping** (for [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8)): This resource control policy (RCP) governs Amazon DynamoDB API calls against your resources with the `aws:PrincipalOrgID` condition key. As a result, identities outside your organization are restricted from accessing your resources. AWS services that access your resources on your behalf are exempt (`aws:PrincipalIsAWSService`), with those service requests scoped by the `aws:SourceOrgID` and `aws:SourceAccount` condition keys.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.DYNAMODB.Table.29

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Table` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:DescribeExport`, `dynamodb:DescribeTable`, `dynamodb:ExportTableToPointInTime` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/export/${ExportName}` | `aws:SourceAccount`, `aws:SourceArn` |

**Implements:** [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)

**Scoping** (for [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)): This resource-based policy governs `dynamodb:ExportTableToPointInTime`, `dynamodb:DescribeTable`, and `dynamodb:DescribeExport` on your table with the `aws:SourceArn` and `aws:SourceAccount` condition keys. As a result, requests made on behalf of a resource or account you have not designated are restricted from exporting or inspecting your table data.

**Reference:** To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.DYNAMODB.Table.30

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)

**Scoping** (for [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)): This implementation validates your replica's resource-based policy in `$.Replicas[*].ResourcePolicy.PolicyDocument` at create time, failing the stack operation on a `RESTRICT_ACCESS_TO_SERVICE_PRINCIPAL` warning. The hook therefore requires `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths`, restricting an AWS service to requests made on behalf of resources in your organization.

**Reference:** This implementation recommends using IAM Access Analyzer basic policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.DYNAMODB.Table.31

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)

**Scoping** (for [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)): At create time, this implementation validates `$.ResourcePolicy.PolicyDocument`, the table's resource-based policy, for the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, and `aws:SourceOrgPaths` condition keys. Rejecting a grant that omits them helps prevent an AWS service from being called to access your table on behalf of resources outside your organization.

**Reference:** This implementation recommends using IAM Access Analyzer basic policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.DYNAMODB.Table.32

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb:ListTables`; check `$.TableNames[*]`
2. Call `dynamodb:DescribeTable` with `$.TableName`; check `$.Table.TableArn`
3. Call `dynamodb:GetResourcePolicy` with `$.ResourceArn`; check `$.Policy`
4. Call `accessanalyzer:ValidatePolicy` with `$.policyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.findings[*].issueCode` = `RESTRICT_ACCESS_TO_SERVICE_PRINCIPAL`

**Implements:** [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)

**Scoping** (for [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)): This rule validates a table's resource-based policy at `$.Policy` and flags a `$.findings[*].issueCode` of `RESTRICT_ACCESS_TO_SERVICE_PRINCIPAL` when `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, and `aws:SourceOrgPaths` are absent. That finding helps you detect access that is not restricted to requests made on behalf of resources in your organization.

**Reference:** This implementation recommends using IAM Access Analyzer basic policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.DYNAMODB.Table.33

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)

**Scoping** (for [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9)): This implementation restricts Amazon DynamoDB API calls AWS services make against your tables on your behalf (`aws:PrincipalIsAWSService`) with the `aws:SourceOrgID` and `aws:SourceAccount` condition keys. Service requests on behalf of a resource or account outside your organization are denied, regardless of what your table's resource-based policy grants.

**Reference:** To restrict service access to your resources at scale, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. These keys are included in the request context only for requests by AWS service principals, so apply them when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: enforce controls on AWS services that use service principals](https://github.com/aws-samples/resource-control-policy-examples/blob/main/Restrict-resource-access-patterns/Enforce-controls-on-AWS-services-that-use-service-principals-to-access-your-resources.json)

### Implementation.DYNAMODB.Table.34

*Resource policy — Preventive*

Attaches to `AWS::DynamoDB::Table` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/index/${IndexName}` | `aws:SecureTransport` |

**Implements:** [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)

**Scoping** (for [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)): This implementation restricts Amazon DynamoDB API calls against your table and its indexes with the `aws:SecureTransport` condition key. Requests that do not arrive over TLS are denied, so the service does not process them or return data over an unencrypted connection.

### Implementation.DYNAMODB.Table.35

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb:ListTables`; check `$.TableNames[*]`
2. Call `dynamodb:DescribeTable` with `$.TableName`; check `$.Table.TableArn`
3. Call `dynamodb:GetResourcePolicy` with `$.ResourceArn`; check `$.Policy`

**Implements:** [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)

**Scoping** (for [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)): This check reads each table's attached resource-based policy at `$.Policy` and flags tables whose policy does not deny requests where `aws:SecureTransport` is false. That helps address the mitigation by surfacing tables where requests over unencrypted connections are not denied.

### Implementation.DYNAMODB.Table.36

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `*` | `aws:SecureTransport` |

**Implements:** [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)

**Scoping** (for [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)): This implementation restricts Amazon DynamoDB API calls against your resources with the `aws:SecureTransport` condition key, so requests that do not use TLS are denied. That helps address the mitigation by keeping read and write paths to your DynamoDB resources encrypted in transit.

**Reference:** To require encryption in transit, use the `aws:SecureTransport` condition key so that requests made over unencrypted connections are denied.
- [RCP sample: require HTTPS connections to your resources](https://github.com/aws-samples/resource-control-policy-examples/blob/main/Restrict-resource-access-patterns/Restrict-access-to-only-HTTPS-connections-to-your-resources.json)

### Implementation.DYNAMODB.Table.37

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:*` | `*` | `aws:SecureTransport` |

**Implements:** [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)

**Scoping** (for [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10)): This implementation restricts Amazon DynamoDB API calls against your tables with the `aws:SecureTransport` condition key. Requests that are not sent over TLS are denied, so principals in your organization cannot access your tables over unencrypted connections.

### Implementation.DYNAMODB.Table.38

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].Tags[*].Key`, `$.Replicas[*].Tags[*].Value`

**Implements:** [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)

**Scoping** (for [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)): This hook validates that your Amazon DynamoDB table declares the required owner and purpose tag keys in `$.Replicas[*].Tags[*].Key` and values in `$.Replicas[*].Tags[*].Value`. Failing a create request that omits those keys or values helps keep ownership and business function attributable for the table.

### Implementation.DYNAMODB.Table.39

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)

**Scoping** (for [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)): This implementation validates at create time that your table's tag keys and values, `$.Tags[*].Key` and `$.Tags[*].Value`, include owner and purpose entries. Creation is blocked when either entry is missing, so you can identify the owner and purpose of each table.

### Implementation.DYNAMODB.Table.40

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DynamoDB::Table`: `$.tags[*].key`, `$.tags[*].value`, `$.supplementaryConfiguration.Tags[*].key`, `$.supplementaryConfiguration.Tags[*].value`

**Implements:** [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)

**Scoping** (for [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)): This implementation flags tables whose recorded `$.tags[*].key` and `$.tags[*].value` entries omit the owner and purpose tags you require. Surfacing those tables helps you identify which of your data stores carry no accountable owner or documented business function.

### Implementation.DYNAMODB.Table.41

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:CreateTable`, `dynamodb:TagResource` | `arn:aws:dynamodb:*:*:table/*` | `aws:RequestTag/owner` |
| `dynamodb:CreateTable`, `dynamodb:TagResource` | `arn:aws:dynamodb:*:*:table/*` | `aws:RequestTag/purpose` |
| `dynamodb:CreateTable`, `dynamodb:TagResource` | `arn:aws:dynamodb:*:*:table/*` | `aws:TagKeys` |
| `dynamodb:UntagResource` | `arn:aws:dynamodb:*:*:table/*` | `aws:TagKeys` |

**Implements:** [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)

**Scoping** (for [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11)): This implementation restricts `dynamodb:CreateTable`, `dynamodb:TagResource`, and `dynamodb:UntagResource` on your table ARNs with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys. Requests that omit or remove the owner and purpose tag keys are denied, which helps keep owner and purpose metadata on your tables.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.DYNAMODB.Table.42

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ResourcePolicy.PolicyDocument`, `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12)

**Scoping** (for [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12)): This hook inspects the policy documents declared in `$.Replicas[*].ResourcePolicy.PolicyDocument` and `$.Replicas[*].ReplicaStreamSpecification.ResourcePolicy.PolicyDocument` before a replica is created or updated. Rejecting a document that removes all access to `dynamodb:PutResourcePolicy` helps you keep your table policy manageable.

### Implementation.DYNAMODB.Table.43

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.ResourcePolicy.PolicyDocument`, `$.StreamSpecification.ResourcePolicy.PolicyDocument`

**Implements:** [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12)

**Scoping** (for [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12)): This implementation validates the table policy declared in `$.ResourcePolicy.PolicyDocument` at create time, checking that it retains access to `dynamodb:PutResourcePolicy` and `dynamodb:DeleteResourcePolicy`. Rejecting a template that removes that access helps address the mitigation by keeping your table policy manageable after deployment.

### Implementation.DYNAMODB.Table.44

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `dynamodb:CreateTable` — `$.ResourcePolicy`, `$.TableName`
- `dynamodb:PutResourcePolicy` — `$.Policy`, `$.ResourceArn`, `$.x-amz-confirm-remove-self-resource-access` = `true`

**Implements:** [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12)

**Scoping** (for [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12)): Monitoring `PutResourcePolicy` records each resource-based policy write on your table, including the `$.Policy` document and the `$.x-amz-confirm-remove-self-resource-access` confirmation. Recording `CreateTable` calls that carry `$.ResourcePolicy` covers the creation path, so you can detect a table left without policy-management access.

### Implementation.DYNAMODB.Table.45

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].ContributorInsightsSpecification.Enabled`, `$.Replicas[*].ContributorInsightsSpecification.Mode`, `$.Replicas[*].GlobalSecondaryIndexes[*].ContributorInsightsSpecification.Enabled`, `$.Replicas[*].GlobalSecondaryIndexes[*].ContributorInsightsSpecification.Mode`, `$.Replicas[*].GlobalSecondaryIndexes[*].IndexName`, `$.TableName`

**Implements:** [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)

**Scoping** (for [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)): This implementation validates `$.Replicas[*].ContributorInsightsSpecification.Enabled` at create time, so a table whose replicas omit or disable CloudWatch Contributor Insights does not deploy. Each replica of your table then collects the more granular CloudWatch metrics that Contributor Insights provides.

### Implementation.DYNAMODB.Table.46

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.ContributorInsightsSpecification.Enabled`, `$.ContributorInsightsSpecification.Mode`, `$.GlobalSecondaryIndexes[*].ContributorInsightsSpecification.Enabled`, `$.GlobalSecondaryIndexes[*].ContributorInsightsSpecification.Mode`

**Implements:** [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)

**Scoping** (for [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)): This implementation validates `$.ContributorInsightsSpecification.Enabled`, the CloudWatch Contributor Insights setting on your table. Requiring it at create time helps address the mitigation by preventing tables that are provisioned without detailed CloudWatch metrics.

### Implementation.DYNAMODB.Table.47

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `dynamodb:UpdateContributorInsights` — `$.ContributorInsightsAction` = `DISABLE`, `$.ContributorInsightsMode` = `THROTTLED_KEYS`, `$.IndexName`, `$.TableName`

**Implements:** [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)

**Scoping** (for [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)): This implementation records `UpdateContributorInsights` calls carrying `$.ContributorInsightsAction` or `$.ContributorInsightsMode` values that reduce the metrics collected for your table. Watching this operation helps address the mitigation by surfacing attempts to stop the per-key access and throttling metrics.

### Implementation.DYNAMODB.Table.48

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dynamodb:ListTables`; check `$.TableNames[*]`
2. Call `dynamodb:DescribeContributorInsights` with `$.TableName`; check `$.ContributorInsightsStatus`, `$.ContributorInsightsMode`

**Implements:** [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)

**Scoping** (for [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13)): This periodic check reads `$.ContributorInsightsStatus` and `$.ContributorInsightsMode` for each of your DynamoDB tables. Flagging tables where Contributor Insights is off helps address the mitigation by making per-key access and throttling metrics available in CloudWatch for anomaly detection.

### Implementation.DYNAMODB.Table.49

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::GlobalTable`: `$.Replicas[*].DeletionProtectionEnabled`

**Implements:** [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)

**Scoping** (for [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)): This hook verifies that the deletion protection setting at `$.Replicas[*].DeletionProtectionEnabled` is enabled on your DynamoDB table. Stack operations that leave it disabled fail, helping you avoid deploying tables without protection from unintended deletion.

### Implementation.DYNAMODB.Table.50

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DynamoDB::Table`: `$.DeletionProtectionEnabled`

**Implements:** [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)

**Scoping** (for [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)): This implementation verifies that your template enables deletion protection through `$.DeletionProtectionEnabled` before a table is created. Because the setting is off by default, this check flags tables that would be created without deletion protection.

### Implementation.DYNAMODB.Table.51

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `dynamodb:DeleteTable` — `$.TableName`
- `dynamodb:UpdateTable` — `$.DeletionProtectionEnabled` = `false`, `$.TableName`

**Implements:** [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)

**Scoping** (for [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)): This implementation flags `UpdateTable` calls that set `$.DeletionProtectionEnabled` to false on your table, revealing removal of the guard that blocks deletion. Watching `DeleteTable` for the same `$.TableName` helps address the mitigation by exposing deletion attempts that follow, including those still refused while protection remains on.

### Implementation.DYNAMODB.Table.52

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DynamoDB::Table`: `$.configuration.tableName`

**Implements:** [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)

**Scoping** (for [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)): This check resolves the table named in `$.configuration.tableName` and reads its deletion protection setting from `$.Table.DeletionProtectionEnabled`. Flagging a table where deletion protection is disabled helps address the mitigation by identifying tables that lack this safeguard against accidental deletion.

### Implementation.DYNAMODB.Table.53

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:DeleteTable`, `dynamodb:DeleteTableReplica` | `arn:aws:dynamodb:*:*:table/*` | `aws:PrincipalArn` (alternatives: `aws:PrincipalTag/${TagKey}`), `aws:PrincipalIsAWSService` |

**Implements:** [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)

**Scoping** (for [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)): This implementation restricts `dynamodb:DeleteTable` and `dynamodb:DeleteTableReplica` with the `aws:PrincipalArn` condition key, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). Requests that do not satisfy that condition are denied, so table deletion is restricted even when the table's deletion protection setting `$.DeletionProtectionEnabled` is off.

**Reference:** To restrict access to select principals, use the `aws:PrincipalArn` or `aws:PrincipalTag/tag-key` condition keys. Use `aws:PrincipalArn` to specify the Amazon Resource Names (ARNs) of the principals. Those ARNs can follow a naming convention or an IAM path that scopes the restriction to a set of principals without enumerating each one. Use `aws:PrincipalTag/tag-key` to identify those principals by a tag attached to them. Choose the condition key that matches your IAM policy management strategy. Apply the restriction only when `aws:PrincipalIsAWSService` is `false`, so that AWS services accessing your resources using their service principals continue to operate as expected.

### Implementation.DYNAMODB.Table.54

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dynamodb:DeleteTable`, `dynamodb:DeleteTableReplica` | `arn:aws:dynamodb:*:*:table/*` | `aws:PrincipalArn` (alternatives: `aws:PrincipalTag/${TagKey}`) |

**Implements:** [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)

**Scoping** (for [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)): This implementation restricts `dynamodb:DeleteTable` and `dynamodb:DeleteTableReplica` on your tables with the `aws:PrincipalArn` condition key. Deletion requests that do not satisfy that condition are denied, complementing the table's `$.DeletionProtectionEnabled` setting and helping guard against accidental or unauthorized deletion.

**Reference:** To restrict access to select principals, use the `aws:PrincipalArn` or `aws:PrincipalTag/tag-key` condition keys. Use `aws:PrincipalArn` to specify the Amazon Resource Names (ARNs) of the principals. Those ARNs can follow a naming convention or an IAM path that scopes the restriction to a set of principals without enumerating each one. Use `aws:PrincipalTag/tag-key` to identify those principals by a tag attached to them. Choose the condition key that matches your IAM policy management strategy.

### ManagedControl.DYNAMODB.Table.1

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require that point-in-time recovery for an Amazon DynamoDB table is activated

**Aliases:** `CT.DYNAMODB.PR.1`

**ARN:** `arn:aws:controlcatalog:::control/4docid6lj7n5tstdmm7btegt1`

This control checks whether point-in-time recovery (PITR) is enabled for an Amazon DynamoDB table.

**Implements:** [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)

### ManagedControl.DYNAMODB.Table.2

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if point-in-time recovery (PITR) is enabled for Amazon DynamoDB tables

**Aliases:** `CONFIG.DYNAMODB.DT.8`

**Identifier:** `DYNAMODB_PITR_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/84nv76hw5unav96w8dhvnx5pw`

Checks if point-in-time recovery (PITR) is enabled for Amazon DynamoDB tables. The rule is NON_COMPLIANT if PITR is not enabled for DynamoDB tables.

**Implements:** [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2)

### ManagedControl.DYNAMODB.Table.3

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if a recovery point was created for Amazon DynamoDB Tables within the specified period

**Aliases:** `CONFIG.DYNAMODB.DT.3`

**Identifier:** `DYNAMODB_LAST_BACKUP_RECOVERY_POINT_CREATED`

**ARN:** `arn:aws:controlcatalog:::control/4gqcm1tpsuij5vmhtk4n6b4vf`

Checks if a recovery point was created for Amazon DynamoDB Tables within the specified period. The rule is NON_COMPLIANT if the DynamoDB Table does not have a corresponding recovery point created within the specified time period.

**Implements:** [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)

### ManagedControl.DYNAMODB.Table.4

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon DynamoDB tables are protected by a backup plan

**Aliases:** `CONFIG.DYNAMODB.DT.2`

**Identifier:** `DYNAMODB_RESOURCES_PROTECTED_BY_BACKUP_PLAN`

**ARN:** `arn:aws:controlcatalog:::control/6am65w27qcfni6boy9ote2vpr`

Checks if Amazon DynamoDB tables are protected by a backup plan. The rule is NON_COMPLIANT if the DynamoDB Table is not covered by a backup plan.

**Implements:** [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)

### ManagedControl.DYNAMODB.Table.5

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if an Amazon DynamoDB table is present in AWS Backup plans

**Aliases:** `CONFIG.DYNAMODB.DT.7`

**Identifier:** `DYNAMODB_IN_BACKUP_PLAN`

**ARN:** `arn:aws:controlcatalog:::control/dm91qhaj7bjtyrovbq0szj49u`

Checks if an Amazon DynamoDB table is present in AWS Backup plans. The rule is NON_COMPLIANT if DynamoDB tables are not present in any AWS Backup plan.

**Implements:** [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)

### ManagedControl.DYNAMODB.Table.6

*AWS Config rule — Detective*

AWS managed control from General controls (service-agnostic — no governed resource).

**Title:** Checks if a recovery point expires no earlier than after the specified period

**Aliases:** `BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK`

**Identifier:** `BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK`

**ARN:** `arn:aws:controlcatalog:::control/m7a5gbdf08wg2o0en010mkng`

Checks if a recovery point expires no earlier than after the specified period. The rule is NON_COMPLIANT if the recovery point has a retention point that is less than the required retention period.

**Implements:** [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4)

### ManagedControl.DYNAMODB.Table.7

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks whether Amazon DynamoDB table is encrypted with AWS Key Management Service (KMS)

**Aliases:** `CONFIG.DYNAMODB.DT.4`

**Identifier:** `DYNAMODB_TABLE_ENCRYPTED_KMS`

**ARN:** `arn:aws:controlcatalog:::control/a0ufrfbomg90qbskltuop262u`

Checks whether Amazon DynamoDB table is encrypted with AWS Key Management Service (KMS). The rule is NON_COMPLIANT if Amazon DynamoDB table is not encrypted with AWS KMS.

**Implements:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

### ManagedControl.DYNAMODB.Table.8

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require an Amazon DynamoDB table to be encrypted at rest using an AWS KMS key

**Aliases:** `CT.DYNAMODB.PR.2`

**ARN:** `arn:aws:controlcatalog:::control/alyxc1dmqnr9nzz4luslcsl44`

This control checks whether your Amazon DynamoDB table is encrypted with an AWS Key Management Service (KMS) key.

**Implements:** [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5)

### ManagedControl.DYNAMODB.Table.9

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if an Amazon DynamoDB table have deletion protection set to enabled

**Aliases:** `CONFIG.DYNAMODB.DT.6`

**Identifier:** `DYNAMODB_TABLE_DELETION_PROTECTION_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/1h5467ejzh5x9hy9vdopyzvv`

Checks if an Amazon DynamoDB table have deletion protection set to enabled. The rule is NON_COMPLIANT if the table have deletion protection set to disabled.

**Implements:** [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14)

---

## Appendix

### Resources

#### Backup

*DATA_STORE*

An Amazon DynamoDB backup is a full, self-contained snapshot of one table's data at a point in time. It captures every item together with the table's secondary indexes, stream settings, and throughput configuration. Creating a backup does not interrupt the table and does not draw on its read or write capacity.

- IAM resource ARN: `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/backup/${BackupName}`
- CloudFormation: `AWS::DynamoDB::Backup`
- 4 operations, 0 threats, 0 mitigations

#### GlobalTable

*CONFIGURATION*

A DynamoDB global table joins DynamoDB tables in different AWS Regions into one replication group. Each member table, called a replica, stores the same complete set of items. An application can read from and write to any replica, and DynamoDB propagates each write to the replicas in the other Regions.

- IAM resource ARN: `arn:${Partition}:dynamodb::${Account}:global-table/${GlobalTableName}`
- CloudFormation: `AWS::DynamoDB::GlobalTable`
- AWS Config: `AWS::DynamoDB::Table`
- 6 operations, 1 threats, 1 mitigations

#### Stream

*INTEGRATION*

An Amazon DynamoDB stream captures item-level changes to a table as an ordered log of change records. Every write, update, and delete on the table appears in the stream in near-real time. Applications read these records to drive event-driven processing, cross-Region replication, archiving, and near-real-time analytics.

- IAM resource ARN: `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/stream/${StreamLabel}`
- CloudFormation: `AWS::DynamoDB::Stream`, `AWS::DynamoDB::Table`, `AWS::DynamoDB::GlobalTable`
- AWS Config: `AWS::DynamoDB::Table`
- 6 operations, 7 threats, 7 mitigations

#### Table

*DATA_STORE*

An Amazon DynamoDB table is the primary container for data in DynamoDB. It stores a collection of items, and each item is a set of attributes. Every item is addressed by a primary key, either a partition key alone or a partition key combined with a sort key. Apart from the key, tables are schemaless, so each item can carry its own attributes.

- IAM resource ARNs: `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/export/${ExportName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/import/${ImportName}`, `arn:${Partition}:dynamodb:${Region}:${Account}:table/${TableName}/index/${IndexName}`
- CloudFormation: `AWS::DynamoDB::Table`, `AWS::DynamoDB::GlobalTable`, `AWS::DynamoDB::Export`
- AWS Config: `AWS::DynamoDB::Table`
- 50 operations, 18 threats, 14 mitigations

### Operations

| Operation | Analyses | Resources | Mitigations |
|---|---|---|---|
| `BatchExecuteStatement` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5) |
| `BatchGetItem` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `BatchWriteItem` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `CreateBackup` | DOCUMENTATION, DATA_PERIMETER | [Backup](#backup) |  |
| `CreateGlobalTable` | DOCUMENTATION, DATA_PERIMETER | [GlobalTable](#globaltable) | [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1) |
| `CreateTable` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9), [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10), [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11), [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13), [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14) |
| `DeleteBackup` | DOCUMENTATION, DATA_PERIMETER | [Backup](#backup) |  |
| `DeleteItem` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `DeleteResourcePolicy` | DOCUMENTATION, DATA_PERIMETER | [Stream](#stream), [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1), [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8) |
| `DeleteTable` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14) |
| `DescribeBackup` | DOCUMENTATION, DATA_PERIMETER | [Backup](#backup) |  |
| `DescribeContinuousBackups` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2) |
| `DescribeContributorInsights` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `DescribeEndpoints` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `DescribeExport` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `DescribeGlobalTable` | DOCUMENTATION, DATA_PERIMETER | [GlobalTable](#globaltable) | [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1) |
| `DescribeGlobalTableSettings` | DOCUMENTATION, DATA_PERIMETER | [GlobalTable](#globaltable), [Table](#table) | [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1) |
| `DescribeImport` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `DescribeKinesisStreamingDestination` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `DescribeLimits` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `DescribeTable` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14) |
| `DescribeTableReplicaAutoScaling` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7) |
| `DescribeTimeToLive` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `DisableKinesisStreamingDestination` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `EnableKinesisStreamingDestination` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `ExecuteStatement` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5) |
| `ExecuteTransaction` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4) |
| `ExportTableToPointInTime` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6) |
| `GetItem` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `GetResourcePolicy` | DOCUMENTATION, DATA_PERIMETER | [Stream](#stream), [Table](#table) | [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1), [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3), [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4), [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5), [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9), [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10), [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12) |
| `ImportTable` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `ListBackups` | DOCUMENTATION, DATA_PERIMETER | [Backup](#backup) |  |
| `ListContributorInsights` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `ListExports` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `ListGlobalTables` | DOCUMENTATION, DATA_PERIMETER | [GlobalTable](#globaltable) | [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1) |
| `ListImports` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `ListTables` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `ListTagsOfResource` | DOCUMENTATION, DATA_PERIMETER | [Stream](#stream), [Table](#table) | [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11) |
| `PutItem` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `PutResourcePolicy` | DOCUMENTATION, DATA_PERIMETER | [Stream](#stream), [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1), [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3), [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4), [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5), [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9), [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10), [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12) |
| `Query` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `RestoreTableFromBackup` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `RestoreTableToPointInTime` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `Scan` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `SearchVectors` | DOCUMENTATION | [Table](#table) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5) |
| `TagResource` | DOCUMENTATION, DATA_PERIMETER | [Stream](#stream), [Table](#table) | [Mitigation.DYNAMODB.Stream.2](#mitigationdynamodbstream2), [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4), [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6), [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11) |
| `TransactGetItems` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6) |
| `TransactWriteItems` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4) |
| `UntagResource` | DOCUMENTATION, DATA_PERIMETER | [Stream](#stream), [Table](#table) | [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4), [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6), [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11) |
| `UpdateContinuousBackups` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2) |
| `UpdateContributorInsights` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `UpdateGlobalTable` | DOCUMENTATION, DATA_PERIMETER | [GlobalTable](#globaltable) | [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1) |
| `UpdateGlobalTableSettings` | DOCUMENTATION, DATA_PERIMETER | [GlobalTable](#globaltable), [Table](#table) |  |
| `UpdateItem` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.2](#mitigationdynamodb2), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| `UpdateKinesisStreamingDestination` | DOCUMENTATION | [Table](#table) |  |
| `UpdateTable` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) | [Mitigation.DYNAMODB.1](#mitigationdynamodb1), [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3), [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8), [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14) |
| `UpdateTableReplicaAutoScaling` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |
| `UpdateTimeToLive` | DOCUMENTATION, DATA_PERIMETER | [Table](#table) |  |

Mitigations attach to an operation directly (the operation appears in the mitigation's request parameters) or through a threat whose request parameters include the operation.

### Goal Coverage (16 of 38)

| Mitigation Goal | Covered | Mitigations |
|---|---|---|
| [AI Services Opt-Out Policy](../mitigation-goals.md#ai-services-opt-out-policy) | — |  |
| [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup) | yes | [Mitigation.DYNAMODB.Table.4](#mitigationdynamodbtable4) |
| [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native) | yes | [Mitigation.DYNAMODB.Table.2](#mitigationdynamodbtable2) |
| [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery) | — |  |
| [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2) | — |  |
| [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest) | — |  |
| [Cross-Region Replication](../mitigation-goals.md#cross-region-replication) | yes | [Mitigation.DYNAMODB.GlobalTable.1](#mitigationdynamodbglobaltable1), [Mitigation.DYNAMODB.Table.7](#mitigationdynamodbtable7) |
| [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies) | yes | [Mitigation.DYNAMODB.Table.9](#mitigationdynamodbtable9) |
| [Customer Code Execution - Govern Code Content](../mitigation-goals.md#customer-code-execution---govern-code-content) | — |  |
| [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage) | — |  |
| [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection) | yes | [Mitigation.DYNAMODB.Table.14](#mitigationdynamodbtable14) |
| [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics) | yes | [Mitigation.DYNAMODB.Table.13](#mitigationdynamodbtable13) |
| [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics) | — |  |
| [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy) | yes | [Mitigation.DYNAMODB.Stream.5](#mitigationdynamodbstream5), [Mitigation.DYNAMODB.Table.10](#mitigationdynamodbtable10) |
| [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature) | — |  |
| [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication) | — |  |
| [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network) | yes | [Mitigation.DYNAMODB.2](#mitigationdynamodb2) |
| [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources) | yes | [Mitigation.DYNAMODB.Stream.4](#mitigationdynamodbstream4), [Mitigation.DYNAMODB.Table.8](#mitigationdynamodbtable8) |
| [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption) | yes | [Mitigation.DYNAMODB.Table.5](#mitigationdynamodbtable5) |
| [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment) | — |  |
| [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment) | — |  |
| [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks) | — |  |
| [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks) | yes | [Mitigation.DYNAMODB.Stream.1](#mitigationdynamodbstream1), [Mitigation.DYNAMODB.Table.1](#mitigationdynamodbtable1) |
| [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement) | — |  |
| [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets) | — |  |
| [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources) | yes | [Mitigation.DYNAMODB.Stream.2](#mitigationdynamodbstream2), [Mitigation.DYNAMODB.Table.3](#mitigationdynamodbtable3) |
| [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network) | yes | [Mitigation.DYNAMODB.1](#mitigationdynamodb1) |
| [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition) | yes | [Mitigation.DYNAMODB.Stream.3](#mitigationdynamodbstream3), [Mitigation.DYNAMODB.Table.6](#mitigationdynamodbtable6) |
| [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention) | yes | [Mitigation.DYNAMODB.Stream.7](#mitigationdynamodbstream7), [Mitigation.DYNAMODB.Table.12](#mitigationdynamodbtable12) |
| [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose) | yes | [Mitigation.DYNAMODB.Stream.6](#mitigationdynamodbstream6), [Mitigation.DYNAMODB.Table.11](#mitigationdynamodbtable11) |
| [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role) | — |  |
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
| [Account Access Removal](../threat-impacts.md#account-access-removal) | [Threat.DYNAMODB.Table.2](#threatdynamodbtable2) |
| [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery) | [Threat.DYNAMODB.Stream.7](#threatdynamodbstream7), [Threat.DYNAMODB.Table.18](#threatdynamodbtable18) |
| [Data Collection](../threat-impacts.md#data-collection) | [Threat.DYNAMODB.1](#threatdynamodb1), [Threat.DYNAMODB.2](#threatdynamodb2), [Threat.DYNAMODB.Stream.4](#threatdynamodbstream4), [Threat.DYNAMODB.Stream.6](#threatdynamodbstream6), [Threat.DYNAMODB.Stream.7](#threatdynamodbstream7), [Threat.DYNAMODB.Table.7](#threatdynamodbtable7), [Threat.DYNAMODB.Table.8](#threatdynamodbtable8), [Threat.DYNAMODB.Table.10](#threatdynamodbtable10), [Threat.DYNAMODB.Table.12](#threatdynamodbtable12), [Threat.DYNAMODB.Table.13](#threatdynamodbtable13), [Threat.DYNAMODB.Table.15](#threatdynamodbtable15), [Threat.DYNAMODB.Table.16](#threatdynamodbtable16), [Threat.DYNAMODB.Table.18](#threatdynamodbtable18) |
| [Data Destruction](../threat-impacts.md#data-destruction) | [Threat.DYNAMODB.Stream.1](#threatdynamodbstream1), [Threat.DYNAMODB.Table.1](#threatdynamodbtable1), [Threat.DYNAMODB.Table.2](#threatdynamodbtable2), [Threat.DYNAMODB.Table.3](#threatdynamodbtable3), [Threat.DYNAMODB.Table.11](#threatdynamodbtable11), [Threat.DYNAMODB.Table.13](#threatdynamodbtable13), [Threat.DYNAMODB.Table.14](#threatdynamodbtable14) |
| [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Threat.DYNAMODB.Table.5](#threatdynamodbtable5) |
| [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Threat.DYNAMODB.1](#threatdynamodb1), [Threat.DYNAMODB.2](#threatdynamodb2), [Threat.DYNAMODB.Stream.3](#threatdynamodbstream3), [Threat.DYNAMODB.Stream.4](#threatdynamodbstream4), [Threat.DYNAMODB.Stream.5](#threatdynamodbstream5), [Threat.DYNAMODB.Stream.6](#threatdynamodbstream6), [Threat.DYNAMODB.Table.8](#threatdynamodbtable8), [Threat.DYNAMODB.Table.10](#threatdynamodbtable10), [Threat.DYNAMODB.Table.11](#threatdynamodbtable11), [Threat.DYNAMODB.Table.12](#threatdynamodbtable12), [Threat.DYNAMODB.Table.13](#threatdynamodbtable13), [Threat.DYNAMODB.Table.15](#threatdynamodbtable15), [Threat.DYNAMODB.Table.16](#threatdynamodbtable16), [Threat.DYNAMODB.Table.17](#threatdynamodbtable17) |
| [Defense Evasion](../threat-impacts.md#defense-evasion) | [Threat.DYNAMODB.Stream.2](#threatdynamodbstream2), [Threat.DYNAMODB.Stream.5](#threatdynamodbstream5), [Threat.DYNAMODB.Table.4](#threatdynamodbtable4), [Threat.DYNAMODB.Table.6](#threatdynamodbtable6), [Threat.DYNAMODB.Table.9](#threatdynamodbtable9), [Threat.DYNAMODB.Table.11](#threatdynamodbtable11) |
| [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Threat.DYNAMODB.Table.3](#threatdynamodbtable3), [Threat.DYNAMODB.Table.5](#threatdynamodbtable5) |
| [Initial Access](../threat-impacts.md#initial-access) | [Threat.DYNAMODB.Stream.5](#threatdynamodbstream5), [Threat.DYNAMODB.Stream.6](#threatdynamodbstream6), [Threat.DYNAMODB.Table.11](#threatdynamodbtable11), [Threat.DYNAMODB.Table.13](#threatdynamodbtable13) |
| [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Threat.DYNAMODB.Table.17](#threatdynamodbtable17) |
| [Service Stop](../threat-impacts.md#service-stop) | [Threat.DYNAMODB.GlobalTable.1](#threatdynamodbglobaltable1), [Threat.DYNAMODB.Stream.1](#threatdynamodbstream1) |
