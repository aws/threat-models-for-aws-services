# Amazon Aurora DSQL (dsql) Threat Model

[Threat Summary](#threat-summary) · [Threats](#threats) · [Mitigations](#mitigations) · [Implementations](#implementations) · [Appendix](#appendix)

## Overview

Amazon Aurora DSQL is a serverless, distributed relational database service built for transactional workloads. It presents a PostgreSQL-compatible interface, so existing drivers, object-relational mappings, and SQL tooling work with minimal change. Compute, I/O, and storage scale automatically with demand. There are no database instances to provision, patch, or upgrade.

17 threats and 14 mitigations across 2 resource types and 16 API operations. 13 of 38 mitigation goals are covered, implemented by 38 implementations.

Source service model: [dsql](https://github.com/aws/api-models-aws/blob/a0767ac42e279dfc8b311564c33436f8abfd3031/models/dsql/service/2018-05-10/dsql-2018-05-10.json)

---

## Threat Summary

### [Cross-service confused deputy problem through resource-based policies](../potential-threats.md#cross-service-confused-deputy-problem-through-resource-based-policies)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.13](#threatdsqlcluster13) | Untrusted IAM principal, with access to configure an AWS service that has been granted access to an Aurora DSQL cluster through its resource-based policy, may be able to cause that AWS service to interact with the cluster using the principal's data or context, such as coercing AWS Backup into reading the cluster and producing a full cluster backup on their behalf. | [Data Collection](../threat-impacts.md#data-collection), [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7) |

### [Data inaccessibility through resource-based policy lockout](../potential-threats.md#data-inaccessibility-through-resource-based-policy-lockout)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.2](#threatdsqlcluster2) | IAM principal, with IAM permissions to modify a resource-based policy governing an Aurora DSQL cluster, can make the cluster inaccessible by modifying the cluster resource-based policy to deny all principals both policy management and database connect actions. | [Account Access Removal](../threat-impacts.md#account-access-removal), [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9) |

### [Data loss through encryption key deletion](../potential-threats.md#data-loss-through-encryption-key-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.3](#threatdsqlcluster3) | IAM principal, with IAM permissions to delete the AWS KMS customer managed key used by an Aurora DSQL cluster, can delete the customer managed key used by the cluster. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5) |

### [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.4](#threatdsqlcluster4) | IAM principal, with IAM permissions to create or modify data in an Aurora DSQL cluster, can encrypt the data stored in the cluster with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4), [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5) |

### [Data loss through resource deletion](../potential-threats.md#data-loss-through-resource-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.1](#threatdsqlcluster1) | IAM principal, with IAM permissions to delete an Aurora DSQL cluster, can delete the cluster and the database, schemas, and tables that the cluster contains. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3), [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5), [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10) |

### [Direct data deletion](../potential-threats.md#direct-data-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.11](#threatdsqlcluster11) | IAM principal, with IAM permissions to delete rows and tables from an Aurora DSQL cluster, can delete rows and tables from the cluster, destroying specific records or entire tables while the cluster itself remains intact. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3) |

### [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.1](#threatdsql1) | Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Aurora DSQL service API calls. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DSQL.2](#mitigationdsql2) |

### [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.10](#threatdsqlcluster10) | Untrusted IAM principal, that has been granted access to my Aurora DSQL cluster through its resource-based policy, can use my cluster. | [Data Collection](../threat-impacts.md#data-collection), [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6) |

### [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Stream.2](#threatdsqlstream2) | Untrusted IAM principal, that has been granted access to my Aurora DSQL stream's CDC service role through its trust policy, can assume my CDC service role and act with its permissions. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1) |

### [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.6](#threatdsqlcluster6) | IAM principal, with IAM permissions to create or modify an Aurora DSQL cluster, can encrypt the cluster with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4) |

### [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.8](#threatdsqlcluster8) | IAM principal, with IAM permissions on my Aurora DSQL cluster and access to an unexpected network, can interact with my cluster from an unexpected network, connecting to its PostgreSQL endpoint to read stored data or calling control plane operations to reconfigure or delete the cluster. | [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1) |

### [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.5](#threatdsqlcluster5) | IAM principal, with IAM permissions to create or modify an Aurora DSQL cluster, can configure the cluster without required tagging information, by omitting governance tags such as owner, purpose, environment or data classification when creating the cluster, or by removing or overwriting those tags on an existing cluster. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8) |
| [Threat.DSQL.Stream.1](#threatdsqlstream1) | IAM principal, with IAM permissions to create or modify an Aurora DSQL stream, can configure the stream without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2) |

### [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.2](#threatdsql2) | IAM principal, with IAM permissions on untrusted Aurora DSQL resources, can interact with those untrusted resources from a governed VPC network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DSQL.1](#mitigationdsql1) |

### [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.7](#threatdsqlcluster7) | My IAM principal, with IAM permissions to create, modify, delete, or read an untrusted Amazon Aurora DSQL cluster, can interact with the untrusted cluster, including sending data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2) |

### [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.12](#threatdsqlcluster12) | IAM principal, with unintended IAM permissions to read the table data from an Aurora DSQL cluster that has been encrypted with an AWS owned KMS key, can read the table data in the cluster. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4) |

### [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.DSQL.Cluster.9](#threatdsqlcluster9) | IAM principal, with unintended IAM permissions to read user and system table data from an Aurora DSQL cluster that has not been encrypted at rest with a KMS customer managed key, can read plaintext user and system table data in the cluster. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4) |

---

## Threats

Each scoping statement below describes the exposure the threat's mapped mitigations address. It assumes those mitigations are not applied and makes no claim about your resources.

### Threat.DSQL.1

Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Aurora DSQL service API calls.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DSQL.2](#mitigationdsql2)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Protocols:** `PostgreSQL`

**Scoping** (for [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)): Aurora DSQL is reachable from your VPC through an interface VPC endpoint (powered by AWS PrivateLink), for control-plane calls and cluster connections. An identity outside your organization that operates on that network can make Aurora DSQL API calls with its own credentials. That access enables transfer of data from your VPC to Aurora DSQL clusters in an AWS account outside your organization.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateStream` | `$.clusterIdentifier`, `$.targetDefinition.kinesis.roleArn`, `$.targetDefinition.kinesis.streamArn` |
| `dsql:DeleteCluster` | `$.identifier` |
| `dsql:DeleteClusterPolicy` | `$.identifier` |
| `dsql:PutClusterPolicy` | `$.bypassPolicyLockoutSafetyCheck`, `$.identifier`, `$.policy` |
| `dsql:UpdateCluster` | `$.deletionProtectionEnabled`, `$.identifier`, `$.kmsEncryptionKey` |

### Threat.DSQL.2

IAM principal, with IAM permissions on untrusted Aurora DSQL resources, can interact with those untrusted resources from a governed VPC network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DSQL.1](#mitigationdsql1)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Protocols:** `PostgreSQL`

**Scoping** (for [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)): Aurora DSQL does not currently support cross-account access through resource-based policies, so your principals can access only clusters in their own account. If cross-account access becomes available, an IAM principal operating from a governed VPC network could then reach Aurora DSQL resources in accounts outside your organization.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateStream` | `$.clusterIdentifier`, `$.targetDefinition.kinesis.roleArn`, `$.targetDefinition.kinesis.streamArn` |
| `dsql:DeleteCluster` | `$.identifier` |
| `dsql:PutClusterPolicy` | `$.identifier`, `$.policy` |
| `dsql:UpdateCluster` | `$.identifier`, `$.multiRegionProperties.clusters[*]` |

### Threat.DSQL.Cluster.1

IAM principal, with IAM permissions to delete an Aurora DSQL cluster, can delete the cluster and the database, schemas, and tables that the cluster contains.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3), [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5), [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Scoping** (for [Data loss through resource deletion](../potential-threats.md#data-loss-through-resource-deletion)): An Aurora DSQL cluster holds your database, schemas, and tables. An IAM principal with permission to delete the cluster can delete that data along with it.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:DeleteCluster` | `$.identifier` |

### Threat.DSQL.Cluster.2

IAM principal, with IAM permissions to modify a resource-based policy governing an Aurora DSQL cluster, can make the cluster inaccessible by modifying the cluster resource-based policy to deny all principals both policy management and database connect actions.

**Impacts:** [Account Access Removal](../threat-impacts.md#account-access-removal), [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/), [T1531](https://attack.mitre.org/techniques/T1531/)

**Scoping** (for [Data inaccessibility through resource-based policy lockout](../potential-threats.md#data-inaccessibility-through-resource-based-policy-lockout)): The resource-based policy on your Aurora DSQL cluster governs both its policy-management operations and its database connect actions. An IAM principal with permissions to modify that policy can deny those actions to all principals and may leave the cluster inaccessible.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.bypassPolicyLockoutSafetyCheck`, `$.policy` |
| `dsql:PutClusterPolicy` | `$.bypassPolicyLockoutSafetyCheck`, `$.identifier`, `$.policy` |

### Threat.DSQL.Cluster.3

IAM principal, with IAM permissions to delete the AWS KMS customer managed key used by an Aurora DSQL cluster, can delete the customer managed key used by the cluster.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Scoping** (for [Data loss through encryption key deletion](../potential-threats.md#data-loss-through-encryption-key-deletion)): Your Aurora DSQL cluster can encrypt data at rest under an AWS KMS customer managed key that you select. A threat actor with permissions to delete that key can leave your cluster data unreadable after the deletion waiting period ends.

**Request parameters**

| Operation | Parameters |
|---|---|
| `kms:DeleteImportedKeyMaterial` | `$.KeyId`, `$.KeyMaterialId` |
| `kms:DisconnectCustomKeyStore` | `$.CustomKeyStoreId` |
| `kms:ScheduleKeyDeletion` | `$.KeyId`, `$.PendingWindowInDays` |

### Threat.DSQL.Cluster.4

IAM principal, with IAM permissions to create or modify data in an Aurora DSQL cluster, can encrypt the data stored in the cluster with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact)

**Mitigations:** [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4), [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal able to create or modify your cluster chooses the KMS key protecting its data at rest. Whoever controls that key can later remove its key material and leave your cluster data unreadable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.kmsEncryptionKey` |
| `dsql:UpdateCluster` | `$.identifier`, `$.kmsEncryptionKey` |

### Threat.DSQL.Cluster.5

IAM principal, with IAM permissions to create or modify an Aurora DSQL cluster, can configure the cluster without required tagging information, by omitting governance tags such as owner, purpose, environment or data classification when creating the cluster, or by removing or overwriting those tags on an existing cluster.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal that can create or modify your Aurora DSQL cluster can omit governance tags such as owner and purpose, or overwrite them. Such a cluster keeps serving traffic and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.tags{key}` |
| `dsql:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `dsql:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.DSQL.Cluster.6

IAM principal, with IAM permissions to create or modify an Aurora DSQL cluster, can encrypt the cluster with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal able to create or modify your cluster chooses whether the AWS owned key protects its data at rest. Because that key resides in an Aurora DSQL service account rather than your own, its permissions, lifecycle, and usage records sit outside your control. No key policy of yours governs its use, you cannot disable it or schedule its deletion, and the service's use of it produces no CloudTrail events in your account. A customer managed key would restore those governance controls, though its key policy authorizes the Aurora DSQL service principal for the cluster as a whole rather than gating individual callers.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.kmsEncryptionKey` |
| `dsql:UpdateCluster` | `$.identifier`, `$.kmsEncryptionKey` |

### Threat.DSQL.Cluster.7

My IAM principal, with IAM permissions to create, modify, delete, or read an untrusted Amazon Aurora DSQL cluster, can interact with the untrusted cluster, including sending data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Protocols:** `PostgreSQL`

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): Aurora DSQL does not currently support cross-account access through resource-based policies, so your principals can access only clusters in their own account. If cross-account access becomes available, your principals could then create, modify, or delete an external cluster, or read and write its data.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.multiRegionProperties.clusters[*]` |
| `dsql:DeleteCluster` | `$.identifier` |
| `dsql:PutClusterPolicy` | `$.identifier`, `$.policy` |
| `dsql:UpdateCluster` | `$.identifier`, `$.kmsEncryptionKey`, `$.multiRegionProperties.clusters[*]` |

### Threat.DSQL.Cluster.8

IAM principal, with IAM permissions on my Aurora DSQL cluster and access to an unexpected network, can interact with my cluster from an unexpected network, connecting to its PostgreSQL endpoint to read stored data or calling control plane operations to reconfigure or delete the cluster.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1485](https://attack.mitre.org/techniques/T1485/), [T1556.009](https://attack.mitre.org/techniques/T1556/009/)

**Protocols:** `PostgreSQL`

**Scoping** (for [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)): Your Aurora DSQL cluster exposes a PostgreSQL endpoint that is reachable from the public internet by default. An IAM principal with connect or cluster management permissions can reach your Aurora DSQL cluster from an unexpected network. That principal can read stored data through the PostgreSQL endpoint, or reconfigure or delete the cluster.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:DeleteCluster` | `$.identifier` |
| `dsql:DeleteClusterPolicy` | `$.identifier` |
| `dsql:PutClusterPolicy` | `$.bypassPolicyLockoutSafetyCheck`, `$.identifier`, `$.policy` |
| `dsql:UpdateCluster` | `$.deletionProtectionEnabled`, `$.identifier`, `$.kmsEncryptionKey` |

### Threat.DSQL.Cluster.9

IAM principal, with unintended IAM permissions to read user and system table data from an Aurora DSQL cluster that has not been encrypted at rest with a KMS customer managed key, can read plaintext user and system table data in the cluster.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/)

**Protocols:** `PostgreSQL`

**Scoping** (for [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)): By default, an AWS owned key protects the cluster's data at rest, and Aurora DSQL decrypts it transparently on access under a key policy you do not control. An IAM principal granted broader permissions than intended can therefore read user and system table data in plaintext over a PostgreSQL session, with no customer managed key policy available to deny the decryption performed on its behalf.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.kmsEncryptionKey` |
| `dsql:UpdateCluster` | `$.identifier`, `$.kmsEncryptionKey` |

### Threat.DSQL.Cluster.10

Untrusted IAM principal, that has been granted access to my Aurora DSQL cluster through its resource-based policy, can use my cluster.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1485](https://attack.mitre.org/techniques/T1485/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Protocols:** `PostgreSQL`

**Scoping** (for [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)): Aurora DSQL does not currently support cross-account access through resource-based policies, so your clusters can be accessed only by principals in the same account. If cross-account access becomes available, an untrusted IAM principal named in the policy could read, alter, or drop your schemas, tables, and rows.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:DeleteCluster` | `$.identifier` |
| `dsql:DeleteClusterPolicy` | `$.identifier` |
| `dsql:PutClusterPolicy` | `$.identifier`, `$.policy` |
| `dsql:UpdateCluster` | `$.deletionProtectionEnabled`, `$.identifier`, `$.kmsEncryptionKey` |

### Threat.DSQL.Cluster.11

IAM principal, with IAM permissions to delete rows and tables from an Aurora DSQL cluster, can delete rows and tables from the cluster, destroying specific records or entire tables while the cluster itself remains intact.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Protocols:** `PostgreSQL`

**Scoping** (for [Direct data deletion](../potential-threats.md#direct-data-deletion)): An Aurora DSQL cluster stores your data as rows in tables in a PostgreSQL-compatible database. A principal authorized to delete that data can remove individual rows or drop entire tables, while the cluster resource remains.

### Threat.DSQL.Cluster.12

IAM principal, with unintended IAM permissions to read the table data from an Aurora DSQL cluster that has been encrypted with an AWS owned KMS key, can read the table data in the cluster.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1213.006](https://attack.mitre.org/techniques/T1213/006/)

**Protocols:** `PostgreSQL`

**Scoping** (for [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)): Aurora DSQL encrypts cluster data at rest and decrypts it transparently for any authenticated connection, regardless of which KMS key protects it. An IAM principal granted broader permissions than intended can therefore retrieve table data in plaintext over an ordinary PostgreSQL session.

### Threat.DSQL.Cluster.13

Untrusted IAM principal, with access to configure an AWS service that has been granted access to an Aurora DSQL cluster through its resource-based policy, may be able to cause that AWS service to interact with the cluster using the principal's data or context, such as coercing AWS Backup into reading the cluster and producing a full cluster backup on their behalf.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Privilege Escalation](../threat-impacts.md#privilege-escalation)

**Mitigations:** [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)

**Frameworks:** MITRE ATT&CK [T1213.006](https://attack.mitre.org/techniques/T1213/006/), [T1548.005](https://attack.mitre.org/techniques/T1548/005/)

**Scoping** (for [Cross-service confused deputy problem through resource-based policies](../potential-threats.md#cross-service-confused-deputy-problem-through-resource-based-policies)): The resource-based policy on your cluster can grant access to an AWS service principal such as `backup.amazonaws.com`. An untrusted IAM principal who can configure AWS Backup may then have it produce a full backup of your cluster on their behalf.

**Request parameters**

| Operation | Parameters |
|---|---|
| `backup:CreateBackupSelection` | `$.BackupSelection.IamRoleArn`, `$.BackupSelection.Resources[*]` |
| `backup:StartBackupJob` | `$.IamRoleArn`, `$.ResourceArn` |
| `dsql:CreateCluster` | `$.policy` |
| `dsql:PutClusterPolicy` | `$.identifier`, `$.policy` |

### Threat.DSQL.Stream.1

IAM principal, with IAM permissions to create or modify an Aurora DSQL stream, can configure the stream without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Aurora DSQL stream can omit the tagging information your governance requires. A stream missing those tags stays outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateStream` | `$.clusterIdentifier`, `$.tags{key}` |
| `dsql:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `dsql:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.DSQL.Stream.2

Untrusted IAM principal, that has been granted access to my Aurora DSQL stream's CDC service role through its trust policy, can assume my CDC service role and act with its permissions.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1550.001](https://attack.mitre.org/techniques/T1550/001/), [T1580](https://attack.mitre.org/techniques/T1580/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Aurora DSQL assumes a CDC service role that you create (designated by `$.targetDefinition.kinesis.roleArn`) to deliver your stream's change records. If that role's trust policy grants unrestricted assumption, an identity outside your organization could assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `dsql:CreateStream` | `$.targetDefinition.kinesis.roleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

---

## Mitigations

Each scoping statement below names the capability the mitigation goal governs on the entity and the objective of governing it. Each mitigation's implementations describe how to govern or monitor that capability.

### Mitigation.DSQL.1

Aurora DSQL service API calls on a governed VPC network should only be made against trusted resources or AWS service-owned resources

**Goal:** [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)

**Addresses:** [Threat.DSQL.2](#threatdsql2)

**Implemented by:** [Implementation.DSQL.1](#implementationdsql1)

**Scoping** (for [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)): Amazon Aurora DSQL API calls from your VPCs can target resources outside your organization. Restricting those calls to resources in your organization enforces the resource perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.DSQL.2

Aurora DSQL service API calls on a governed VPC network should only be made by trusted identities or AWS services

**Goal:** [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)

**Addresses:** [Threat.DSQL.1](#threatdsql1)

**Implemented by:** [Implementation.DSQL.2](#implementationdsql2)

**Scoping** (for [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)): Amazon Aurora DSQL API calls from your VPCs can be made by principals outside your organization. Restricting those calls to principals in your organization enforces the identity perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.DSQL.Cluster.1

An Aurora DSQL cluster should be accessed only from expected networks

**Goal:** [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)

**Addresses:** [Threat.DSQL.Cluster.8](#threatdsqlcluster8)

**Implemented by:** [Implementation.DSQL.Cluster.1](#implementationdsqlcluster1), [Implementation.DSQL.Cluster.2](#implementationdsqlcluster2), [Implementation.DSQL.Cluster.3](#implementationdsqlcluster3)

**Scoping** (for [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)): Your Aurora DSQL cluster is a database resource whose endpoint is reachable from the public internet by default. A network perimeter helps you confirm that your cluster can be accessed only from expected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.policy` |
| `dsql:PutClusterPolicy` | `$.identifier`, `$.policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetClusterPolicy` | `$.identifier` |

### Mitigation.DSQL.Cluster.2

IAM principals should be restricted to accessing only trusted Aurora DSQL cluster resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.DSQL.Cluster.7](#threatdsqlcluster7)

**Implemented by:** [Implementation.DSQL.Cluster.4](#implementationdsqlcluster4)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): Aurora DSQL does not currently support cross-account access through resource-based policies, so your principals can access only clusters in their own account. Restricting requests to clusters in your organization is a defense-in-depth measure.

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

### Mitigation.DSQL.Cluster.3

An Aurora DSQL cluster should be protected by an AWS Backup backup plan

**Goal:** [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup)

**Addresses:** [Threat.DSQL.Cluster.1](#threatdsqlcluster1), [Threat.DSQL.Cluster.11](#threatdsqlcluster11)

**Implemented by:** [Implementation.DSQL.Cluster.5](#implementationdsqlcluster5), [Implementation.DSQL.Cluster.6](#implementationdsqlcluster6), [Implementation.DSQL.Cluster.7](#implementationdsqlcluster7), [ManagedControl.DSQL.Cluster.1](#managedcontroldsqlcluster1)

**Scoping** (for [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup)): An Aurora DSQL cluster is stateful and contains all of your user and system tables. AWS Backup supports Aurora DSQL clusters as assignable backup targets and creates full-cluster recovery points.

**Configure via**

| Operation | Parameters |
|---|---|
| `backup:CreateBackupPlan` | `$.BackupPlan.BackupPlanName`, `$.BackupPlan.Rules[*].CopyActions[*].DestinationBackupVaultArn`, `$.BackupPlan.Rules[*].Lifecycle.DeleteAfterDays`, `$.BackupPlan.Rules[*].ScheduleExpression`, `$.BackupPlan.Rules[*].TargetBackupVaultName` |
| `backup:CreateBackupSelection` | `$.BackupPlanId`, `$.BackupSelection.Conditions.StringEquals[*].ConditionKey`, `$.BackupSelection.Conditions.StringEquals[*].ConditionValue`, `$.BackupSelection.IamRoleArn`, `$.BackupSelection.ListOfTags[*].ConditionKey`, `$.BackupSelection.ListOfTags[*].ConditionValue`, `$.BackupSelection.Resources[*]` |
| `backup:UpdateRegionSettings` | `$.ResourceTypeOptInPreference{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `backup:DescribeRegionSettings` | *(any parameter)* |
| `backup:GetBackupPlan` | `$.BackupPlanId` |
| `backup:GetBackupSelection` | `$.BackupPlanId`, `$.SelectionId` |
| `backup:ListBackupPlans` | *(any parameter)* |
| `backup:ListBackupSelections` | `$.BackupPlanId` |
| `backup:ListProtectedResources` | *(any parameter)* |
| `backup:ListRecoveryPointsByResource` | `$.ResourceArn` |

### Mitigation.DSQL.Cluster.4

An Aurora DSQL cluster should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.DSQL.Cluster.4](#threatdsqlcluster4), [Threat.DSQL.Cluster.6](#threatdsqlcluster6), [Threat.DSQL.Cluster.9](#threatdsqlcluster9), [Threat.DSQL.Cluster.12](#threatdsqlcluster12)

**Implemented by:** [Implementation.DSQL.Cluster.8](#implementationdsqlcluster8), [Implementation.DSQL.Cluster.9](#implementationdsqlcluster9), [Implementation.DSQL.Cluster.10](#implementationdsqlcluster10), [ManagedControl.DSQL.Cluster.2](#managedcontroldsqlcluster2)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): Aurora DSQL encrypts your cluster data at rest under an AWS KMS key. A cluster supports a customer managed key in your own account as an alternative to the default AWS owned key.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.kmsEncryptionKey` |
| `dsql:UpdateCluster` | `$.identifier`, `$.kmsEncryptionKey` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetCluster` | `$.identifier` |

### Mitigation.DSQL.Cluster.5

An Aurora DSQL cluster should be replicated to at least one other AWS Region

**Goal:** [Cross-Region Replication](../mitigation-goals.md#cross-region-replication)

**Addresses:** [Threat.DSQL.Cluster.1](#threatdsqlcluster1), [Threat.DSQL.Cluster.3](#threatdsqlcluster3), [Threat.DSQL.Cluster.4](#threatdsqlcluster4)

**Implemented by:** [Implementation.DSQL.Cluster.11](#implementationdsqlcluster11), [Implementation.DSQL.Cluster.12](#implementationdsqlcluster12)

**Scoping** (for [Cross-Region Replication](../mitigation-goals.md#cross-region-replication)): An Aurora DSQL cluster stores your relational data. Aurora DSQL supports multi-Region clusters that keep a full copy of that data in a peered cluster in another Region.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.multiRegionProperties.clusters[*]`, `$.multiRegionProperties.witnessRegion` |
| `dsql:UpdateCluster` | `$.identifier`, `$.multiRegionProperties.clusters[*]`, `$.multiRegionProperties.witnessRegion` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetCluster` | `$.identifier` |

### Mitigation.DSQL.Cluster.6

An Aurora DSQL cluster should be accessed only by trusted identities or AWS services acting on behalf of trusted resources

**Goal:** [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)

**Addresses:** [Threat.DSQL.Cluster.10](#threatdsqlcluster10)

**Implemented by:** [Implementation.DSQL.Cluster.13](#implementationdsqlcluster13), [Implementation.DSQL.Cluster.14](#implementationdsqlcluster14), [Implementation.DSQL.Cluster.15](#implementationdsqlcluster15), [Implementation.DSQL.Cluster.16](#implementationdsqlcluster16)

**Scoping** (for [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)): Aurora DSQL supports a resource-based policy that specifies who can access the cluster and what actions they can perform. The service does not currently support cross-account access through the resource-based policy, so restricting access to principals in your organization is a defense-in-depth measure.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.policy` |
| `dsql:DeleteClusterPolicy` | `$.identifier` |
| `dsql:PutClusterPolicy` | `$.identifier`, `$.policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetClusterPolicy` | `$.identifier` |

### Mitigation.DSQL.Cluster.7

An Aurora DSQL cluster should be accessed by an AWS service principal, such as the AWS Backup service principal, only when the service principal is acting on behalf of a trusted resource, AWS account, or organization

**Goal:** [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies)

**Addresses:** [Threat.DSQL.Cluster.13](#threatdsqlcluster13)

**Implemented by:** [Implementation.DSQL.Cluster.17](#implementationdsqlcluster17), [Implementation.DSQL.Cluster.18](#implementationdsqlcluster18), [Implementation.DSQL.Cluster.19](#implementationdsqlcluster19), [Implementation.DSQL.Cluster.20](#implementationdsqlcluster20)

**Scoping** (for [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies)): Your Aurora DSQL cluster supports a resource-based policy that can grant cluster actions such as `dsql:GetCluster` to the AWS Backup service principal, `backup.amazonaws.com`. Such a grant may raise the cross-service confused deputy problem, where the service calls Aurora DSQL on behalf of a resource outside your organization.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.policy` |
| `dsql:PutClusterPolicy` | `$.identifier`, `$.policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetClusterPolicy` | `$.identifier` |

### Mitigation.DSQL.Cluster.8

An Aurora DSQL cluster should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.DSQL.Cluster.5](#threatdsqlcluster5)

**Implemented by:** [Implementation.DSQL.Cluster.21](#implementationdsqlcluster21), [Implementation.DSQL.Cluster.22](#implementationdsqlcluster22), [Implementation.DSQL.Cluster.23](#implementationdsqlcluster23)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): An Aurora DSQL cluster is a durable, top-level resource that you can tag through its own ARN, either at creation using `$.tags{key}` or afterward with `TagResource`. Tag keys are free-form, so you can record ownership and purpose directly on each cluster.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.tags{key}` |
| `dsql:TagResource` | `$.resourceArn`, `$.tags{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetCluster` | `$.identifier` |
| `dsql:ListTagsForResource` | `$.resourceArn` |

### Mitigation.DSQL.Cluster.9

An Aurora DSQL cluster should be configured to disallow resource-based policy modifications that would remove all policy-management access for the cluster

**Goal:** [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention)

**Addresses:** [Threat.DSQL.Cluster.2](#threatdsqlcluster2)

**Implemented by:** [Implementation.DSQL.Cluster.2](#implementationdsqlcluster2), [Implementation.DSQL.Cluster.24](#implementationdsqlcluster24)

**Scoping** (for [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention)): An Aurora DSQL cluster carries its own resource-based policy that governs who can manage the cluster and connect to it. Replacing that policy with one that removes all policy-management access leaves the cluster unable to be managed.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.bypassPolicyLockoutSafetyCheck`, `$.policy` |
| `dsql:PutClusterPolicy` | `$.bypassPolicyLockoutSafetyCheck`, `$.identifier`, `$.policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetClusterPolicy` | `$.identifier` |

### Mitigation.DSQL.Cluster.10

An Aurora DSQL cluster should have deletion protection enabled

**Goal:** [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection)

**Addresses:** [Threat.DSQL.Cluster.1](#threatdsqlcluster1)

**Implemented by:** [Implementation.DSQL.Cluster.25](#implementationdsqlcluster25), [Implementation.DSQL.Cluster.26](#implementationdsqlcluster26), [Implementation.DSQL.Cluster.27](#implementationdsqlcluster27), [Implementation.DSQL.Cluster.28](#implementationdsqlcluster28), [Implementation.DSQL.Cluster.29](#implementationdsqlcluster29)

**Scoping** (for [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection)): An Aurora DSQL cluster has its own deletion protection setting, `$.deletionProtectionEnabled`, that you can turn on at creation or later. This setting helps protect the cluster from unintended deletion.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateCluster` | `$.deletionProtectionEnabled` |
| `dsql:UpdateCluster` | `$.deletionProtectionEnabled`, `$.identifier` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetCluster` | `$.identifier` |

### Mitigation.DSQL.Stream.1

An Aurora DSQL stream's service role should be assumed only by Aurora DSQL acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.DSQL.Stream.2](#threatdsqlstream2)

**Implemented by:** [Implementation.DSQL.Stream.1](#implementationdsqlstream1), [Implementation.DSQL.Stream.2](#implementationdsqlstream2), [Implementation.DSQL.Stream.3](#implementationdsqlstream3)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): Your Aurora DSQL stream's service role, passed in the `$.targetDefinition.kinesis.roleArn` request parameter, carries a trust policy that specifies who can assume it. Allowing only Aurora DSQL, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateStream` | `$.clusterIdentifier`, `$.targetDefinition.kinesis.roleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetStream` | `$.clusterIdentifier`, `$.streamIdentifier` |
| `dsql:ListStreams` | `$.clusterIdentifier` |

### Mitigation.DSQL.Stream.2

An Aurora DSQL stream should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.DSQL.Stream.1](#threatdsqlstream1)

**Implemented by:** [Implementation.DSQL.Stream.4](#implementationdsqlstream4), [Implementation.DSQL.Stream.5](#implementationdsqlstream5)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): Aurora DSQL CDC streams are independently taggable resources, each with its own ARN and tag set separate from the parent cluster. You can tag a stream at creation and manage its tags afterwards with `TagResource` and `UntagResource`.

**Configure via**

| Operation | Parameters |
|---|---|
| `dsql:CreateStream` | `$.clusterIdentifier`, `$.tags{key}` |
| `dsql:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `dsql:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `dsql:GetStream` | `$.clusterIdentifier`, `$.streamIdentifier` |
| `dsql:ListStreams` | `$.clusterIdentifier` |
| `dsql:ListTagsForResource` | `$.resourceArn` |

---

## Implementations

### Implementation.DSQL.1

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.DSQL.1](#mitigationdsql1)

**Scoping** (for [Mitigation.DSQL.1](#mitigationdsql1)): This implementation governs Aurora DSQL API calls through your VPC endpoint with the `aws:ResourceOrgID` condition key, exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, principals using that endpoint are restricted from accessing Aurora DSQL clusters outside your organization.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.DSQL.2

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.DSQL.2](#mitigationdsql2)

**Scoping** (for [Mitigation.DSQL.2](#mitigationdsql2)): This VPC endpoint policy restricts Aurora DSQL API calls with the `aws:PrincipalOrgID` condition key, while exempting AWS services that act on your behalf (`aws:PrincipalIsAWSService`). Requests crossing your governed VPC network are denied unless the caller belongs to your organization, helping establish an identity perimeter.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.DSQL.Cluster.1

*Resource policy — Preventive*

Attaches to `AWS::DSQL::Cluster` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:*` | `arn:${Partition}:dsql:${Region}:${Account}:cluster/${Identifier}` | `aws:PrincipalIsAWSService`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `dsql:*` | `arn:${Partition}:dsql:${Region}:${Account}:cluster/${Identifier}` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1)

**Scoping** (for [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1)): This implementation restricts Amazon Aurora DSQL API calls against your cluster with the `aws:SourceVpc` (or `aws:SourceVpce`), `aws:SourceIp`, and `aws:RequestedRegion` condition keys. It denies requests from unexpected networks while exempting AWS services that access your cluster on your behalf (`aws:PrincipalIsAWSService`, `aws:ViaAWSService`) and tagged identities (`aws:PrincipalTag/dp:exclude:network`).

**Reference:** To restrict access to expected networks, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys. These scope access to a specific VPC or VPC endpoint. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalIsAWSService` to exempt AWS services that access your resources with their service principals. Use `aws:PrincipalTag/tag-key` to exempt principals that must legitimately operate from outside your expected networks. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)

### Implementation.DSQL.Cluster.2

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DSQL::Cluster`: `$.PolicyDocument`

**Implements:** [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1), [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9)

**Scoping** (for [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1)): This implementation validates `$.PolicyDocument` at create and update time, requiring network restrictions built on the `aws:SourceVpc`, `aws:SourceVpce`, and `aws:SourceIp` condition keys. A cluster whose policy would permit `dsql:DbConnect` and `dsql:DbConnectAdmin` from unexpected networks is not provisioned, which helps establish a network perimeter.

**Scoping** (for [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9)): This implementation rejects a cluster resource-based policy in `$.PolicyDocument` that would remove all access to `dsql:PutClusterPolicy` and `dsql:DeleteClusterPolicy`. Blocking such a policy at creation and on later updates helps you retain the ability to update or remove that policy.

### Implementation.DSQL.Cluster.3

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:*` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `dsql:*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1)

**Scoping** (for [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1)): This implementation governs Amazon Aurora DSQL API calls against your clusters with the `aws:SourceVpc`, `aws:SourceVpce`, and `aws:SourceIp` condition keys. As a result, principals on unexpected networks are restricted from accessing your clusters, while exempting AWS services that access them on your behalf (`aws:PrincipalIsAWSService`, `aws:ViaAWSService`).

**Reference:** To restrict access to expected networks, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys. These scope access to a specific VPC or VPC endpoint. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalIsAWSService` to exempt AWS services that access your resources with their service principals. Use `aws:PrincipalTag/tag-key` to exempt principals that must legitimately operate from outside your expected networks. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [RCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/network_perimeter_sourcevpc_rcp.json)

### Implementation.DSQL.Cluster.4

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2)

**Scoping** (for [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern Aurora DSQL API calls your identities make. As a result, your identities are restricted from accessing Aurora DSQL clusters owned outside your organization.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.DSQL.Cluster.5

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `backup:DeleteBackupPlan` — `$.BackupPlanId`
- `backup:DeleteBackupSelection` — `$.BackupPlanId`, `$.SelectionId`
- `backup:DeleteRecoveryPoint` — `$.BackupVaultName`, `$.RecoveryPointArn`
- `backup:StopBackupJob` — `$.BackupJobId`
- `backup:UpdateBackupPlan` — `$.BackupPlan.Rules[*].Lifecycle.DeleteAfterDays`, `$.BackupPlan.Rules[*].ScheduleExpression`, `$.BackupPlan.Rules[*].TargetBackupVaultName`, `$.BackupPlanId`
- `backup:UpdateRecoveryPointLifecycle` — `$.BackupVaultName`, `$.Lifecycle.DeleteAfterDays`, `$.RecoveryPointArn`
- `backup:UpdateRegionSettings` — `$.ResourceTypeOptInPreference{key}`
- `dsql:TagResource` — `$.resourceArn`, `$.tags{key}`
- `dsql:UntagResource` — `$.resourceArn`, `$.tagKeys[*]`

**Implements:** [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3)

**Scoping** (for [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3)): This implementation records API calls that delete or modify your backup plans, selections, recovery points, in-flight jobs, Region opt-in settings, and cluster protection tags. These events help you detect changes that remove or weaken the AWS Backup recovery path for your cluster.

### Implementation.DSQL.Cluster.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dsql:ListClusters`; check `$.clusters[*].arn`, `$.clusters[*].identifier`
2. Call `dsql:ListTagsForResource` with `$.resourceArn`; check `$.tags{key}`
3. Call `backup:ListBackupPlans`; check `$.BackupPlansList[*].BackupPlanId`
4. Call `backup:ListBackupSelections` with `$.BackupPlanId`; check `$.BackupSelectionsList[*].SelectionId`
5. Call `backup:GetBackupSelection` with `$.BackupPlanId`, `$.SelectionId`; check `$.BackupSelection.Resources`, `$.BackupSelection.NotResources`, `$.BackupSelection.ListOfTags`, `$.BackupSelection.Conditions`

**Implements:** [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3)

**Scoping** (for [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3)): This periodic check compares each cluster ARN in `$.clusters[*].arn` with the resource assignments of your backup plans, retrieved through `ListBackupPlans`, `ListBackupSelections`, and `GetBackupSelection`. A cluster is covered when at least one backup selection matches it, either its ARN matches a `$.BackupSelection.Resources` pattern and no `$.BackupSelection.NotResources` pattern, or its tags satisfy the selection's `$.BackupSelection.ListOfTags` or `$.BackupSelection.Conditions`. Flagging a cluster no selection covers helps address the mitigation by surfacing databases that have no centrally governed recovery points.

### Implementation.DSQL.Cluster.7

*Declarative policy — Preventive*

Governs the `BACKUP_POLICY` declarative policy attributes below.

- `$.plans.AuroraDsqlBackupPlan.selections.resources.AuroraDsqlClusters.not_resource_types`
- `$.plans.AuroraDsqlBackupPlan.selections.resources.AuroraDsqlClusters.resource_types`

**Implements:** [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3)

**Scoping** (for [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3)): This implementation assigns the Aurora DSQL cluster resource-type pattern to `$.plans.AuroraDsqlBackupPlan.selections.resources.AuroraDsqlClusters.resource_types`, enrolling your clusters in a centrally governed backup plan. It also governs `$.plans.AuroraDsqlBackupPlan.selections.resources.AuroraDsqlClusters.not_resource_types`, so a broader selection does not exclude those clusters from scheduled recovery points.

### Implementation.DSQL.Cluster.8

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DSQL::Cluster`: `$.KmsEncryptionKey`

**Implements:** [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)

**Scoping** (for [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)): This implementation checks that `$.KmsEncryptionKey` names a customer managed key, so a cluster cannot be created on the default AWS owned key. Encryption at rest then uses a key whose policy, rotation, and deletion you control.

### Implementation.DSQL.Cluster.9

*CloudTrail data event — Detective*

Monitors the API calls below in CloudTrail.

- `dsql:DbConnect`
- `dsql:DbConnectAdmin`

**Implements:** [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)

**Scoping** (for [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)): When enabled for your cluster, CloudTrail data events record `DbConnect` and `DbConnectAdmin` connection attempts along with the calling identity. This complements the customer managed key set through `$.kmsEncryptionKey` by giving you a reviewable record for identifying unexpected access to the encrypted data.

**Reference:** Enforcing a customer managed key governs which key protects the data. It does not record which permitted principals read that data. The data events this guide monitors record those reads. CloudTrail does not log data events by default, and they are high volume. To scope log volume and cost, add advanced event selectors to the trail or event data store. Filter `resources.ARN` to the governed resources, using `Equals` for exact ARNs or `StartsWith` for a naming prefix. Filter `eventName` to the operations this guide monitors. A `readOnly` filter selects by CloudTrail's read or write classification, not by this guide's operation set, so do not rely on it alone. A trail or event data store accepts at most 500 values across all advanced event selector conditions.
- [Filtering data events by using advanced event selectors](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/filtering-data-events.html)
- [Logging data events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)
- [Quotas in AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/WhatIsCloudTrail-Limits.html)

### Implementation.DSQL.Cluster.10

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DSQL::Cluster`: `$.configuration.EncryptionDetails.EncryptionType`, `$.configuration.EncryptionDetails.KmsKeyArn`

**Implements:** [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)

**Scoping** (for [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)): This implementation reads `$.configuration.EncryptionDetails.EncryptionType` and resolves `$.configuration.EncryptionDetails.KmsKeyArn` to confirm `$.KeyMetadata.KeyManager` is `CUSTOMER`. It flags Aurora DSQL clusters left on the AWS owned key, where you do not control the key policy, rotation, or deletion of the key protecting your data.

### Implementation.DSQL.Cluster.11

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DSQL::Cluster`: `$.MultiRegionProperties.Clusters[*]`, `$.MultiRegionProperties.WitnessRegion`

**Implements:** [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5)

**Scoping** (for [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5)): This implementation validates that `$.MultiRegionProperties.Clusters[*]` names at least one peer cluster in another Region and that `$.MultiRegionProperties.WitnessRegion` is set. Rejecting a cluster template without these values at create time helps address the mitigation by preventing single-Region clusters that cannot survive the loss of a Region.

### Implementation.DSQL.Cluster.12

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dsql:ListClusters`; check `$.clusters[*].identifier`, `$.clusters[*].arn`
2. Call `dsql:GetCluster` with `$.identifier`; check `$.multiRegionProperties.clusters[*]`, `$.multiRegionProperties.witnessRegion`, `$.arn`, `$.status`

**Implements:** [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5)

**Scoping** (for [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5)): This periodic check flags any cluster whose `$.multiRegionProperties.clusters[*]` names no peer cluster in another Region. Flagging those clusters helps address the mitigation by surfacing databases that keep only one Regional copy of your data.

### Implementation.DSQL.Cluster.13

*Resource policy — Preventive*

Attaches to `AWS::DSQL::Cluster` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:*` | `arn:${Partition}:dsql:${Region}:${Account}:cluster/${Identifier}` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `dsql:*` | `arn:${Partition}:dsql:${Region}:${Account}:cluster/${Identifier}` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)

**Scoping** (for [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)): This implementation governs Aurora DSQL API calls against your cluster with `aws:PrincipalOrgID`, while exempting AWS services that access it on your behalf (`aws:PrincipalIsAWSService`). As a result, identities outside your organization are restricted from accessing your cluster, and those service requests are scoped by `aws:SourceOrgID`.

**Reference:** Confirm that resource-based policies grant access only to trusted identities and to services acting on your behalf.

As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`.

To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys.

Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.DSQL.Cluster.14

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DSQL::Cluster`: `$.PolicyDocument`

**Implements:** [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)

**Scoping** (for [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)): This implementation flags `dsql:DbConnect` and `dsql:DbConnectAdmin` grants in your cluster resource-based policy (`$.PolicyDocument`) to principals beyond your reference policy's allowlist. Reviewing those grants before your cluster is created or updated helps you establish an identity perimeter.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks on your cluster resource-based policy (`$.PolicyDocument`). Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. `ValidatePolicy` reports the `RESTRICT_ACCESS_TO_SERVICE_PRINCIPAL` finding when a grant to AWS services acting on your behalf does not specify a source. Custom policy checks (`CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to analyze the access your policy can grant. `CheckNoNewAccess` compares your policy with a reference policy that you maintain (`$.existingPolicyDocument`). The reference policy allowlists the principals that may be granted access to your cluster. The check fails when your policy might grant access beyond that allowlist. `CheckAccessNotGranted` fails when your policy might grant `dsql:DbConnect` or `dsql:DbConnectAdmin`. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM JSON policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DSQL.Cluster.15

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DSQL::Cluster`: `$.configuration.PolicyDocument`

**Implements:** [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)

**Scoping** (for [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)): This implementation flags grants in your cluster's resource-based policy (`$.configuration.PolicyDocument`) to identities outside your reference policy or to AWS services without source condition keys. Flagging these grants helps you detect access to your cluster that extends beyond trusted identities.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant new access beyond a reference policy you maintain or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DSQL.Cluster.16

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:*` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `dsql:*` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)

**Scoping** (for [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6)): This implementation governs Aurora DSQL API calls against your clusters with `aws:PrincipalOrgID` and `aws:SourceOrgID` condition keys, exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, identities outside your organization are restricted from accessing your clusters.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.DSQL.Cluster.17

*Resource policy — Preventive*

Attaches to `AWS::DSQL::Cluster` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:GetCluster`, `dsql:ListTagsForResource`, `dsql:StartBackupJob` | `arn:${Partition}:dsql:${Region}:${Account}:cluster/${Identifier}` | `aws:SourceAccount`, `aws:SourceArn` |

**Implements:** [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)

**Scoping** (for [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)): A resource-based policy on your Aurora DSQL cluster can grant access to AWS services acting on your behalf, such as AWS Backup. The `aws:SourceArn` and `aws:SourceAccount` condition keys restrict that access to requests made on behalf of a resource in your account.

**Reference:** To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.DSQL.Cluster.18

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DSQL::Cluster`: `$.PolicyDocument`

**Implements:** [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)

**Scoping** (for [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)): This implementation validates the resource-based policy your template declares in `$.PolicyDocument`, helping address the cross-service confused deputy problem for your cluster. When a grant to an AWS service principal omits the `aws:SourceArn`, `aws:SourceAccount`, or `aws:SourceOrgID` condition keys, the hook fails the stack operation.

**Reference:** This implementation recommends using IAM Access Analyzer basic policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.DSQL.Cluster.19

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DSQL::Cluster`: `$.configuration.PolicyDocument`

**Implements:** [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)

**Scoping** (for [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)): This rule reads the resource-based policy on your cluster from `$.configuration.PolicyDocument`, then flags grants that omit an `aws:SourceArn`, `aws:SourceAccount`, or `aws:SourceOrgID` condition key. Flagging those grants helps you detect access by AWS services acting on your behalf that lacks a source restriction.

**Reference:** This implementation recommends using IAM Access Analyzer basic policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.DSQL.Cluster.20

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:*` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)

**Scoping** (for [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7)): This implementation restricts Amazon Aurora DSQL API calls against your clusters with the `aws:SourceOrgID`, `aws:SourceAccount`, and `aws:PrincipalIsAWSService` condition keys. Requests that do not satisfy those conditions are denied, so AWS services cannot access your cluster on behalf of resources outside your organization.

**Reference:** To restrict service access to your resources at scale, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. These keys are included in the request context only for requests by AWS service principals, so apply them when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: enforce controls on AWS services that use service principals](https://github.com/aws-samples/resource-control-policy-examples/blob/main/Restrict-resource-access-patterns/Enforce-controls-on-AWS-services-that-use-service-principals-to-access-your-resources.json)

### Implementation.DSQL.Cluster.21

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DSQL::Cluster`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8)

**Scoping** (for [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8)): This implementation validates `$.Tags[*].Key` and `$.Tags[*].Value` before a cluster is provisioned, so clusters that omit your required owner and purpose tags are not created. Enforcing the tags at provisioning time helps keep ownership and business function attributable for every cluster.

### Implementation.DSQL.Cluster.22

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DSQL::Cluster`: `$.configuration.Tags[*].Key`, `$.configuration.Tags[*].Value`

**Implements:** [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8)

**Scoping** (for [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8)): This implementation reads `$.configuration.Tags[*].Key` and `$.configuration.Tags[*].Value` on each cluster configuration item. Flagging clusters missing your required owner and purpose tag keys helps you keep each cluster attributable to an accountable team and business function.

### Implementation.DSQL.Cluster.23

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:CreateCluster` | `*` | `aws:RequestTag/owner` |
| `dsql:CreateCluster` | `*` | `aws:RequestTag/purpose` |
| `dsql:CreateCluster` | `*` | `aws:TagKeys` |
| `dsql:TagResource` | `arn:aws:dsql:*:*:cluster/*` | `aws:RequestTag/owner` |
| `dsql:TagResource` | `arn:aws:dsql:*:*:cluster/*` | `aws:RequestTag/purpose` |
| `dsql:TagResource` | `arn:aws:dsql:*:*:cluster/*` | `aws:TagKeys` |
| `dsql:UntagResource` | `arn:aws:dsql:*:*:cluster/*` | `aws:TagKeys` |

**Implements:** [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8)

**Scoping** (for [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8)): This implementation restricts `dsql:CreateCluster` and `dsql:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys. Requests that omit either tag key from the cluster tag map `$.tags{key}` are denied, so successful calls record ownership and purpose. This implementation also restricts `dsql:UntagResource` with the `aws:TagKeys` condition key, so requests that remove either required tag key are denied.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.DSQL.Cluster.24

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `dsql:CreateCluster` — `$.bypassPolicyLockoutSafetyCheck` = `true`, `$.policy`
- `dsql:PutClusterPolicy` — `$.bypassPolicyLockoutSafetyCheck` = `true`, `$.identifier`, `$.policy`

**Implements:** [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9)

**Scoping** (for [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9)): This implementation flags `PutClusterPolicy` and `CreateCluster` calls that attach a `$.policy` with `$.bypassPolicyLockoutSafetyCheck` set to true. Recording these events together with the calling identity helps you detect and respond to an attempt to leave a cluster unmanageable.

### Implementation.DSQL.Cluster.25

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::DSQL::Cluster`: `$.DeletionProtectionEnabled`

**Implements:** [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)

**Scoping** (for [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)): This implementation validates that `$.DeletionProtectionEnabled`, the cluster's deletion protection setting, is enabled when a cluster is created or updated. With the setting enabled from creation, delete requests against the cluster are rejected until you disable it.

### Implementation.DSQL.Cluster.26

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `dsql:UpdateCluster` — `$.deletionProtectionEnabled` = `false`

**Implements:** [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)

**Scoping** (for [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)): Monitoring `UpdateCluster` records calls that turn off deletion protection on your cluster by setting `$.deletionProtectionEnabled` to false. Because deletion protection must be disabled before the cluster can be deleted, this event gives you an early signal to investigate.

### Implementation.DSQL.Cluster.27

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::DSQL::Cluster`: `$.configuration.DeletionProtectionEnabled`

**Implements:** [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)

**Scoping** (for [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)): This implementation reads `$.configuration.DeletionProtectionEnabled` on each recorded Aurora DSQL cluster. It flags any cluster where the flag is not enabled, so you can find clusters that `DeleteCluster` would accept without an explicit opt-out first.

### Implementation.DSQL.Cluster.28

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:DeleteCluster`, `dsql:UpdateCluster` | `arn:aws:dsql:*:*:cluster/*` | `aws:PrincipalArn` (alternatives: `aws:PrincipalTag/${TagKey}`), `aws:PrincipalIsAWSService` |

**Implements:** [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)

**Scoping** (for [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)): This implementation restricts `dsql:DeleteCluster` and `dsql:UpdateCluster` with the `aws:PrincipalArn` condition key, exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). Requests that do not satisfy it are denied, so only the principals you allowlist can delete a cluster or turn off deletion protection (`$.deletionProtectionEnabled`).

**Reference:** To restrict access to select principals, use the `aws:PrincipalArn` or `aws:PrincipalTag/tag-key` condition keys. Use `aws:PrincipalArn` to specify the Amazon Resource Names (ARNs) of the principals. Those ARNs can follow a naming convention or an IAM path that scopes the restriction to a set of principals without enumerating each one. Use `aws:PrincipalTag/tag-key` to identify those principals by a tag attached to them. Choose the condition key that matches your IAM policy management strategy. Apply the restriction only when `aws:PrincipalIsAWSService` is `false`, so that AWS services accessing your resources using their service principals continue to operate as expected.

### Implementation.DSQL.Cluster.29

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:DeleteCluster`, `dsql:UpdateCluster` | `arn:aws:dsql:*:*:cluster/*` | `aws:PrincipalArn` (alternatives: `aws:PrincipalTag/${TagKey}`) |

**Implements:** [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)

**Scoping** (for [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10)): This implementation restricts `dsql:DeleteCluster` and `dsql:UpdateCluster`, the action that turns off the `$.deletionProtectionEnabled` setting, with the `aws:PrincipalArn` condition key. Requests that do not satisfy that condition are denied, so only your trusted administrative role can delete a cluster or remove its deletion protection.

**Reference:** To restrict access to select principals, use the `aws:PrincipalArn` or `aws:PrincipalTag/tag-key` condition keys. Use `aws:PrincipalArn` to specify the Amazon Resource Names (ARNs) of the principals. Those ARNs can follow a naming convention or an IAM path that scopes the restriction to a set of principals without enumerating each one. Use `aws:PrincipalTag/tag-key` to identify those principals by a tag attached to them. Choose the condition key that matches your IAM policy management strategy.

### Implementation.DSQL.Stream.1

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dsql:ListClusters`; check `$.clusters[*].identifier`
2. Call `dsql:ListStreams` with `$.clusterIdentifier`; check `$.streams[*].clusterIdentifier`, `$.streams[*].streamIdentifier`
3. Call `dsql:GetStream` with `$.clusterIdentifier`, `$.streamIdentifier`; check `$.targetDefinition.kinesis.roleArn`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result` = `PASS`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result` = `PASS`
7. Call `accessanalyzer:ValidatePolicy` with `$.policyDocument`, `$.policyType` = `RESOURCE_POLICY`, `$.validatePolicyResourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.findings[*].issueCode` = `RESTRICT_ACCESS_TO_SERVICE_PRINCIPAL`

**Implements:** [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1)

**Scoping** (for [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1)): This implementation reads `$.targetDefinition.kinesis.roleArn`, your CDC stream's service role, then inspects `$.Role.AssumeRolePolicyDocument`, that role's trust policy. Flagging policies that allow public role assumption, or that omit the `aws:SourceArn`, `aws:SourceAccount`, or `aws:SourceOrgID` condition keys, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.DSQL.Stream.2

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `dsql.amazonaws.com` | `sts:AssumeRole` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1)

**Scoping** (for [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1)): This implementation guidance recommends restricting `sts:AssumeRole` on the CDC service role you pass in `$.targetDefinition.kinesis.roleArn` to `dsql.amazonaws.com` with the `aws:SourceArn` and `aws:SourceAccount` condition keys in the role's trust policy. They help prevent Aurora DSQL from assuming the role on behalf of streams outside the ARNs and account you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.DSQL.Stream.3

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1)

**Scoping** (for [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1)): This implementation restricts `sts:AssumeRole` with the `aws:PrincipalOrgID` and `aws:SourceOrgID` condition keys, while exempting AWS services that assume your role on your behalf (`aws:PrincipalIsAWSService`). Requests that do not satisfy these conditions are denied, so identities outside your organization cannot assume your CDC stream service role (`$.targetDefinition.kinesis.roleArn`).

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.DSQL.Stream.4

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `dsql:ListClusters`; check `$.clusters[*].identifier`
2. Call `dsql:ListStreams` with `$.clusterIdentifier`; check `$.streams[*].arn`
3. Call `dsql:ListTagsForResource` with `$.resourceArn`; check `$.tags{key}`

**Implements:** [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2)

**Scoping** (for [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2)): This periodic check enumerates your change data capture streams by `$.streams[*].arn` and reads the `$.tags{key}` map for each one. Flagging streams that lack your owner and purpose tag keys helps you identify who is accountable for a stream and what it supports.

### Implementation.DSQL.Stream.5

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `dsql:CreateStream` | `*` | `aws:RequestTag/owner` |
| `dsql:CreateStream` | `*` | `aws:RequestTag/purpose` |
| `dsql:CreateStream` | `*` | `aws:TagKeys` |
| `dsql:TagResource` | `arn:aws:dsql:*:*:cluster/*/stream/*` | `aws:RequestTag/owner` |
| `dsql:TagResource` | `arn:aws:dsql:*:*:cluster/*/stream/*` | `aws:RequestTag/purpose` |
| `dsql:TagResource` | `arn:aws:dsql:*:*:cluster/*/stream/*` | `aws:TagKeys` |
| `dsql:UntagResource` | `arn:aws:dsql:*:*:cluster/*/stream/*` | `aws:TagKeys` |

**Implements:** [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2)

**Scoping** (for [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2)): This implementation denies `dsql:CreateStream` and stream-scoped `dsql:TagResource` requests that omit the exactly-cased `owner` or `purpose` tag key. Checking `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` on both actions helps keep each stream attributable to an owner and a business purpose. Denying stream-scoped `dsql:UntagResource` requests whose `aws:TagKeys` include `owner` or `purpose` helps keep the required keys from being removed.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### ManagedControl.DSQL.Cluster.1

*AWS Config rule — Detective*

AWS managed control from General controls (service-agnostic — no governed resource).

**Title:** Checks if a recovery point expires no earlier than after the specified period

**Aliases:** `BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK`

**Identifier:** `BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK`

**ARN:** `arn:aws:controlcatalog:::control/m7a5gbdf08wg2o0en010mkng`

Checks if a recovery point expires no earlier than after the specified period. The rule is NON_COMPLIANT if the recovery point has a retention point that is less than the required retention period.

**Implements:** [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3)

### ManagedControl.DSQL.Cluster.2

*AWS Config rule — Detective*

AWS managed control from General controls (service-agnostic — no governed resource).

**Title:** Checks if a recovery point is encrypted

**Aliases:** `CONFIG.BACKUP.DT.1`

**Identifier:** `BACKUP_RECOVERY_POINT_ENCRYPTED`

**ARN:** `arn:aws:controlcatalog:::control/f2z4q47tqvr1h0ovoy5klwp7o`

Checks if a recovery point is encrypted. The rule is NON_COMPLIANT if the recovery point is not encrypted.

**Implements:** [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4)

---

## Appendix

### Resources

#### Cluster

*DATA_STORE*

An Aurora DSQL cluster is the serverless, distributed relational database resource that holds your schemas, tables, and data. It is PostgreSQL compatible, so you connect with standard drivers and tools and write familiar SQL. The cluster manages its own infrastructure. Compute, transaction commit, and storage scale independently with your workload, with no instances to size or patch.

- IAM resource ARN: `arn:${Partition}:dsql:${Region}:${Account}:cluster/${Identifier}`
- CloudFormation: `AWS::DSQL::Cluster`
- AWS Config: `AWS::DSQL::Cluster`
- 12 operations, 13 threats, 10 mitigations

#### Stream

*INTEGRATION*

An Aurora DSQL stream provides change data capture for a single cluster. It reads committed transactions and converts each net row-level change into a structured JSON record. It delivers those records to an Amazon Kinesis data stream that you create and operate.

- IAM resource ARN: `arn:${Partition}:dsql:${Region}:${Account}:cluster/${ClusterId}/stream/${StreamId}`
- 7 operations, 2 threats, 2 mitigations

### Operations

| Operation | Analyses | Resources | Mitigations |
|---|---|---|---|
| `CreateCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1), [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2), [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4), [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5), [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6), [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7), [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8), [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9), [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10) |
| `CreateStream` | DOCUMENTATION | [Stream](#stream) | [Mitigation.DSQL.1](#mitigationdsql1), [Mitigation.DSQL.2](#mitigationdsql2), [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1), [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2) |
| `DeleteCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.DSQL.1](#mitigationdsql1), [Mitigation.DSQL.2](#mitigationdsql2), [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1), [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2), [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3), [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5), [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6), [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10) |
| `DeleteClusterPolicy` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.DSQL.2](#mitigationdsql2), [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1), [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6) |
| `DeleteStream` | DOCUMENTATION | [Stream](#stream) |  |
| `GetCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4), [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5), [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8), [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10) |
| `GetClusterPolicy` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1), [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6), [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7), [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9) |
| `GetStream` | DOCUMENTATION | [Stream](#stream) | [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1), [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2) |
| `GetVpcEndpointServiceName` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `ListClusters` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `ListStreams` | DOCUMENTATION | [Stream](#stream) | [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1), [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2) |
| `ListTagsForResource` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster), [Stream](#stream) | [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8), [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2) |
| `PutClusterPolicy` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.DSQL.1](#mitigationdsql1), [Mitigation.DSQL.2](#mitigationdsql2), [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1), [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2), [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6), [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7), [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9) |
| `TagResource` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster), [Stream](#stream) | [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8), [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2) |
| `UntagResource` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster), [Stream](#stream) | [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8), [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2) |
| `UpdateCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.DSQL.1](#mitigationdsql1), [Mitigation.DSQL.2](#mitigationdsql2), [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1), [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2), [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4), [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5), [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6), [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10) |

Mitigations attach to an operation directly (the operation appears in the mitigation's request parameters) or through a threat whose request parameters include the operation.

### Goal Coverage (13 of 38)

| Mitigation Goal | Covered | Mitigations |
|---|---|---|
| [AI Services Opt-Out Policy](../mitigation-goals.md#ai-services-opt-out-policy) | — |  |
| [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup) | yes | [Mitigation.DSQL.Cluster.3](#mitigationdsqlcluster3) |
| [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native) | — |  |
| [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery) | — |  |
| [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2) | — |  |
| [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest) | — |  |
| [Cross-Region Replication](../mitigation-goals.md#cross-region-replication) | yes | [Mitigation.DSQL.Cluster.5](#mitigationdsqlcluster5) |
| [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies) | yes | [Mitigation.DSQL.Cluster.7](#mitigationdsqlcluster7) |
| [Customer Code Execution - Govern Code Content](../mitigation-goals.md#customer-code-execution---govern-code-content) | — |  |
| [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage) | — |  |
| [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection) | yes | [Mitigation.DSQL.Cluster.10](#mitigationdsqlcluster10) |
| [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics) | — |  |
| [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics) | — |  |
| [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy) | — |  |
| [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature) | — |  |
| [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication) | — |  |
| [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network) | yes | [Mitigation.DSQL.2](#mitigationdsql2) |
| [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources) | yes | [Mitigation.DSQL.Cluster.6](#mitigationdsqlcluster6) |
| [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption) | yes | [Mitigation.DSQL.Cluster.4](#mitigationdsqlcluster4) |
| [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment) | — |  |
| [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment) | — |  |
| [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks) | — |  |
| [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks) | yes | [Mitigation.DSQL.Cluster.1](#mitigationdsqlcluster1) |
| [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement) | — |  |
| [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets) | — |  |
| [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources) | yes | [Mitigation.DSQL.Cluster.2](#mitigationdsqlcluster2) |
| [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network) | yes | [Mitigation.DSQL.1](#mitigationdsql1) |
| [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition) | — |  |
| [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention) | yes | [Mitigation.DSQL.Cluster.9](#mitigationdsqlcluster9) |
| [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose) | yes | [Mitigation.DSQL.Cluster.8](#mitigationdsqlcluster8), [Mitigation.DSQL.Stream.2](#mitigationdsqlstream2) |
| [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role) | yes | [Mitigation.DSQL.Stream.1](#mitigationdsqlstream1) |
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
| [Account Access Removal](../threat-impacts.md#account-access-removal) | [Threat.DSQL.Cluster.2](#threatdsqlcluster2) |
| [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery) | [Threat.DSQL.Stream.2](#threatdsqlstream2) |
| [Data Collection](../threat-impacts.md#data-collection) | [Threat.DSQL.1](#threatdsql1), [Threat.DSQL.2](#threatdsql2), [Threat.DSQL.Cluster.6](#threatdsqlcluster6), [Threat.DSQL.Cluster.7](#threatdsqlcluster7), [Threat.DSQL.Cluster.9](#threatdsqlcluster9), [Threat.DSQL.Cluster.10](#threatdsqlcluster10), [Threat.DSQL.Cluster.12](#threatdsqlcluster12), [Threat.DSQL.Cluster.13](#threatdsqlcluster13) |
| [Data Destruction](../threat-impacts.md#data-destruction) | [Threat.DSQL.Cluster.1](#threatdsqlcluster1), [Threat.DSQL.Cluster.2](#threatdsqlcluster2), [Threat.DSQL.Cluster.3](#threatdsqlcluster3), [Threat.DSQL.Cluster.8](#threatdsqlcluster8), [Threat.DSQL.Cluster.10](#threatdsqlcluster10), [Threat.DSQL.Cluster.11](#threatdsqlcluster11) |
| [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Threat.DSQL.Cluster.4](#threatdsqlcluster4) |
| [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Threat.DSQL.1](#threatdsql1), [Threat.DSQL.2](#threatdsql2), [Threat.DSQL.Cluster.7](#threatdsqlcluster7), [Threat.DSQL.Cluster.8](#threatdsqlcluster8), [Threat.DSQL.Cluster.9](#threatdsqlcluster9), [Threat.DSQL.Cluster.10](#threatdsqlcluster10), [Threat.DSQL.Cluster.12](#threatdsqlcluster12) |
| [Defense Evasion](../threat-impacts.md#defense-evasion) | [Threat.DSQL.Cluster.5](#threatdsqlcluster5), [Threat.DSQL.Cluster.8](#threatdsqlcluster8), [Threat.DSQL.Stream.1](#threatdsqlstream1) |
| [Initial Access](../threat-impacts.md#initial-access) | [Threat.DSQL.Cluster.8](#threatdsqlcluster8), [Threat.DSQL.Cluster.10](#threatdsqlcluster10), [Threat.DSQL.Stream.2](#threatdsqlstream2) |
| [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Threat.DSQL.Cluster.13](#threatdsqlcluster13) |
