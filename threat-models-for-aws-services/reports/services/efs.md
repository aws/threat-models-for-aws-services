# Amazon Elastic File System (efs) Threat Model

[Threat Summary](#threat-summary) · [Threats](#threats) · [Mitigations](#mitigations) · [Implementations](#implementations) · [Appendix](#appendix)

## Overview

Amazon Elastic File System provides serverless, elastic file storage for Linux-based workloads. It presents a shared POSIX file system that grows and shrinks automatically as files are added and removed, so you do not provision or manage storage capacity. The service manages the underlying file storage infrastructure, including deployment, patching, and configuration.

27 threats and 22 mitigations across 4 resource types and 31 API operations. 21 of 38 mitigation goals are covered, implemented by 65 implementations.

Source service model: [efs](https://github.com/aws/api-models-aws/blob/a0767ac42e279dfc8b311564c33436f8abfd3031/models/efs/service/2015-02-01/efs-2015-02-01.json)

---

## Threat Summary

### [Data exposure from missing encryption at rest](../potential-threats.md#data-exposure-from-missing-encryption-at-rest)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.8](#threatefsfilesystem8) | IAM principal, with IAM permissions to create an Amazon EFS file system, can create the file system to not use encryption at rest. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2), [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8) |

### [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.19](#threatefsfilesystem19) | Threat actor, with network adjacency to the connection path between clients and an Amazon EFS file system that has not been configured to encrypt data in transit, can capture the file system's NFS network traffic on TCP port 2049 to a mount target elastic network interface and read data in transit that has not been encrypted, including file contents, file names, directory listings and client-asserted POSIX UID and GID values. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14) |

### [Data inaccessibility through resource-based policy lockout](../potential-threats.md#data-inaccessibility-through-resource-based-policy-lockout)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.2](#threatefsfilesystem2) | IAM principal, with IAM permissions to modify a file system policy governing an Amazon EFS file system, can make the file system inaccessible by modifying the file system resource-based policy to deny policy-management and client mount actions for all principals, bypassing the policy lockout safety check. | [Account Access Removal](../threat-impacts.md#account-access-removal), [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16) |

### [Data loss through encryption key deletion](../potential-threats.md#data-loss-through-encryption-key-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.3](#threatefsfilesystem3) | IAM principal, with IAM permissions to delete the AWS KMS customer managed key used by an Amazon EFS file system, can delete the AWS KMS customer managed key used by the file system. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11) |

### [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.4](#threatefsfilesystem4) | IAM principal, with IAM permissions to create or modify data in an Amazon EFS file system, can encrypt the file data written to the file system with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11) |

### [Data loss through resource deletion](../potential-threats.md#data-loss-through-resource-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.1](#threatefsfilesystem1) | IAM principal, with IAM permissions to delete an Amazon EFS file system, can delete the file system and the file data that the file system contains. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11) |

### [Direct data deletion](../potential-threats.md#direct-data-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.15](#threatefsfilesystem15) | IAM principal, with IAM permissions to delete files and directories from an Amazon EFS file system, can delete files and directories from the file system. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6) |

### [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.1](#threatefs1) | Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon EFS service API calls. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EFS.2](#mitigationefs2) |

### [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.14](#threatefsfilesystem14) | Untrusted IAM principal, that has been granted access to my Amazon EFS file system through its resource-based policy, can use my file system. | [Data Collection](../threat-impacts.md#data-collection), [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13) |

### [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.16](#threatefsfilesystem16) | Untrusted IAM principal, that has been granted access to my Amazon EFS file system's IAM replication service role through its trust policy, can assume my replication role and act with its permissions, reading the replicated file system data and creating a replication configuration that targets a destination file system they control. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1) |

### [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.MountTarget.2](#threatefsmounttarget2) | Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon EFS mount target, can interact with the mount target over NFS on TCP port 2049 through overly permissive network rules and move laterally within the VPC. | [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3) |

### [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.6](#threatefsfilesystem6) | IAM principal, with IAM permissions to create or modify an Amazon EFS file system, can encrypt the file system with an AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle. | [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8) |

### [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.MountTarget.3](#threatefsmounttarget3) | Threat actor, with network access to an Amazon EFS mount target that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the mount target. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3) |

### [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.12](#threatefsfilesystem12) | IAM principal, with IAM permissions on my Amazon EFS file system and access to an unexpected network, can interact with my file system from an unexpected network. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3) |

### [Network traffic interception through AWS network-inspection features](../potential-threats.md#network-traffic-interception-through-aws-network-inspection-features)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.18](#threatefsfilesystem18) | Threat actor, with permissions to configure AWS network-inspection resources that target the network interfaces used by the Amazon EFS file system in a VPC, or to redirect traffic to or from those interfaces through an inspection appliance they control, can capture network traffic at the mount target network interfaces used by the file system or at the redirected path, and read NFS data in transit that has not been encrypted. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14) |

### [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.MountTarget.4](#threatefsmounttarget4) | Threat actor, with network access to an Amazon EFS mount target that has been placed in a public subnet, can access the mount target directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1) |

### [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.AccessPoint.1](#threatefsaccesspoint1) | IAM principal, with IAM permissions to create or modify an Amazon EFS access point, can configure the access point without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1) |
| [Threat.EFS.FileSystem.5](#threatefsfilesystem5) | IAM principal, with IAM permissions to create or modify an Amazon EFS file system, can configure the file system without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15) |

### [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.2](#threatefs2) | IAM principal, with IAM permissions on untrusted Amazon EFS resources, can interact with those untrusted resources from a governed VPC network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EFS.1](#mitigationefs1) |

### [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.10](#threatefsfilesystem10) | My IAM principal, with IAM permissions on an untrusted Amazon EFS file system, can interact with the untrusted file system, including sending data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EFS.FileSystem.5](#mitigationefsfilesystem5) |

### [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.11](#threatefsfilesystem11) | IAM principal, with IAM permissions to create an Amazon EFS file system, can create the file system in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11) |
| [Threat.EFS.MountTarget.1](#threatefsmounttarget1) | IAM principal, with IAM permissions to create an Amazon EFS mount target, can create the mount target in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2) |

### [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.MountTarget.5](#threatefsmounttarget5) | Threat actor, with access to an Amazon EFS mount target, can submit a volume of NFS requests that exceeds the capacity of the mount target or the network it is connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1) |

### [Unauthenticated access from connected networks](../potential-threats.md#unauthenticated-access-from-connected-networks)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.13](#threatefsfilesystem13) | Threat actor, with network access to an Amazon EFS file system that does not require authentication for data plane operations, can access the file system without presenting valid credentials. | [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9), [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12) |

### [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.9](#threatefsfilesystem9) | IAM principal, with unintended IAM permissions to read file data from an Amazon EFS file system where the service uses grant-based KMS authorization, can read file data in the file system over the NFS protocol because the KMS key policy cannot restrict which IAM principals can decrypt the data. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13) |

### [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.7](#threatefsfilesystem7) | Threat actor, with a method for enumerating public Amazon EFS file systems, or prior knowledge of an existing public file system, can access a file system that has been configured to allow public access. | [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10), [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12) |

### [Weak authentication problem](../potential-threats.md#weak-authentication-problem)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EFS.FileSystem.17](#threatefsfilesystem17) | Threat actor, with knowledge of an Amazon EFS file system that has been configured to use client-asserted POSIX UID/GID authentication for NFS clients, can take advantage of the weak authentication mechanism to access the file system by mounting it over NFS and asserting an arbitrary UID/GID, including root UID 0. | [Initial Access](../threat-impacts.md#initial-access), [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9) |

---

## Threats

Each scoping statement below describes the exposure the threat's mapped mitigations address. It assumes those mitigations are not applied and makes no claim about your resources.

### Threat.EFS.1

Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon EFS service API calls.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EFS.2](#mitigationefs2)

**Frameworks:** MITRE ATT&CK [T1074.002](https://attack.mitre.org/techniques/T1074/002/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)): Amazon EFS supports interface VPC endpoints, so where one is deployed, the `elasticfilesystem` control plane is reachable privately from your VPC network. A principal outside your zone of trust operating from that network can send API requests, signed with its own credentials, against its own resources.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateAccessPoint` | `$.FileSystemId`, `$.PosixUser.Uid`, `$.RootDirectory.Path` |
| `efs:CreateFileSystem` | `$.CreationToken`, `$.Encrypted`, `$.KmsKeyId` |
| `efs:CreateMountTarget` | `$.FileSystemId`, `$.SecurityGroups[*]`, `$.SubnetId` |
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].FileSystemId`, `$.Destinations[*].Region`, `$.Destinations[*].RoleArn`, `$.SourceFileSystemId` |
| `efs:DeleteFileSystem` | `$.FileSystemId` |
| `efs:DeleteFileSystemPolicy` | `$.FileSystemId` |
| `efs:PutFileSystemPolicy` | `$.BypassPolicyLockoutSafetyCheck`, `$.FileSystemId`, `$.Policy` |
| `efs:UpdateFileSystem` | `$.FileSystemId`, `$.ThroughputMode` |

### Threat.EFS.2

IAM principal, with IAM permissions on untrusted Amazon EFS resources, can interact with those untrusted resources from a governed VPC network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EFS.1](#mitigationefs1)

**Frameworks:** MITRE ATT&CK [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)): Principals in your VPC network can call Amazon EFS API operations against file systems in accounts outside your organization. Calling `CreateReplicationConfiguration` with a cross-account file system ARN in `$.Destinations[*].FileSystemId` can copy your file data to that account.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].FileSystemId`, `$.Destinations[*].RoleArn`, `$.SourceFileSystemId` |
| `efs:DeleteFileSystem` | `$.FileSystemId` |
| `efs:PutFileSystemPolicy` | `$.FileSystemId`, `$.Policy` |
| `efs:UpdateFileSystem` | `$.FileSystemId` |

### Threat.EFS.AccessPoint.1

IAM principal, with IAM permissions to create or modify an Amazon EFS access point, can configure the access point without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An Amazon EFS access point can be created with no tags, or have its existing tags removed later. An untagged access point still works normally and carries no owner or purpose information, so it escapes ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateAccessPoint` | `$.FileSystemId`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `efs:UntagResource` | `$.ResourceId`, `$.TagKeys[*]` |

### Threat.EFS.FileSystem.1

IAM principal, with IAM permissions to delete an Amazon EFS file system, can delete the file system and the file data that the file system contains.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Scoping** (for [Data loss through resource deletion](../potential-threats.md#data-loss-through-resource-deletion)): The file system stores your file data, so deleting the resource also removes that data. An IAM principal holding delete permissions can remove the file system through the `DeleteFileSystem` operation, without permissions on individual files.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:DeleteFileSystem` | `$.FileSystemId` |

### Threat.EFS.FileSystem.2

IAM principal, with IAM permissions to modify a file system policy governing an Amazon EFS file system, can make the file system inaccessible by modifying the file system resource-based policy to deny policy-management and client mount actions for all principals, bypassing the policy lockout safety check.

**Impacts:** [Account Access Removal](../threat-impacts.md#account-access-removal), [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/), [T1531](https://attack.mitre.org/techniques/T1531/)

**Scoping** (for [Data inaccessibility through resource-based policy lockout](../potential-threats.md#data-inaccessibility-through-resource-based-policy-lockout)): A principal with `elasticfilesystem:PutFileSystemPolicy` permissions can replace your file system policy with a document that denies policy-management and client mount actions. Bypassing the policy lockout safety check leaves the file system in place but unmanageable, with its stored file data unreachable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:PutFileSystemPolicy` | `$.BypassPolicyLockoutSafetyCheck`, `$.FileSystemId`, `$.Policy` |

### Threat.EFS.FileSystem.3

IAM principal, with IAM permissions to delete the AWS KMS customer managed key used by an Amazon EFS file system, can delete the AWS KMS customer managed key used by the file system.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Scoping** (for [Data loss through encryption key deletion](../potential-threats.md#data-loss-through-encryption-key-deletion)): Your file system permanently binds a symmetric AWS KMS customer managed key at creation, with no re-key path. A threat actor able to delete that key can render the file system permanently inaccessible without calling any Amazon EFS operation.

**Request parameters**

| Operation | Parameters |
|---|---|
| `kms:DeleteImportedKeyMaterial` | `$.KeyId`, `$.KeyMaterialId` |
| `kms:DisconnectCustomKeyStore` | `$.CustomKeyStoreId` |
| `kms:ScheduleKeyDeletion` | `$.KeyId`, `$.PendingWindowInDays` |

### Threat.EFS.FileSystem.4

IAM principal, with IAM permissions to create or modify data in an Amazon EFS file system, can encrypt the file data written to the file system with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/), [T1490](https://attack.mitre.org/techniques/T1490/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): A principal with permissions to create a file system or a replication destination chooses which AWS KMS key encrypts the file data. That key can belong to an account you do not administer.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.Encrypted`, `$.KmsKeyId` |
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].KmsKeyId`, `$.SourceFileSystemId` |

### Threat.EFS.FileSystem.5

IAM principal, with IAM permissions to create or modify an Amazon EFS file system, can configure the file system without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): Tagging is optional on a file system and reversible after creation. A file system without the required tags may fall outside your ownership attribution and cost allocation processes.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.Tags[*].Key`, `$.Tags[*].Value` |
| `efs:DeleteTags` | `$.FileSystemId`, `$.TagKeys[*]` |
| `efs:TagResource` | `$.ResourceId`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `efs:UntagResource` | `$.ResourceId`, `$.TagKeys[*]` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.EFS.FileSystem.6

IAM principal, with IAM permissions to create or modify an Amazon EFS file system, can encrypt the file system with an AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8)

**Frameworks:** MITRE ATT&CK [T1074.002](https://attack.mitre.org/techniques/T1074/002/)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): A principal that can create or modify your Amazon EFS file system can encrypt its file data under the AWS managed key. That key is managed by AWS, so you do not control its permissions, usage scope, or rotation schedule.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.KmsKeyId` |
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].KmsKeyId`, `$.SourceFileSystemId` |

### Threat.EFS.FileSystem.7

Threat actor, with a method for enumerating public Amazon EFS file systems, or prior knowledge of an existing public file system, can access a file system that has been configured to allow public access.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10), [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1485](https://attack.mitre.org/techniques/T1485/)

**Protocols:** `NFS`

**Scoping** (for [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)): Your file system can be configured to allow public access. A threat actor that enumerates public file systems, or already knows of one, can then access it.

### Threat.EFS.FileSystem.8

IAM principal, with IAM permissions to create an Amazon EFS file system, can create the file system to not use encryption at rest.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2), [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Data exposure from missing encryption at rest](../potential-threats.md#data-exposure-from-missing-encryption-at-rest)): Encryption at rest is an optional setting on your Amazon EFS file system, chosen when you create it. A principal with permissions to create the file system can leave that setting off, storing file data in plaintext.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.CreationToken`, `$.Encrypted`, `$.KmsKeyId` |

### Threat.EFS.FileSystem.9

IAM principal, with unintended IAM permissions to read file data from an Amazon EFS file system where the service uses grant-based KMS authorization, can read file data in the file system over the NFS protocol because the KMS key policy cannot restrict which IAM principals can decrypt the data.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `NFS`

**Scoping** (for [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)): Amazon EFS decrypts your file data under a KMS grant it holds, so the key policy cannot restrict which principals read the data. An IAM principal with unintended read permissions and network access to a mount target can read plaintext file data over NFS.

**Data perimeter:** `ClientMount` is used to access file systems in your VPCs with IAM authentication.

**Data perimeter:** `ClientRootAccess` is used to access file systems in your VPCs with IAM authentication.

**Data perimeter:** `ClientWrite` is used to access file systems in your VPCs with IAM authentication.

**Data perimeter:** `PutFileSystemPolicy` allows you to apply a resource-based policy to grant access to a file system. The service currently does not support resource control policies.

### Threat.EFS.FileSystem.10

My IAM principal, with IAM permissions on an untrusted Amazon EFS file system, can interact with the untrusted file system, including sending data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EFS.FileSystem.5](#mitigationefsfilesystem5)

**Frameworks:** MITRE ATT&CK [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): Your IAM principals can hold permissions on an untrusted Amazon EFS file system, not only on the file systems you own. Such a principal can send data to that untrusted file system or receive data from it.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].FileSystemId`, `$.Destinations[*].Region`, `$.Destinations[*].RoleArn`, `$.SourceFileSystemId` |
| `efs:DeleteFileSystem` | `$.FileSystemId` |

### Threat.EFS.FileSystem.11

IAM principal, with IAM permissions to create an Amazon EFS file system, can create the file system in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11)

**Frameworks:** MITRE ATT&CK [T1499](https://attack.mitre.org/techniques/T1499/)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): Supplying `$.AvailabilityZoneName` on `CreateFileSystem` creates a One Zone file system that stores your data in exactly one Availability Zone. That file system can have only a single mount target in the same Availability Zone, so an impairment there removes the only path your clients use to reach the data.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.AvailabilityZoneName` |
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].AvailabilityZoneName` |

### Threat.EFS.FileSystem.12

IAM principal, with IAM permissions on my Amazon EFS file system and access to an unexpected network, can interact with my file system from an unexpected network.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1537](https://attack.mitre.org/techniques/T1537/), [T1562.007](https://attack.mitre.org/techniques/T1562/007/)

**Protocols:** `NFS`

**Scoping** (for [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)): An IAM principal with permissions on your file system can interact with it from an unexpected network. That interaction can include mounting the file system over NFS and making Amazon EFS API calls against it.

**Data perimeter:** `ClientMount` is used to access file systems in your VPCs with IAM authentication.

**Data perimeter:** `ClientRootAccess` is used to access file systems in your VPCs with IAM authentication.

**Data perimeter:** `ClientWrite` is used to access file systems in your VPCs with IAM authentication.

**Data perimeter:** `PutFileSystemPolicy` allows you to apply a resource-based policy to grant access to a file system. The service currently does not support resource control policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].FileSystemId`, `$.Destinations[*].Region`, `$.SourceFileSystemId` |
| `efs:DeleteFileSystem` | `$.FileSystemId` |
| `efs:DeleteFileSystemPolicy` | `$.FileSystemId` |
| `efs:PutFileSystemPolicy` | `$.BypassPolicyLockoutSafetyCheck`, `$.Policy` |
| `efs:UpdateFileSystem` | `$.ThroughputMode` |

### Threat.EFS.FileSystem.13

Threat actor, with network access to an Amazon EFS file system that does not require authentication for data plane operations, can access the file system without presenting valid credentials.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9), [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1133](https://attack.mitre.org/techniques/T1133/), [T1485](https://attack.mitre.org/techniques/T1485/)

**Protocols:** `NFS`

**Scoping** (for [Unauthenticated access from connected networks](../potential-threats.md#unauthenticated-access-from-connected-networks)): Your file system exposes an NFSv4 data plane on TCP port 2049 at each mount target, reachable from the local VPC, connected VPCs, and on-premises networks. If a newly created file system grants anonymous clients full access, a client asserting the root user ID of 0 can read, write, and delete any file.

### Threat.EFS.FileSystem.14

Untrusted IAM principal, that has been granted access to my Amazon EFS file system through its resource-based policy, can use my file system.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1485](https://attack.mitre.org/techniques/T1485/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Protocols:** `NFS`

**Scoping** (for [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)): The resource-based policy on your Amazon EFS file system can grant access to an identity outside your organization, deliberately or through an over-broad grant. With network reachability to a mount target, that identity can mount the file system to read, overwrite, or delete your data. It can also copy that data to a file system it owns.

**Data perimeter:** `PutFileSystemPolicy` allows you to apply a resource-based policy to grant access to a file system. The service currently does not support resource control policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:DeleteFileSystem` | `$.FileSystemId` |
| `efs:PutFileSystemPolicy` | `$.FileSystemId`, `$.Policy` |

### Threat.EFS.FileSystem.15

IAM principal, with IAM permissions to delete files and directories from an Amazon EFS file system, can delete files and directories from the file system.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Protocols:** `NFS`

**Scoping** (for [Direct data deletion](../potential-threats.md#direct-data-deletion)): Your Amazon EFS file system stores individually addressable files and directories. An IAM principal with delete permissions can remove them over NFS while the file system itself persists.

### Threat.EFS.FileSystem.16

Untrusted IAM principal, that has been granted access to my Amazon EFS file system's IAM replication service role through its trust policy, can assume my replication role and act with its permissions, reading the replicated file system data and creating a replication configuration that targets a destination file system they control.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): You author the trust policy on the replication role that your file system's replication configuration designates (`$.Destinations[*].RoleArn`). That policy can name identities outside your organization that can assume the role and replicate your file system's data to a destination they control.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].RoleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.EFS.FileSystem.17

Threat actor, with knowledge of an Amazon EFS file system that has been configured to use client-asserted POSIX UID/GID authentication for NFS clients, can take advantage of the weak authentication mechanism to access the file system by mounting it over NFS and asserting an arbitrary UID/GID, including root UID 0.

**Impacts:** [Initial Access](../threat-impacts.md#initial-access), [Privilege Escalation](../threat-impacts.md#privilege-escalation)

**Mitigations:** [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9)

**Frameworks:** MITRE ATT&CK [T1078](https://attack.mitre.org/techniques/T1078/), [T1078.001](https://attack.mitre.org/techniques/T1078/001/)

**Protocols:** `NFS`

**Scoping** (for [Weak authentication problem](../potential-threats.md#weak-authentication-problem)): By default, your file system trusts the numeric POSIX user and group IDs that a mounting NFS client asserts about itself. A threat actor with network reachability to a mount target can mount the file system and claim any identity, including root UID 0.

### Threat.EFS.FileSystem.18

Threat actor, with permissions to configure AWS network-inspection resources that target the network interfaces used by the Amazon EFS file system in a VPC, or to redirect traffic to or from those interfaces through an inspection appliance they control, can capture network traffic at the mount target network interfaces used by the file system or at the redirected path, and read NFS data in transit that has not been encrypted.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/), [T1580](https://attack.mitre.org/techniques/T1580/)

**Scoping** (for [Network traffic interception through AWS network-inspection features](../potential-threats.md#network-traffic-interception-through-aws-network-inspection-features)): Your file system's data plane is reachable only through mount targets in your VPC. Each mount target is a customer-visible elastic network interface that can be named as a flow log, traffic mirror, or route target.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ec2:CreateFlowLogs` | `$.LogDestination`, `$.ResourceId[*]`, `$.ResourceType`, `$.TrafficType` |
| `ec2:CreateRoute` | `$.VpcEndpointId`, `$.destinationCidrBlock`, `$.networkInterfaceId`, `$.routeTableId` |
| `ec2:CreateTrafficMirrorFilterRule` | `$.DestinationCidrBlock`, `$.DestinationPortRange.FromPort`, `$.DestinationPortRange.ToPort`, `$.TrafficDirection` |
| `ec2:CreateTrafficMirrorSession` | `$.NetworkInterfaceId`, `$.PacketLength`, `$.TrafficMirrorFilterId`, `$.TrafficMirrorTargetId` |
| `ec2:CreateTrafficMirrorTarget` | `$.GatewayLoadBalancerEndpointId`, `$.NetworkInterfaceId`, `$.NetworkLoadBalancerArn` |
| `ec2:ReplaceRoute` | `$.VpcEndpointId`, `$.destinationCidrBlock`, `$.networkInterfaceId`, `$.routeTableId` |
| `efs:DescribeMountTargets` | `$.FileSystemId` |
| `network-firewall:StartFlowCapture` | `$.FirewallArn`, `$.FlowFilters[*].DestinationAddress.AddressDefinition`, `$.FlowFilters[*].DestinationPort` |

### Threat.EFS.FileSystem.19

Threat actor, with network adjacency to the connection path between clients and an Amazon EFS file system that has not been configured to encrypt data in transit, can capture the file system's NFS network traffic on TCP port 2049 to a mount target elastic network interface and read data in transit that has not been encrypted, including file contents, file names, directory listings and client-asserted POSIX UID and GID values.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Protocols:** `NFS`

**Scoping** (for [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)): A threat actor with network adjacency to the client-to-mount-target path can capture your file system's NFS traffic on TCP port 2049. Captured traffic that is not encrypted reveals file contents, file names, directory listings, and client-asserted POSIX UID and GID values.

### Threat.EFS.MountTarget.1

IAM principal, with IAM permissions to create an Amazon EFS mount target, can create the mount target in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2)

**Frameworks:** MITRE ATT&CK [T1489](https://attack.mitre.org/techniques/T1489/), [T1499](https://attack.mitre.org/techniques/T1499/)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An Amazon EFS mount target is a zonal endpoint whose Availability Zone is fixed by the `$.SubnetId` you pass to `CreateMountTarget`, and EFS allows only one mount target per zone. A file system left with a single mount target has one reachable NFS endpoint, so clients lose access to file data if that Availability Zone fails.

**Request parameters**

| Operation | Parameters |
|---|---|
| `efs:CreateMountTarget` | `$.FileSystemId`, `$.SubnetId` |
| `efs:DeleteMountTarget` | `$.MountTargetId` |

### Threat.EFS.MountTarget.2

Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon EFS mount target, can interact with the mount target over NFS on TCP port 2049 through overly permissive network rules and move laterally within the VPC.

**Impacts:** [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3)

**Frameworks:** MITRE ATT&CK [T1021](https://attack.mitre.org/techniques/T1021/), [T1080](https://attack.mitre.org/techniques/T1080/)

**Protocols:** `NFS`

**Scoping** (for [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)): Overly permissive inbound network rules on your mount target can admit NFS traffic on TCP port 2049 from any VPC-connected resource. A threat actor controlling such a resource may mount your file system and move laterally within your VPC.

### Threat.EFS.MountTarget.3

Threat actor, with network access to an Amazon EFS mount target that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the mount target.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1133](https://attack.mitre.org/techniques/T1133/)

**Protocols:** `NFS`

**Scoping** (for [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)): An Amazon EFS mount target presents a network interface in your VPC whose inbound security group rules admit traffic to it. An unrestricted source range in those rules lets any host that can route to the interface reach its listening services.

### Threat.EFS.MountTarget.4

Threat actor, with network access to an Amazon EFS mount target that has been placed in a public subnet, can access the mount target directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1580](https://attack.mitre.org/techniques/T1580/)

**Protocols:** `NFS`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): A mount target is a VPC network endpoint that NFS clients connect to in order to reach your file system. A mount target placed in a subnet that routes to an internet gateway can be reached directly from an ungoverned network.

### Threat.EFS.MountTarget.5

Threat actor, with access to an Amazon EFS mount target, can submit a volume of NFS requests that exceeds the capacity of the mount target or the network it is connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1)

**Frameworks:** MITRE ATT&CK [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1499.002](https://attack.mitre.org/techniques/T1499/002/)

**Protocols:** `NFS`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): Your mount target is a network endpoint in your subnet that receives NFS requests and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

---

## Mitigations

Each scoping statement below names the capability the mitigation goal governs on the entity and the objective of governing it. Each mitigation's implementations describe how to govern or monitor that capability.

### Mitigation.EFS.1

Amazon EFS service API calls on a governed VPC network should only be made against trusted resources or AWS service-owned resources

**Goal:** [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)

**Addresses:** [Threat.EFS.2](#threatefs2)

**Implemented by:** [Implementation.EFS.1](#implementationefs1)

**Scoping** (for [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)): Calls to the Amazon EFS API through an interface VPC endpoint (powered by AWS PrivateLink) can target resources outside your organization. Under the resource perimeter objective, your identities can access only trusted resources from your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.EFS.2

Amazon EFS service API calls on a governed VPC network should only be made by trusted identities or AWS services

**Goal:** [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)

**Addresses:** [Threat.EFS.1](#threatefs1)

**Implemented by:** [Implementation.EFS.2](#implementationefs2)

**Scoping** (for [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)): You can call the Amazon EFS API privately from your VPC through an interface VPC endpoint powered by AWS PrivateLink. Amazon EFS API calls made over that network path are therefore in scope for the identity perimeter applied on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.EFS.AccessPoint.1

An Amazon EFS access point should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EFS.AccessPoint.1](#threatefsaccesspoint1)

**Implemented by:** [Implementation.EFS.AccessPoint.1](#implementationefsaccesspoint1), [Implementation.EFS.AccessPoint.2](#implementationefsaccesspoint2), [Implementation.EFS.AccessPoint.3](#implementationefsaccesspoint3)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): An Amazon EFS access point is a durable resource that supports its own tags. You can apply tags that describe the owner and purpose of each access point.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateAccessPoint` | `$.FileSystemId`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `efs:TagResource` | `$.ResourceId`, `$.Tags[*].Key`, `$.Tags[*].Value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeAccessPoints` | `$.AccessPointId` |
| `efs:ListTagsForResource` | `$.ResourceId` |

### Mitigation.EFS.FileSystem.1

An Amazon EFS file system's IAM replication service role should be assumed only by Amazon EFS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.EFS.FileSystem.16](#threatefsfilesystem16)

**Implemented by:** [Implementation.EFS.FileSystem.1](#implementationefsfilesystem1), [Implementation.EFS.FileSystem.2](#implementationefsfilesystem2), [Implementation.EFS.FileSystem.3](#implementationefsfilesystem3), [Implementation.EFS.FileSystem.4](#implementationefsfilesystem4)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): When you replicate an Amazon EFS file system, you can pass an IAM role through `$.Destinations[*].RoleArn` that Amazon EFS assumes to synchronize your data. Restricting this role's trust policy to Amazon EFS, acting on behalf of trusted resources, helps prevent an unintended principal from assuming the role.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].RoleArn`, `$.SourceFileSystemId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeReplicationConfigurations` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.2

An Amazon EFS file system should have encryption at rest enabled

**Goal:** [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest)

**Addresses:** [Threat.EFS.FileSystem.8](#threatefsfilesystem8)

**Implemented by:** [Implementation.EFS.FileSystem.5](#implementationefsfilesystem5), [Implementation.EFS.FileSystem.6](#implementationefsfilesystem6), [Implementation.EFS.FileSystem.7](#implementationefsfilesystem7), [ManagedControl.EFS.FileSystem.1](#managedcontrolefsfilesystem1), [ManagedControl.EFS.FileSystem.2](#managedcontrolefsfilesystem2), [ManagedControl.EFS.FileSystem.3](#managedcontrolefsfilesystem3)

**Scoping** (for [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest)): Amazon EFS treats encryption at rest as an optional setting that you request through `$.Encrypted` when you create a file system. You cannot encrypt a file system after it is created, so enabling it at creation helps protect your stored file data.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.Encrypted` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystems` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.3

An Amazon EFS file system should be accessed only from expected networks

**Goal:** [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)

**Addresses:** [Threat.EFS.FileSystem.9](#threatefsfilesystem9), [Threat.EFS.FileSystem.12](#threatefsfilesystem12)

**Implemented by:** [Implementation.EFS.FileSystem.8](#implementationefsfilesystem8), [Implementation.EFS.FileSystem.9](#implementationefsfilesystem9), [Implementation.EFS.FileSystem.10](#implementationefsfilesystem10), [Implementation.EFS.FileSystem.11](#implementationefsfilesystem11), [ManagedControl.EFS.FileSystem.4](#managedcontrolefsfilesystem4)

**Scoping** (for [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)): Your Amazon EFS file system supports a file system policy that EFS evaluates when NFS clients connect. That policy can restrict client access based on the network location a connection originates from.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:PutFileSystemPolicy` | `$.FileSystemId`, `$.Policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystemPolicy` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.4

An Amazon EFS file system should have automatic backups enabled

**Goal:** [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native)

**Addresses:** [Threat.EFS.FileSystem.1](#threatefsfilesystem1), [Threat.EFS.FileSystem.15](#threatefsfilesystem15)

**Implemented by:** [Implementation.EFS.FileSystem.12](#implementationefsfilesystem12), [Implementation.EFS.FileSystem.13](#implementationefsfilesystem13), [Implementation.EFS.FileSystem.14](#implementationefsfilesystem14), [ManagedControl.EFS.FileSystem.5](#managedcontrolefsfilesystem5), [ManagedControl.EFS.FileSystem.6](#managedcontrolefsfilesystem6), [ManagedControl.EFS.FileSystem.7](#managedcontrolefsfilesystem7), [ManagedControl.EFS.FileSystem.8](#managedcontrolefsfilesystem8), [ManagedControl.EFS.FileSystem.9](#managedcontrolefsfilesystem9), [ManagedControl.EFS.FileSystem.10](#managedcontrolefsfilesystem10)

**Scoping** (for [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native)): An Amazon EFS file system stores durable file data that EFS can capture as restorable recovery points. Amazon EFS provides a native automatic backup setting on the file system itself.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.Backup` |
| `efs:PutBackupPolicy` | `$.BackupPolicy.Status`, `$.FileSystemId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeBackupPolicy` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.5

IAM principals should be restricted to accessing only trusted Amazon EFS file system resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.EFS.FileSystem.10](#threatefsfilesystem10)

**Implemented by:** [Implementation.EFS.FileSystem.15](#implementationefsfilesystem15)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): Amazon EFS API calls can target a file system outside your organization, such as the source or destination of a cross-account replication configuration. Restricting those calls to file systems in your organization helps establish a resource perimeter, in which your identities can access only trusted resources.

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

### Mitigation.EFS.FileSystem.6

An Amazon EFS file system should be protected by an AWS Backup backup plan

**Goal:** [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup)

**Addresses:** [Threat.EFS.FileSystem.1](#threatefsfilesystem1), [Threat.EFS.FileSystem.15](#threatefsfilesystem15)

**Implemented by:** [Implementation.EFS.FileSystem.16](#implementationefsfilesystem16), [Implementation.EFS.FileSystem.17](#implementationefsfilesystem17), [Implementation.EFS.FileSystem.18](#implementationefsfilesystem18), [ManagedControl.EFS.FileSystem.5](#managedcontrolefsfilesystem5), [ManagedControl.EFS.FileSystem.6](#managedcontrolefsfilesystem6), [ManagedControl.EFS.FileSystem.7](#managedcontrolefsfilesystem7), [ManagedControl.EFS.FileSystem.8](#managedcontrolefsfilesystem8), [ManagedControl.EFS.FileSystem.9](#managedcontrolefsfilesystem9), [ManagedControl.EFS.FileSystem.10](#managedcontrolefsfilesystem10), [ManagedControl.EFS.FileSystem.11](#managedcontrolefsfilesystem11)

**Scoping** (for [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup)): An Amazon EFS file system stores durable customer file data that AWS Backup can capture in recovery points. AWS Backup natively supports the file system as an independently protectable resource type that a backup plan can target.

**Configure via**

| Operation | Parameters |
|---|---|
| `backup:CreateBackupPlan` | `$.BackupPlan.BackupPlanName`, `$.BackupPlan.Rules[*].Lifecycle.DeleteAfterDays`, `$.BackupPlan.Rules[*].ScheduleExpression`, `$.BackupPlan.Rules[*].TargetBackupVaultName` |
| `backup:CreateBackupSelection` | `$.BackupPlanId`, `$.BackupSelection.Conditions.StringEquals[*].ConditionKey`, `$.BackupSelection.Conditions.StringEquals[*].ConditionValue`, `$.BackupSelection.IamRoleArn`, `$.BackupSelection.ListOfTags[*].ConditionKey`, `$.BackupSelection.ListOfTags[*].ConditionType`, `$.BackupSelection.ListOfTags[*].ConditionValue`, `$.BackupSelection.NotResources[*]`, `$.BackupSelection.Resources[*]` |

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

### Mitigation.EFS.FileSystem.7

An Amazon EFS file system should be deployed across more than one Availability Zone with the Regional file system type

**Goal:** [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment)

**Addresses:** [Threat.EFS.FileSystem.11](#threatefsfilesystem11)

**Implemented by:** [Implementation.EFS.FileSystem.19](#implementationefsfilesystem19), [Implementation.EFS.FileSystem.20](#implementationefsfilesystem20)

**Scoping** (for [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment)): Amazon EFS lets you choose the file system type when you create a file system, and a Regional file system stores your data redundantly across three or more Availability Zones. A One Zone file system stores data in a single Availability Zone, so it is not resilient to the loss of that Availability Zone.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.AvailabilityZoneName` |
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].AvailabilityZoneName`, `$.SourceFileSystemId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystems` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.8

An Amazon EFS file system should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.EFS.FileSystem.4](#threatefsfilesystem4), [Threat.EFS.FileSystem.6](#threatefsfilesystem6), [Threat.EFS.FileSystem.8](#threatefsfilesystem8)

**Implemented by:** [Implementation.EFS.FileSystem.21](#implementationefsfilesystem21), [Implementation.EFS.FileSystem.22](#implementationefsfilesystem22), [ManagedControl.EFS.FileSystem.2](#managedcontrolefsfilesystem2), [ManagedControl.EFS.FileSystem.3](#managedcontrolefsfilesystem3)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): An Amazon EFS file system accepts an AWS KMS key identifier through `$.KmsKeyId` to protect its data at rest with a customer managed key. A customer managed key gives you control over the key's lifecycle, audit of its use, and the ability to make your file data unavailable.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.KmsKeyId` |
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].KmsKeyId`, `$.SourceFileSystemId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystems` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.9

An Amazon EFS file system should use AWS IAM for NFS client authorization

**Goal:** [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication)

**Addresses:** [Threat.EFS.FileSystem.13](#threatefsfilesystem13), [Threat.EFS.FileSystem.17](#threatefsfilesystem17)

**Implemented by:** [Implementation.EFS.FileSystem.9](#implementationefsfilesystem9), [Implementation.EFS.FileSystem.23](#implementationefsfilesystem23)

**Scoping** (for [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication)): By default, your file system authorizes NFS clients from the POSIX identity that the client asserts, rather than from IAM. Amazon EFS also supports IAM authorization for the NFS clients that mount your file system.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:PutFileSystemPolicy` | `$.FileSystemId`, `$.Policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystemPolicy` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.10

An Amazon EFS file system's policy should limit permissions to a subset of IAM principals, rather than granting access to all IAM principals

**Goal:** [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)

**Addresses:** [Threat.EFS.FileSystem.7](#threatefsfilesystem7)

**Implemented by:** [Implementation.EFS.FileSystem.24](#implementationefsfilesystem24), [Implementation.EFS.FileSystem.25](#implementationefsfilesystem25), [Implementation.EFS.FileSystem.26](#implementationefsfilesystem26)

**Scoping** (for [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)): You set the file system policy for your Amazon EFS file system as a complete IAM resource-based policy document through `$.Policy`. When no explicit policy is set, the default policy grants full access to any client that can connect through a mount target.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:PutFileSystemPolicy` | `$.FileSystemId`, `$.Policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystemPolicy` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.11

An Amazon EFS file system should be replicated to at least one other AWS Region

**Goal:** [Cross-Region Replication](../mitigation-goals.md#cross-region-replication)

**Addresses:** [Threat.EFS.FileSystem.1](#threatefsfilesystem1), [Threat.EFS.FileSystem.3](#threatefsfilesystem3), [Threat.EFS.FileSystem.4](#threatefsfilesystem4), [Threat.EFS.FileSystem.11](#threatefsfilesystem11)

**Implemented by:** [Implementation.EFS.FileSystem.27](#implementationefsfilesystem27), [Implementation.EFS.FileSystem.28](#implementationefsfilesystem28)

**Scoping** (for [Cross-Region Replication](../mitigation-goals.md#cross-region-replication)): Amazon EFS can replicate a file system to a service-managed destination file system in another AWS Region. That second copy of your file data helps protect it from a disruption affecting the source Region.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateReplicationConfiguration` | `$.Destinations[*].AvailabilityZoneName`, `$.Destinations[*].FileSystemId`, `$.Destinations[*].Region`, `$.SourceFileSystemId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeReplicationConfigurations` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.12

An Amazon EFS file system should not allow unauthenticated access through its file system policy

**Goal:** [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention)

**Addresses:** [Threat.EFS.FileSystem.7](#threatefsfilesystem7), [Threat.EFS.FileSystem.13](#threatefsfilesystem13)

**Implemented by:** [Implementation.EFS.FileSystem.25](#implementationefsfilesystem25), [Implementation.EFS.FileSystem.26](#implementationefsfilesystem26), [Implementation.EFS.FileSystem.29](#implementationefsfilesystem29)

**Scoping** (for [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention)): By default, an EFS file system permits clients that connect through a mount target to reach file data without presenting AWS credentials. Requiring authenticated NFS clients is an opt-in choice rather than the default behavior.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:PutFileSystemPolicy` | `$.FileSystemId`, `$.Policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystemPolicy` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.13

An Amazon EFS file system should be accessed only by trusted identities or AWS services acting on behalf of trusted resources

**Goal:** [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)

**Addresses:** [Threat.EFS.FileSystem.9](#threatefsfilesystem9), [Threat.EFS.FileSystem.14](#threatefsfilesystem14)

**Implemented by:** [Implementation.EFS.FileSystem.10](#implementationefsfilesystem10), [Implementation.EFS.FileSystem.11](#implementationefsfilesystem11), [Implementation.EFS.FileSystem.30](#implementationefsfilesystem30), [Implementation.EFS.FileSystem.31](#implementationefsfilesystem31), [Implementation.EFS.FileSystem.32](#implementationefsfilesystem32), [Implementation.EFS.FileSystem.33](#implementationefsfilesystem33)

**Scoping** (for [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)): An Amazon EFS file system supports its own IAM resource-based policy, known as the file system policy. That policy determines which accounts and IAM identities, including those in other accounts, are trusted to access the file system.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:PutFileSystemPolicy` | `$.FileSystemId`, `$.Policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystemPolicy` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.14

An Amazon EFS file system should be configured with a file system policy that requires all connections to be encrypted in transit

**Goal:** [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy)

**Addresses:** [Threat.EFS.FileSystem.18](#threatefsfilesystem18), [Threat.EFS.FileSystem.19](#threatefsfilesystem19)

**Implemented by:** [Implementation.EFS.FileSystem.9](#implementationefsfilesystem9), [Implementation.EFS.FileSystem.23](#implementationefsfilesystem23), [Implementation.EFS.FileSystem.34](#implementationefsfilesystem34), [Implementation.EFS.FileSystem.35](#implementationefsfilesystem35)

**Scoping** (for [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy)): Amazon EFS supports resource-based file system policies, evaluated when NFS clients connect. Encryption in transit is selected per connection at mount time, and the service does not require it by default.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:PutFileSystemPolicy` | `$.FileSystemId`, `$.Policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystemPolicy` | `$.FileSystemId` |

### Mitigation.EFS.FileSystem.15

An Amazon EFS file system should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EFS.FileSystem.5](#threatefsfilesystem5)

**Implemented by:** [Implementation.EFS.FileSystem.36](#implementationefsfilesystem36), [Implementation.EFS.FileSystem.37](#implementationefsfilesystem37), [Implementation.EFS.FileSystem.38](#implementationefsfilesystem38), [ManagedControl.EFS.FileSystem.12](#managedcontrolefsfilesystem12)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): Your Amazon EFS file system is a top-level resource that you can tag at creation and afterward with `TagResource`. Tags let you categorize each file system by owner and purpose.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateFileSystem` | `$.Tags[*].Key`, `$.Tags[*].Value` |
| `efs:TagResource` | `$.ResourceId`, `$.Tags[*].Key`, `$.Tags[*].Value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystems` | `$.FileSystemId` |
| `efs:ListTagsForResource` | `$.ResourceId` |

### Mitigation.EFS.FileSystem.16

An Amazon EFS file system should be configured to disallow file system policy modifications that would remove all policy-management access for the file system

**Goal:** [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention)

**Addresses:** [Threat.EFS.FileSystem.2](#threatefsfilesystem2)

**Implemented by:** [Implementation.EFS.FileSystem.39](#implementationefsfilesystem39), [Implementation.EFS.FileSystem.40](#implementationefsfilesystem40)

**Scoping** (for [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention)): Your file system carries a resource-based file system policy that governs both policy management and client access to your stored file data. A policy update that removes policy-management access can leave the file system unmanageable and its stored data inaccessible.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:PutFileSystemPolicy` | `$.BypassPolicyLockoutSafetyCheck`, `$.FileSystemId`, `$.Policy` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeFileSystemPolicy` | `$.FileSystemId` |

### Mitigation.EFS.MountTarget.1

An Amazon EFS mount target should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.EFS.MountTarget.4](#threatefsmounttarget4), [Threat.EFS.MountTarget.5](#threatefsmounttarget5)

**Implemented by:** [Implementation.EFS.MountTarget.1](#implementationefsmounttarget1), [Implementation.EFS.MountTarget.2](#implementationefsmounttarget2), [ManagedControl.EFS.MountTarget.1](#managedcontrolefsmounttarget1)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): Subnet placement is a required setting on every mount target, carried by `$.SubnetId`. Placing a mount target in a private subnet helps limit which clients can reach your file data.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateMountTarget` | `$.FileSystemId`, `$.IpAddressType`, `$.SubnetId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeMountTargets` | `$.FileSystemId`, `$.MountTargetId` |

### Mitigation.EFS.MountTarget.2

An Amazon EFS mount target should be created in each Availability Zone (AZ) that the file system serves, so that mount target elastic network interfaces (ENIs) exist in more than one AZ

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.EFS.MountTarget.1](#threatefsmounttarget1)

**Implemented by:** [Implementation.EFS.MountTarget.3](#implementationefsmounttarget3), [Implementation.EFS.MountTarget.4](#implementationefsmounttarget4)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): Each mount target is backed by one requester-managed elastic network interface that resides in a single Availability Zone. A file system supports a mount target in each Availability Zone, so multi-zone placement reduces dependence on any one zone.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateMountTarget` | `$.FileSystemId`, `$.SubnetId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeMountTargets` | `$.FileSystemId` |

### Mitigation.EFS.MountTarget.3

An Amazon EFS mount target should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.EFS.MountTarget.2](#threatefsmounttarget2), [Threat.EFS.MountTarget.3](#threatefsmounttarget3)

**Implemented by:** [Implementation.EFS.MountTarget.5](#implementationefsmounttarget5), [Implementation.EFS.MountTarget.6](#implementationefsmounttarget6), [Implementation.EFS.MountTarget.7](#implementationefsmounttarget7)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): You attach your own Amazon VPC security groups to a mount target through `$.SecurityGroups[*]`, and those groups act as the virtual firewall controlling which clients can reach your file system over NFS. Omitting them attaches the VPC default security group, so the rule set needs to be chosen deliberately.

**Configure via**

| Operation | Parameters |
|---|---|
| `efs:CreateMountTarget` | `$.FileSystemId`, `$.SecurityGroups[*]` |
| `efs:ModifyMountTargetSecurityGroups` | `$.MountTargetId`, `$.SecurityGroups[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `efs:DescribeMountTargetSecurityGroups` | `$.MountTargetId` |
| `efs:DescribeMountTargets` | `$.FileSystemId`, `$.MountTargetId` |

---

## Implementations

### Implementation.EFS.1

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.EFS.1](#mitigationefs1)

**Scoping** (for [Mitigation.EFS.1](#mitigationefs1)): This VPC endpoint policy governs Amazon EFS API calls made through your interface VPC endpoint (powered by AWS PrivateLink) with the `aws:ResourceOrgID` condition key. As a result, calls to resources outside your organization are restricted, while exempting AWS services that access resources on your behalf (`aws:PrincipalIsAWSService`).

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.EFS.2

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.EFS.2](#mitigationefs2)

**Scoping** (for [Mitigation.EFS.2](#mitigationefs2)): This VPC endpoint policy governs Amazon EFS API calls with the `aws:PrincipalOrgID` condition key, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, identities outside your organization are restricted from using your interface VPC endpoint (powered by AWS PrivateLink), helping establish an identity perimeter.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.EFS.AccessPoint.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::AccessPoint`: `$.AccessPointTags[*].Key`, `$.AccessPointTags[*].Value`

**Implements:** [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1)

**Scoping** (for [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1)): This implementation checks the access point tags in `$.AccessPointTags[*].Key` and `$.AccessPointTags[*].Value`, so a resource missing your required tag keys is not created. Confirming these tag keys and values are present at create time supports attribution of each access point to an owner and a business purpose.

### Implementation.EFS.AccessPoint.2

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::AccessPoint`: `$.tags[*].key`, `$.tags[*].value`, `$.configuration.AccessPointTags[*].Key`, `$.configuration.AccessPointTags[*].Value`

**Implements:** [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1)

**Scoping** (for [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1)): This implementation flags an access point whose recorded tag keys at `$.tags[*].key` omit the owner and purpose keys you require. Detecting untagged access points helps you attribute each application-specific entry point to an accountable owner and a business function.

### Implementation.EFS.AccessPoint.3

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:CreateAccessPoint` | `*` | `aws:RequestTag/owner` |
| `elasticfilesystem:CreateAccessPoint` | `*` | `aws:RequestTag/purpose` |
| `elasticfilesystem:CreateAccessPoint` | `*` | `aws:TagKeys` |
| `elasticfilesystem:TagResource` | `arn:aws:elasticfilesystem:*:*:access-point/*` | `aws:RequestTag/owner` |
| `elasticfilesystem:TagResource` | `arn:aws:elasticfilesystem:*:*:access-point/*` | `aws:RequestTag/purpose` |
| `elasticfilesystem:TagResource` | `arn:aws:elasticfilesystem:*:*:access-point/*` | `aws:TagKeys` |
| `elasticfilesystem:UntagResource` | `arn:aws:elasticfilesystem:*:*:access-point/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1)

**Scoping** (for [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1)): This implementation governs `elasticfilesystem:CreateAccessPoint` and `elasticfilesystem:TagResource` with the `aws:RequestTag/owner` and `aws:RequestTag/purpose` condition keys. As a result, requests that omit the owner or purpose tag key (`$.Tags[*].Key`) are restricted, helping you identify each access point's owner and purpose. This implementation also restricts `elasticfilesystem:UntagResource` with the `aws:TagKeys` condition key; requests that remove the owner or purpose key from an access point are denied.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EFS.FileSystem.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.ReplicationConfiguration.Destinations[*].RoleArn`

**Implements:** [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)

**Scoping** (for [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)): This implementation resolves the replication role that `$.ReplicationConfiguration.Destinations[*].RoleArn` references and checks its trust policy, `$.Role.AssumeRolePolicyDocument`, before the file system is created. Rejecting templates that grant role assumption publicly or beyond your reference policy helps keep the role assumable only by trusted identities, including Amazon EFS.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EFS.FileSystem.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `efs:DescribeReplicationConfigurations`; check `$.Replications[*].SourceFileSystemId`, `$.Replications[*].Destinations[*].RoleArn`
2. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
3. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
4. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)

**Scoping** (for [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)): This AWS Config rule reads the replication role ARN at `$.Replications[*].Destinations[*].RoleArn` and retrieves that role's trust policy at `$.Role.AssumeRolePolicyDocument`. A `FAIL` verdict at `$.result` flags a trust policy that might allow public role assumption, helping you detect broad access to your replication role.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EFS.FileSystem.3

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `elasticfilesystem.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)

**Scoping** (for [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)): This implementation guidance recommends a trust policy that restricts assumption of your replication role (`$.Destinations[*].RoleArn`) to the `elasticfilesystem.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.EFS.FileSystem.4

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)

**Scoping** (for [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1)): This implementation governs `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping their requests with `aws:SourceOrgID`. As a result, principals outside your organization are restricted from assuming the replication role you pass in the `$.Destinations[*].RoleArn` request parameter.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.EFS.FileSystem.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.Encrypted`

**Implements:** [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2)

**Scoping** (for [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2)): This implementation validates that `$.Encrypted` is set to true at create time, requiring encryption at rest on your Amazon EFS file system. That check helps prevent a stack from provisioning a file system that stores data unencrypted.

### Implementation.EFS.FileSystem.6

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::FileSystem`: `$.configuration.Encrypted`, `$.configuration.KmsKeyId`

**Implements:** [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2)

**Scoping** (for [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2)): This implementation flags any EFS file system whose recorded `$.configuration.Encrypted` value is not true, so unencrypted file systems are surfaced rather than left undetected. It can also compare the recorded `$.configuration.KmsKeyId` against the KMS keys you approve for encrypting file data at rest.

### Implementation.EFS.FileSystem.7

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:CreateFileSystem` | `*` | `elasticfilesystem:Encrypted` |

**Implements:** [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2)

**Scoping** (for [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2)): This implementation governs `elasticfilesystem:CreateFileSystem` through the `elasticfilesystem:Encrypted` condition key, so file systems are created only with encryption at rest enabled. Because the encryption-at-rest setting `$.Encrypted` is fixed at creation, the create action is the only enforcement point.

### Implementation.EFS.FileSystem.8

*Guidance — Directive*

Consider implementing standard infrastructure security controls to restrict which networks and IP addresses can access your Amazon EFS file systems. These controls include security groups, network access control lists, and firewalls such as AWS Network Firewall. Security groups on your mount targets act as virtual firewalls for inbound NFS traffic. Network access control lists allow or deny that traffic at the subnet level. These infrastructure controls are effective only for client connections that traverse a mount target. Use the `elasticfilesystem:AccessedViaMountTarget` condition key in your file system policy to require that clients connect through a mount target. Requiring mount target access helps keep these network controls effective.

- [Control traffic to your AWS resources using security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [Control subnet traffic with network access control lists](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [AWS Network Firewall](https://aws.amazon.com/network-firewall/)

**Implements:** [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3)

**Scoping** (for [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3)): Your Amazon EFS file system serves shared file data to NFS clients over the network. Limiting which networks can reach that data addresses this goal for the file system.

### Implementation.EFS.FileSystem.9

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.FileSystemPolicy`

**Implements:** [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9), [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)

**Scoping** (for [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3)): This implementation validates the `$.FileSystemPolicy` document declared on your file system at stack create and update time. Requiring conditions such as `aws:SourceIp` there helps confine NFS client access to expected networks, so templates allowing broader access do not deploy.

**Scoping** (for [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)): This implementation validates at create time that `$.FileSystemPolicy` conditions NFS client access on `aws:SecureTransport`. A template that omits that condition does not deploy, which helps prevent clients from mounting your file system without TLS.

**Scoping** (for [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9)): This implementation requires `$.FileSystemPolicy` to declare an explicit file system policy; requests that create or update a file system without one are denied. Amazon EFS evaluates that IAM resource-based policy when authorizing NFS client actions such as `elasticfilesystem:ClientMount`.

**Data perimeter** (for [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3)): The create and update handlers for `AWS::EFS::FileSystem` apply the `$.FileSystemPolicy` property through `elasticfilesystem:PutFileSystemPolicy`, so the policy that controls NFS access is declared in your template. A hook on this resource type inspects that document before the file system is provisioned. Use it to keep developers from declaring a policy that grants access from unexpected networks.
- [AWS::EFS::FileSystem CloudFormation resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-efs-filesystem.html)
- [FileSystemPolicy property on AWS::EFS::FileSystem](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-efs-filesystem.html#cfn-efs-filesystem-filesystempolicy)

### Implementation.EFS.FileSystem.10

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `efs:PutFileSystemPolicy` — `$.Policy`

**Implements:** [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)

**Scoping** (for [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3)): This implementation records `PutFileSystemPolicy` calls together with the `$.Policy` file system policy document they submit. Watching that document helps you detect a rewrite that removes the network conditions restricting access to expected networks.

**Scoping** (for [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)): Monitoring `PutFileSystemPolicy` records each rewrite of the resource-based policy on your file system, including the policy document carried in `$.Policy`. This helps you detect and respond to a policy change that would grant untrusted identities access to your file system.

**Data perimeter:** The file system policy is where the `aws:SourceIp` condition key bounds which networks can reach your file system. Monitoring `PutFileSystemPolicy` management events surfaces each rewrite of that policy, including the submitted `$.Policy` document and the identity that made the call. Review those events, and remediate with the responsive controls of your choice when a change drops your network conditions.
- [PutFileSystemPolicy request syntax: the Policy parameter](https://docs.aws.amazon.com/efs/latest/APIReference/API_PutFileSystemPolicy.html#API_PutFileSystemPolicy_RequestSyntax)
- [Amazon EFS API Reference: PutFileSystemPolicy](https://docs.aws.amazon.com/efs/latest/APIReference/API_PutFileSystemPolicy.html)

**Data perimeter:** The identity perimeter on your file system depends on the statements in its resource-based policy. Watching `PutFileSystemPolicy` and the submitted `$.Policy` document gives you a record of each change and the identity that made it. You can then remediate with the responsive controls of your choice.
- [Policy request parameter in the Amazon EFS PutFileSystemPolicy request syntax](https://docs.aws.amazon.com/efs/latest/APIReference/API_PutFileSystemPolicy.html#API_PutFileSystemPolicy_RequestSyntax)
- [Amazon EFS PutFileSystemPolicy API reference](https://docs.aws.amazon.com/efs/latest/APIReference/API_PutFileSystemPolicy.html)

### Implementation.EFS.FileSystem.11

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:PutFileSystemPolicy` | `arn:aws:elasticfilesystem:*:*:file-system/*` | `aws:PrincipalTag/dp:exclude:identity` |

**Implements:** [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)

**Scoping** (for [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3)): This implementation governs `elasticfilesystem:PutFileSystemPolicy` through the `aws:PrincipalTag/dp:exclude:identity` condition key. Only the principals you select can then replace `$.Policy`, the file system policy where the `aws:SourceIp` network condition is evaluated.

**Scoping** (for [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)): This implementation governs `elasticfilesystem:PutFileSystemPolicy` through the `aws:PrincipalTag/dp:exclude:identity` condition key. Only selected identities can then change `$.Policy`, the file system policy that decides which identities may access your file system.

**Data perimeter:** Amazon EFS currently doesn't support RCPs. Restricting `elasticfilesystem:PutFileSystemPolicy` to selected principals is therefore how you preventively control who may grant access to your file systems. An unreviewed change to the file system policy can extend access to identities outside your organization.
- [Amazon EFS API Reference: PutFileSystemPolicy](https://docs.aws.amazon.com/efs/latest/APIReference/API_PutFileSystemPolicy.html)
- [Data perimeter example SCP: restrict resource-based policy configurations](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_resource_policy_configurations_scp.json)

**Data perimeter:** Amazon EFS evaluates network conditions such as `aws:SourceIp` inside the file system policy. That policy is therefore where the network perimeter for your file system is enforced. Amazon EFS currently doesn't support RCPs, so a service control policy restricts `elasticfilesystem:PutFileSystemPolicy` to select principals. The referenced example applies this pattern across resource-based policy configuration actions, exempting principals tagged with `aws:PrincipalTag/dp:exclude:identity`.
- [Amazon EFS API reference: PutFileSystemPolicy](https://docs.aws.amazon.com/efs/latest/APIReference/API_PutFileSystemPolicy.html)
- [Data perimeter policy example: restrict resource policy configurations (SCP)](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_resource_policy_configurations_scp.json)

### Implementation.EFS.FileSystem.12

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.BackupPolicy.Status`

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4)

**Scoping** (for [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4)): This implementation validates that `$.BackupPolicy.Status`, the file system's backup policy status, is set to `ENABLED` at create time. Requiring that value prevents provisioning a file system whose automatic backups are turned off.

### Implementation.EFS.FileSystem.13

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `backup:DeleteBackupVaultAccessPolicy` — `$.BackupVaultName`
- `backup:DeleteRecoveryPoint` — `$.BackupVaultName`, `$.RecoveryPointArn`
- `backup:PutBackupVaultAccessPolicy` — `$.BackupVaultName`, `$.Policy`
- `backup:UpdateBackupPlan` — `$.BackupPlan.Rules[*].Lifecycle.DeleteAfterDays`, `$.BackupPlan.Rules[*].ScheduleExpression`, `$.BackupPlanId`
- `efs:CreateFileSystem` — `$.Backup`
- `efs:PutBackupPolicy` — `$.BackupPolicy.Status` = `DISABLED`, `$.FileSystemId`

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4)

**Scoping** (for [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4)): This implementation records `PutBackupPolicy` setting `$.BackupPolicy.Status` to `DISABLED`, `CreateFileSystem` leaving `$.Backup` off, and the AWS Backup `UpdateBackupPlan`, `PutBackupVaultAccessPolicy`, `DeleteBackupVaultAccessPolicy`, and `DeleteRecoveryPoint` calls. Flagging them helps you detect lost or weakened backup coverage, so you can re-enable backups, restore plan retention, or reinstate the vault access policy.

### Implementation.EFS.FileSystem.14

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::FileSystem`: `$.configuration.BackupPolicy.Status`

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4)

**Scoping** (for [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4)): This implementation flags a file system whose `$.configuration.BackupPolicy.Status` does not show automatic backups turned on. That helps address the mitigation by revealing file systems with no scheduled recovery points available for restore.

### Implementation.EFS.FileSystem.15

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.EFS.FileSystem.5](#mitigationefsfilesystem5)

**Scoping** (for [Mitigation.EFS.FileSystem.5](#mitigationefsfilesystem5)): This implementation governs the Amazon EFS API calls your identities make against file systems with the `aws:ResourceOrgID` condition key. As a result, your identities are restricted from accessing file systems outside your organization when IAM evaluates the request.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.EFS.FileSystem.16

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `backup:DeleteBackupPlan` — `$.BackupPlanId`
- `backup:DeleteBackupSelection` — `$.BackupPlanId`, `$.SelectionId`
- `backup:DeleteBackupVaultAccessPolicy` — `$.BackupVaultName`
- `backup:DeleteRecoveryPoint` — `$.BackupVaultName`, `$.RecoveryPointArn`
- `backup:PutBackupVaultAccessPolicy` — `$.BackupVaultName`, `$.Policy`
- `backup:StopBackupJob` — `$.BackupJobId`
- `backup:UpdateBackupPlan` — `$.BackupPlan.Rules[*].Lifecycle.DeleteAfterDays`, `$.BackupPlan.Rules[*].ScheduleExpression`, `$.BackupPlanId`
- `efs:PutBackupPolicy` — `$.BackupPolicy.Status` = `DISABLED`, `$.FileSystemId`

**Implements:** [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

**Scoping** (for [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)): Recording Amazon EFS `PutBackupPolicy` calls helps you detect when your file system's automatic backups setting (`$.BackupPolicy.Status`) changes to `DISABLED`. Recording AWS Backup calls such as `DeleteBackupSelection` and `DeleteRecoveryPoint` helps you detect deleted resource assignments and recovery points for your file system.

### Implementation.EFS.FileSystem.17

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `backup:ListBackupPlans`; check `$.BackupPlansList[*].BackupPlanId`
2. Call `backup:ListBackupSelections` with `$.BackupPlanId`; check `$.BackupSelectionsList[*].SelectionId`
3. Call `backup:GetBackupSelection` with `$.BackupPlanId`, `$.SelectionId`; check `$.BackupSelection.Resources[*]`, `$.BackupSelection.NotResources[*]`, `$.BackupSelection.ListOfTags[*].ConditionKey`, `$.BackupSelection.ListOfTags[*].ConditionValue`, `$.BackupSelection.Conditions.StringEquals[*].ConditionKey`, `$.BackupSelection.Conditions.StringEquals[*].ConditionValue`, `$.BackupSelection.Conditions.StringLike[*].ConditionKey`, `$.BackupSelection.Conditions.StringLike[*].ConditionValue`, `$.BackupSelection.Conditions.StringNotEquals[*].ConditionKey`, `$.BackupSelection.Conditions.StringNotEquals[*].ConditionValue`, `$.BackupSelection.Conditions.StringNotLike[*].ConditionKey`, `$.BackupSelection.Conditions.StringNotLike[*].ConditionValue`

**Implements:** [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

**Scoping** (for [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)): This periodic check reads each backup plan's assigned resource ARNs in `$.BackupSelection.Resources[*]` and its tag-based selection conditions in `$.BackupSelection.ListOfTags[*].ConditionKey`. It flags any of your Amazon EFS file systems that no selection covers, so you can find those that no plan backs up.

### Implementation.EFS.FileSystem.18

*Declarative policy — Preventive*

Governs the `BACKUP_POLICY` declarative policy attributes below.

- `$.plans.EFS_Backup_Plan.selections.resources.efs_selection.resource_types`

**Implements:** [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

**Scoping** (for [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)): This declarative policy declares `$.plans.EFS_Backup_Plan.selections.resources.efs_selection.resource_types`, the resource types covered by the backup plan enforced across your accounts. Listing the Amazon EFS file system resource type assigns matched file systems to the plan for scheduled, centrally governed recovery points.

### Implementation.EFS.FileSystem.19

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.AvailabilityZoneName`, `$.ReplicationConfiguration.Destinations[*].AvailabilityZoneName`, `$.ReplicationConfiguration.Destinations[*].FileSystemId`

**Implements:** [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7)

**Scoping** (for [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7)): This implementation validates at create time that `$.AvailabilityZoneName` is absent, which selects the Regional file system type. A template pinning the file system to one Availability Zone is rejected, helping keep your data redundant across multiple Availability Zones.

### Implementation.EFS.FileSystem.20

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::FileSystem`: `$.configuration.FileSystemId`

**Implements:** [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7)

**Scoping** (for [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7)): This implementation resolves the recorded `$.configuration.FileSystemId` through `DescribeFileSystems` and flags any file system reporting `$.FileSystems[*].AvailabilityZoneName`. That field is populated only for One Zone file systems, so the check surfaces data held in a single Availability Zone.

### Implementation.EFS.FileSystem.21

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.Encrypted`, `$.KmsKeyId`, `$.ReplicationConfiguration.Destinations[*].KmsKeyId`

**Implements:** [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8)

**Scoping** (for [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8)): This hook requires your file system's encryption setting (`$.Encrypted`) to be true and its KMS key setting (`$.KmsKeyId`) to reference a customer managed key. In `FAIL` mode, CloudFormation stack operations fail if they would create an unencrypted file system or one using an AWS managed key.

### Implementation.EFS.FileSystem.22

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::FileSystem`: `$.configuration.Encrypted`, `$.configuration.KmsKeyId`

**Implements:** [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8)

**Scoping** (for [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8)): This implementation reads `$.configuration.Encrypted` and `$.configuration.KmsKeyId` from the file system, then checks `$.KeyMetadata.KeyManager` for the referenced key. It flags a file system that is unencrypted or uses an AWS managed key, helping you keep file data under a key you control.

### Implementation.EFS.FileSystem.23

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::FileSystem`: `$.configuration.FileSystemPolicy`

**Implements:** [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9), [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)

**Scoping** (for [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)): This implementation inspects `$.configuration.FileSystemPolicy` and flags a file system whose policy does not condition client access on `aws:SecureTransport`. That addresses the mitigation by surfacing file systems where NFS clients can still mount over an unencrypted connection.

**Scoping** (for [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9)): This rule flags an Amazon EFS file system whose resource-based file system policy (`$.configuration.FileSystemPolicy`) does not require IAM authorization for NFS clients. Flagging these file systems helps you detect where NFS clients may be granted `elasticfilesystem:ClientMount` or `elasticfilesystem:ClientWrite` without IAM authorization.

### Implementation.EFS.FileSystem.24

*Resource policy — Preventive*

Attaches to `AWS::EFS::FileSystem` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:ClientMount`, `elasticfilesystem:ClientRootAccess`, `elasticfilesystem:ClientWrite` | `arn:${Partition}:elasticfilesystem:${Region}:${Account}:file-system/${FileSystemId}` |  |
| `elasticfilesystem:CreateReplicationConfiguration`, `elasticfilesystem:DeleteReplicationConfiguration`, `elasticfilesystem:DescribeFileSystems`, `elasticfilesystem:DescribeReplicationConfigurations`, `elasticfilesystem:ReplicationRead`, `elasticfilesystem:ReplicationWrite` | `arn:${Partition}:elasticfilesystem:${Region}:${Account}:file-system/${FileSystemId}` | `aws:PrincipalAccount` |

**Implements:** [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10)

**Scoping** (for [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10)): This implementation grants `elasticfilesystem:ClientMount`, `elasticfilesystem:ClientWrite`, and `elasticfilesystem:ClientRootAccess` only to the identities you name in the file system policy at `$.Policy`. Clients you do not name receive no access from that policy, even when they can reach a mount target.

### Implementation.EFS.FileSystem.25

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.FileSystemPolicy`

**Implements:** [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10), [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12)

**Scoping** (for [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10)): This implementation validates the file system policy in `$.FileSystemPolicy` at create time with a public-access check. A policy that the check reports as possibly allowing public access is not deployed, helping you avoid granting access to all IAM principals.

**Scoping** (for [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12)): This implementation validates the file system policy your template declares in `$.FileSystemPolicy` for statements that grant unauthenticated access. Stack operations declaring such a policy are denied, helping keep NFS client access subject to IAM authorization.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.EFS.FileSystem.26

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::FileSystem`: `$.configuration.FileSystemPolicy`

**Implements:** [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10), [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12)

**Scoping** (for [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10)): This implementation submits the Amazon EFS file system policy recorded at `$.configuration.FileSystemPolicy` to an automated public-access check. A policy that could grant access to any principal is flagged as noncompliant, helping you limit client actions to a subset of IAM principals.

**Scoping** (for [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12)): This implementation reads the file system policy recorded at `$.configuration.FileSystemPolicy` and evaluates it with an IAM Access Analyzer public-access check for `AWS::EFS::FileSystem`. Flagging file systems whose public-access check result (`$.result`) is not `PASS` helps you detect policies that could grant access to any principal.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.EFS.FileSystem.27

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.ReplicationConfiguration.Destinations[*].Region`, `$.ReplicationConfiguration.Destinations[*].FileSystemId`, `$.ReplicationConfiguration.Destinations[*].AvailabilityZoneName`

**Implements:** [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11)

**Scoping** (for [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11)): This implementation validates at create time that `$.ReplicationConfiguration.Destinations[*].Region` names a Region other than the file system's own Region. Requiring that destination up front helps address the mitigation by keeping a synchronized copy of your file data outside a single Region.

### Implementation.EFS.FileSystem.28

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `efs:DescribeFileSystems`; check `$.FileSystems[*].FileSystemId`
2. Call `efs:DescribeReplicationConfigurations` with `$.FileSystemId`; check `$.Replications[*].SourceFileSystemId`, `$.Replications[*].SourceFileSystemRegion`, `$.Replications[*].Destinations[*].Region`, `$.Replications[*].Destinations[*].Status`

**Implements:** [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11)

**Scoping** (for [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11)): This periodic check compares `$.Replications[*].Destinations[*].Region` with `$.Replications[*].SourceFileSystemRegion` for each EFS file system. Flagging file systems without a different-Region destination helps address the mitigation by surfacing data that has no cross-Region replica.

### Implementation.EFS.FileSystem.29

*Resource policy — Preventive*

Attaches to `AWS::EFS::FileSystem` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:ClientMount`, `elasticfilesystem:ClientRootAccess`, `elasticfilesystem:ClientWrite` | `arn:${Partition}:elasticfilesystem:${Region}:${Account}:file-system/${FileSystemId}` |  |

**Implements:** [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12)

**Scoping** (for [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12)): Set your file system policy through `$.Policy` to grant `elasticfilesystem:ClientMount`, `elasticfilesystem:ClientWrite`, and `elasticfilesystem:ClientRootAccess` only to the IAM principals you specify. The policy then does not allow those actions for NFS clients that present no IAM credentials.

### Implementation.EFS.FileSystem.30

*Guidance — Directive*

Consider using AWS Identity and Access Management Access Analyzer external access analyzers with your Amazon EFS file systems. An external access analyzer helps identify file systems in your accounts that are shared with identities outside your zone of trust. Amazon EFS file systems are a supported resource type for external access analysis. The analyzer evaluates your file system policy, including its condition statements. It generates a finding when a principal outside your zone of trust can perform operations on the file system. Mounting a file system from an account outside your zone of trust also counts as external access. Choose your organization or your account as the zone of trust when you create an analyzer. External access analysis covers only the Region where the analyzer is enabled, so create one in each Region that holds file systems. Review each finding to confirm whether the access is intended. If necessary, remediate with the responsive controls of your choice.

- [AWS Identity and Access Management Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)

**Implements:** [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)

**Scoping** (for [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)): This recommendation directs you to identify file systems whose resource-based policy grants access to identities outside your organization or to a wildcard principal. Reviewing and remediating those grants helps limit file system access to trusted identities.

### Implementation.EFS.FileSystem.31

*Resource policy — Preventive*

Attaches to `AWS::EFS::FileSystem` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:ClientMount`, `elasticfilesystem:ClientRootAccess`, `elasticfilesystem:ClientWrite`, `elasticfilesystem:CreateReplicationConfiguration`, `elasticfilesystem:DeleteReplicationConfiguration`, `elasticfilesystem:DescribeFileSystems`, `elasticfilesystem:DescribeReplicationConfigurations`, `elasticfilesystem:ReplicationWrite` | `arn:${Partition}:elasticfilesystem:${Region}:${Account}:file-system/${FileSystemId}` |  |

**Implements:** [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)

**Scoping** (for [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)): Your Amazon EFS file system holds shared file data that clients mount over NFS. Because the file system can be mounted and shared across AWS accounts, identities outside your organization can reach that data.

### Implementation.EFS.FileSystem.32

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.FileSystemPolicy`

**Implements:** [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)

**Scoping** (for [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)): This implementation checks the file system policy your template sets in `$.FileSystemPolicy`, which helps keep your file system accessible only to trusted identities. Deployment stops when that policy can grant public access or access beyond your reference policy.

**Data perimeter** (for [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)): Your templates can declare the file system policy inline in the `$.FileSystemPolicy` property of `AWS::EFS::FileSystem`. This policy is the resource-based policy that controls NFS access to your file system. A hook on this resource type checks it before the file system is created. This helps you keep developers from granting access to identities outside your organization.
- [AWS::EFS::FileSystem CloudFormation resource reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-efs-filesystem.html)
- [FileSystemPolicy property on AWS::EFS::FileSystem](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-efs-filesystem.html#cfn-efs-filesystem-filesystempolicy)

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EFS.FileSystem.33

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::FileSystem`: `$.configuration.FileSystemPolicy`

**Implements:** [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)

**Scoping** (for [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13)): This rule flags the resource-based policy at `$.configuration.FileSystemPolicy` when it can allow public access or sensitive client actions on your Amazon EFS file system.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.

External access findings, generated by an external access analyzer, use automated reasoning. They identify resource-based policies that allow access from outside your zone of trust.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EFS.FileSystem.34

*Resource policy — Preventive*

Attaches to `AWS::EFS::FileSystem` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:*` | `arn:${Partition}:elasticfilesystem:${Region}:${Account}:file-system/${FileSystemId}` | `aws:SecureTransport` |

**Implements:** [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)

**Scoping** (for [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)): This implementation requires a file system policy that uses the `aws:SecureTransport` condition key to deny client access over unencrypted connections. Denying those connections helps protect your file data from being read while it moves over the network.

### Implementation.EFS.FileSystem.35

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:ClientMount`, `elasticfilesystem:ClientRootAccess`, `elasticfilesystem:ClientWrite` | `*` | `aws:SecureTransport` |

**Implements:** [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)

**Scoping** (for [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14)): This implementation governs `elasticfilesystem:ClientMount`, `elasticfilesystem:ClientWrite`, and `elasticfilesystem:ClientRootAccess` through the `aws:SecureTransport` condition key. As a result, principals in your organization are restricted from client access to your file systems over unencrypted connections.

### Implementation.EFS.FileSystem.36

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.FileSystemTags[*].Key`, `$.FileSystemTags[*].Value`

**Implements:** [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15)

**Scoping** (for [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15)): This implementation validates at create time that `$.FileSystemTags[*].Key` and `$.FileSystemTags[*].Value` declare your required owner and purpose metadata. Templates missing those tags fail before provisioning, helping keep each file system attributable to an owner and a business function.

### Implementation.EFS.FileSystem.37

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EFS::FileSystem`: `$.configuration.FileSystemTags[*].Key`, `$.configuration.FileSystemTags[*].Value`

**Implements:** [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15)

**Scoping** (for [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15)): This implementation flags EFS file systems whose recorded `$.configuration.FileSystemTags[*].Key` values omit your required owner and purpose tag keys. Detecting untagged file systems helps address the mitigation by keeping ownership and business function attributable during inventory review and investigations.

### Implementation.EFS.FileSystem.38

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `elasticfilesystem:CreateFileSystem` | `*` | `aws:RequestTag/owner` |
| `elasticfilesystem:CreateFileSystem` | `*` | `aws:RequestTag/purpose` |
| `elasticfilesystem:CreateFileSystem` | `*` | `aws:TagKeys` |
| `elasticfilesystem:CreateTags`, `elasticfilesystem:TagResource` | `arn:aws:elasticfilesystem:*:*:file-system/*` | `aws:RequestTag/owner` |
| `elasticfilesystem:CreateTags`, `elasticfilesystem:TagResource` | `arn:aws:elasticfilesystem:*:*:file-system/*` | `aws:RequestTag/purpose` |
| `elasticfilesystem:CreateTags`, `elasticfilesystem:TagResource` | `arn:aws:elasticfilesystem:*:*:file-system/*` | `aws:TagKeys` |
| `elasticfilesystem:DeleteTags`, `elasticfilesystem:UntagResource` | `arn:aws:elasticfilesystem:*:*:file-system/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15)

**Scoping** (for [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15)): This implementation governs `elasticfilesystem:CreateFileSystem` and `elasticfilesystem:TagResource` with the `aws:RequestTag/owner` and `aws:RequestTag/purpose` condition keys. Each request must carry both tag keys, helping you identify the owner and purpose of each file system. This implementation also restricts `elasticfilesystem:UntagResource` and `elasticfilesystem:DeleteTags` with the `aws:TagKeys` condition key, so requests that remove either key from a file system are denied.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EFS.FileSystem.39

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::FileSystem`: `$.BypassPolicyLockoutSafetyCheck`, `$.FileSystemPolicy`

**Implements:** [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16)

**Scoping** (for [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16)): This implementation validates `$.BypassPolicyLockoutSafetyCheck` and `$.FileSystemPolicy` at create and update time, rejecting templates that turn off the lockout safety check. Preserving that check helps keep policy-management access to `elasticfilesystem:PutFileSystemPolicy` in place, reducing the risk of an unmanageable file system.

### Implementation.EFS.FileSystem.40

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `efs:PutFileSystemPolicy` — `$.BypassPolicyLockoutSafetyCheck` = `true`, `$.FileSystemId`, `$.Policy`

**Implements:** [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16)

**Scoping** (for [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16)): This implementation flags `PutFileSystemPolicy` calls that set `$.BypassPolicyLockoutSafetyCheck`, helping you detect an attempt to leave your file system unmanageable. The recorded `$.Policy` and calling identity help you attribute a lockout after later policy calls are denied.

### Implementation.EFS.MountTarget.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::MountTarget`: `$.SubnetId`, `$.IpAddressType`

**Implements:** [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1)

**Scoping** (for [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1)): This implementation validates the mount target's subnet placement (`$.SubnetId`) by resolving that subnet's route table for a default route to an internet gateway. Create requests that place the NFS endpoint in a publicly routed subnet are denied.

### Implementation.EFS.MountTarget.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `efs:DescribeFileSystems`; check `$.FileSystems[*].FileSystemId`
2. Call `efs:DescribeMountTargets` with `$.FileSystemId`; check `$.MountTargets[*].MountTargetId`, `$.MountTargets[*].SubnetId`, `$.MountTargets[*].VpcId`, `$.MountTargets[*].LifeCycleState`
3. Call `ec2:DescribeRouteTables` with `$.Filter[*].Name` = `association.subnet-id`, `$.Filter[*].Value[*]`; check `$.routeTableSet[*].associationSet[*].subnetId`, `$.routeTableSet[*].associationSet[*].main`, `$.routeTableSet[*].routeSet[*].gatewayId`, `$.routeTableSet[*].routeSet[*].destinationCidrBlock` = `0.0.0.0/0`, `$.routeTableSet[*].routeSet[*].destinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1)

**Scoping** (for [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1)): This rule reads `$.MountTargets[*].SubnetId`, the subnet that holds your mount target, and resolves that subnet's associated route table. Flagging any mount target whose `$.routeTableSet[*].routeSet[*].gatewayId` shows an internet gateway on a default route helps you detect placement in a public subnet.

### Implementation.EFS.MountTarget.3

*Guidance — Directive*

Amazon EFS creates a requester-managed elastic network interface for each mount target, in the single subnet you supply. A mount target takes exactly one subnet. Only one mount target is allowed per Availability Zone. Multi-AZ network coverage therefore comes from creating several mount targets, not from a subnet array on one mount target. For a file system that uses Regional storage classes, create a mount target in every Availability Zone from which clients mount the file system. A subnet resides entirely within one Availability Zone, so the subnet you choose fixes the zone. Confirm that your chosen subnet IDs belong to distinct Availability Zones before you create each mount target. Check the `AvailabilityZone` response field of the Amazon EC2 `DescribeSubnets` operation. Standardize on infrastructure-as-code review so the templates and modules that create mount targets cover more than one Availability Zone. To verify the deployed state, list the file system's mount targets with `DescribeMountTargets`. Each mount target description returns `SubnetId`, `AvailabilityZoneName`, and `NetworkInterfaceId`. You can also enumerate the interfaces with the Amazon EC2 `DescribeNetworkInterfaces` operation. Confirm that the resulting interfaces span more than one Availability Zone. Spreading mount targets across zones helps keep file system connectivity available when a single Availability Zone is disrupted. A client that reaches the file system only through a mount target in a disrupted zone loses access. To assess and track multi-AZ posture at scale, include the file system in an AWS Resilience Hub application and run a resiliency assessment. The assessment flags single-AZ concentration against your resiliency policy. Note that a One Zone file system supports only one mount target, which must sit in the file system's own Availability Zone. This recommendation therefore applies to file systems that use Regional storage classes.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2)

**Scoping** (for [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2)): This guidance recommends a mount target in every Availability Zone your file system serves, so the interfaces Amazon EFS creates span zones. That placement helps address the mitigation by keeping client connectivity available when one Availability Zone is disrupted.

### Implementation.EFS.MountTarget.4

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `efs:DescribeFileSystems`; check `$.FileSystems[*].FileSystemId`, `$.FileSystems[*].AvailabilityZoneId`, `$.FileSystems[*].NumberOfMountTargets`
2. Call `efs:DescribeMountTargets` with `$.FileSystemId`; check `$.MountTargets[*].AvailabilityZoneId`, `$.MountTargets[*].SubnetId`, `$.MountTargets[*].LifeCycleState`

**Implements:** [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2)

**Scoping** (for [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2)): This periodic check reads each mount target's Availability Zone at `$.MountTargets[*].AvailabilityZoneId` and flags file systems served from a single zone. Because one requester-managed network interface backs each mount target, coverage in additional zones helps preserve client access when a zone becomes unavailable.

### Implementation.EFS.MountTarget.5

*Guidance — Directive*

Configure the security groups associated with your mount target following least-privilege best practices. Restrict both inbound and outbound rules to only the required sources and destinations, protocols, and ports. Sources and destinations can be IP address ranges, referenced security groups, or prefix lists. A mount target needs only inbound TCP access on NFS port 2049, from the clients that mount the file system. Where a client is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the client's network interfaces rather than addresses that can change. For other clients, use a narrow IP address range or a prefix list rather than an unrestricted address such as 0.0.0.0/0 or ::/0. If you omit security groups when you create a mount target, Amazon EFS attaches the VPC default security group. Set them explicitly instead. Use `ModifyMountTargetSecurityGroups` to change the set later. That operation replaces the existing groups rather than merging them.

To govern security groups at scale, use AWS Firewall Manager security group policies across an AWS Organization, scoped by account, organizational unit, or tag. A content audit policy audits inbound and outbound rules against an allow-list or block-list. It flags non-compliant groups in a central dashboard, notifies you, and can automatically remediate them. A usage audit policy removes unused or redundant groups. Note that Firewall Manager does not update security groups on requester-managed network interfaces, which back your mount targets. Scope content audit policies at the security groups themselves so the rules attached to your mount targets are still audited.

To verify unintended access permitted by your security groups, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended, such as any path reaching NFS port 2049 from an unrestricted address. Run an analysis with `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`, then review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3)

**Scoping** (for [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3)): Security groups attached to your mount target restrict inbound traffic to the clients that require NFS port 2049 and outbound traffic to required destinations. Neither direction permits traffic from or to unrestricted addresses such as 0.0.0.0/0 or ::/0, which helps reduce network exposure for the mount target.

### Implementation.EFS.MountTarget.6

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EFS::MountTarget`: `$.SecurityGroups[*]`

**Implements:** [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3)

**Scoping** (for [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3)): This hook resolves the groups referenced by `$.SecurityGroups[*]` and inspects their inbound rules in `$.securityGroupInfo[*].ipPermissions` and outbound rules in `$.securityGroupInfo[*].ipPermissionsEgress`. It fails the mount target's creation or update when a rule allows any address on unintended ports, directly or through a prefix list.

### Implementation.EFS.MountTarget.7

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `efs:DescribeFileSystems`; check `$.FileSystems[*].FileSystemId`
2. Call `efs:DescribeMountTargets` with `$.FileSystemId`; check `$.MountTargets[*].MountTargetId`, `$.MountTargets[*].LifeCycleState`
3. Call `efs:DescribeMountTargetSecurityGroups` with `$.MountTargetId`; check `$.SecurityGroups[*]`
4. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions`, `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
5. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3)

**Scoping** (for [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3)): This check reads the security groups in effect for each mount target at `$.SecurityGroups[*]`. It flags unrestricted `0.0.0.0/0` and `::/0` sources in the inbound rules at `$.securityGroupInfo[*].ipPermissions`, and unrestricted destinations in the outbound rules at `$.securityGroupInfo[*].ipPermissionsEgress`.

### ManagedControl.EFS.FileSystem.1

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require an Amazon EFS file system to encrypt file data at rest using AWS KMS

**Aliases:** `CT.ELASTICFILESYSYSTEM.PR.1`

**ARN:** `arn:aws:controlcatalog:::control/58jratszlp2w2z723ywcn6rmo`

This control checks whether an Amazon EFS file system is configured to encrypt file data using AWS KMS.

**Implements:** [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2)

### ManagedControl.EFS.FileSystem.2

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks whether Amazon EFS are configured to encrypt file data using AWS KMS

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.8`

**Identifier:** `EFS_ENCRYPTED_CHECK`

**ARN:** `arn:aws:controlcatalog:::control/8qmzz7l2841u6bvwpkiug38ta`

Checks whether Amazon EFS are configured to encrypt file data using AWS KMS. The rule is NON_COMPLIANT if the Encrypted key is set to False on DescribeFileSystems or, if specified, KmsKeyId key on DescribeFileSystems is not matching KmsKeyId parameter.

**Implements:** [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2), [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8)

### ManagedControl.EFS.FileSystem.3

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon Elastic File System (Amazon EFS) encrypts data with AWS Key Management Service (AWS KMS)

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.6`

**Identifier:** `EFS_FILESYSTEM_CT_ENCRYPTED`

**ARN:** `arn:aws:controlcatalog:::control/cigxd5pzgllvevu88sc1jgntd`

Checks if Amazon Elastic File System (Amazon EFS) encrypts data with AWS Key Management Service (AWS KMS). The rule is NON_COMPLIANT if a file system is not encrypted. Optionally, you can check if a file system is not encrypted with specified KMS keys.

**Implements:** [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2), [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8)

### ManagedControl.EFS.FileSystem.4

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if an Amazon Elastic File System (Amazon EFS) mount target is not associated with a public subnet

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.2`

**Identifier:** `EFS_MOUNT_TARGET_PUBLIC_ACCESSIBLE`

**ARN:** `arn:aws:controlcatalog:::control/nckpwoov47j6vyd0g113oq3l`

Checks if an Amazon Elastic File System (Amazon EFS) mount target is not associated with a public subnet. The rule is NON-COMPLIANT if the Amazon EFS mount target is associated with a public subnet.

**Implements:** [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3)

### ManagedControl.EFS.FileSystem.5

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if a recovery point was created for Amazon Elastic File System (Amazon EFS) File Systems

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.3`

**Identifier:** `EFS_LAST_BACKUP_RECOVERY_POINT_CREATED`

**ARN:** `arn:aws:controlcatalog:::control/3ji3udns2nasjgzmw8enf212s`

Checks if a recovery point was created for Amazon Elastic File System (Amazon EFS) File Systems. The rule is NON_COMPLIANT if the Amazon EFS File System does not have a corresponding Recovery Point created within the specified time period.

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

### ManagedControl.EFS.FileSystem.6

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if an Amazon Elastic File System (Amazon EFS) file system has automatic backups enabled

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.1`

**Identifier:** `EFS_AUTOMATIC_BACKUPS_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/5f52xzzr3c3pf47kumoyqcq08`

Checks if an Amazon Elastic File System (Amazon EFS) file system has automatic backups enabled. The rule is NON_COMPLIANT if `BackupPolicy.Status` is set to DISABLED.

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

### ManagedControl.EFS.FileSystem.7

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon Elastic File System (Amazon EFS) file systems are added in the backup plans of AWS Backup

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.9`

**Identifier:** `EFS_IN_BACKUP_PLAN`

**ARN:** `arn:aws:controlcatalog:::control/d7tno6cknpf28fwxv732m7kou`

Checks if Amazon Elastic File System (Amazon EFS) file systems are added in the backup plans of AWS Backup. The rule is NON_COMPLIANT if EFS file systems are not included in the backup plans.

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

### ManagedControl.EFS.FileSystem.8

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require an Amazon EFS volume to have an automated backup plan

**Aliases:** `CT.ELASTICFILESYSYSTEM.PR.2`

**ARN:** `arn:aws:controlcatalog:::control/dotzbwbqu9bwjk667tdz1s5gf`

This control checks whether your Amazon EFS file system has been configured with automatic backups through AWS Backup.

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

### ManagedControl.EFS.FileSystem.9

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon Elastic File System (Amazon EFS) File Systems are protected by a backup plan

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.4`

**Identifier:** `EFS_RESOURCES_PROTECTED_BY_BACKUP_PLAN`

**ARN:** `arn:aws:controlcatalog:::control/euj7xhh4c9psr5nhmrlsdu97j`

Checks if Amazon Elastic File System (Amazon EFS) File Systems are protected by a backup plan. The rule is NON_COMPLIANT if the EFS File System is not covered by a backup plan.

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

### ManagedControl.EFS.FileSystem.10

*AWS Config rule — Detective*

AWS managed control from General controls (service-agnostic — no governed resource).

**Title:** Checks if a recovery point expires no earlier than after the specified period

**Aliases:** `BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK`

**Identifier:** `BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK`

**ARN:** `arn:aws:controlcatalog:::control/m7a5gbdf08wg2o0en010mkng`

Checks if a recovery point expires no earlier than after the specified period. The rule is NON_COMPLIANT if the recovery point has a retention point that is less than the required retention period.

**Implements:** [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

### ManagedControl.EFS.FileSystem.11

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon Elastic File System (Amazon EFS) File Systems are in a logically air-gapped vault

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.5`

**Identifier:** `EFS_RESOURCES_IN_LOGICALLY_AIR_GAPPED_VAULT`

**ARN:** `arn:aws:controlcatalog:::control/b59mrft77jolzg42ezypsvn2x`

Checks if Amazon Elastic File System (Amazon EFS) File Systems are in a logically air-gapped vault. The rule is NON_COMPLIANT if an Amazon EFS File System is not in a logically air-gapped vault within the specified time period.

**Implements:** [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6)

### ManagedControl.EFS.FileSystem.12

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon Elastic File System file systems have tags

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.7`

**Identifier:** `EFS_FILE_SYSTEM_TAGGED`

**ARN:** `arn:aws:controlcatalog:::control/2i742gboo6ukis3a0benxxsrq`

Checks if Amazon Elastic File System file systems have tags. Optionally, you can specify tag keys. The rule is NON_COMPLIANT if there are no tags or if the specified tag keys are not present. The rule does not check for tags starting with 'aws:'.

**Implements:** [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15)

### ManagedControl.EFS.MountTarget.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if an Amazon Elastic File System (Amazon EFS) mount target is not associated with a public subnet

**Aliases:** `CONFIG.ELASTICFILESYSTEM.DT.2`

**Identifier:** `EFS_MOUNT_TARGET_PUBLIC_ACCESSIBLE`

**ARN:** `arn:aws:controlcatalog:::control/nckpwoov47j6vyd0g113oq3l`

Checks if an Amazon Elastic File System (Amazon EFS) mount target is not associated with a public subnet. The rule is NON-COMPLIANT if the Amazon EFS mount target is associated with a public subnet.

**Implements:** [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1)

---

## Appendix

### Resources

#### AccessPoint

*ACCESS_CONTROL*

An Amazon EFS access point is an application-specific entry point into an EFS file system. It applies a fixed operating system user and group, plus a file system path, to every request that arrives through it. Applications connect through the access point instead of mounting the file system root directly.

- IAM resource ARN: `arn:${Partition}:elasticfilesystem:${Region}:${Account}:access-point/${AccessPointId}`
- CloudFormation: `AWS::EFS::AccessPoint`
- AWS Config: `AWS::EFS::AccessPoint`
- 6 operations, 1 threats, 1 mitigations

#### AccountPreference

*CONFIGURATION*

Amazon EFS account preferences is an account-level setting, maintained separately in each AWS Region, that controls the identifier format assigned to newly created EFS file systems and mount targets. It selects between the long 17-character identifier format and the legacy short 8-character format.

- 2 operations, 0 threats, 0 mitigations

#### FileSystem

*DATA_STORE*

An Amazon EFS file system is a shared, elastic file store that Linux-based compute resources mount over the NFS protocol. It grows and shrinks automatically as files are added and removed, so no capacity is provisioned in advance. Many clients across Availability Zones can read and write the same data concurrently, which suits shared home directories, content repositories, analytics inputs, and container or serverless workloads that need durable shared state.

- IAM resource ARN: `arn:${Partition}:elasticfilesystem:${Region}:${Account}:file-system/${FileSystemId}`
- CloudFormation: `AWS::EFS::FileSystem`
- AWS Config: `AWS::EFS::FileSystem`
- 21 operations, 19 threats, 16 mitigations

#### MountTarget

*NETWORK*

An Amazon EFS mount target is the network entry point through which clients mount an EFS file system. Amazon EFS creates it in a subnet you choose and assigns it a private IP address from that subnet. Clients connect to that address over NFS to read and write file data.

- CloudFormation: `AWS::EFS::MountTarget`
- 5 operations, 5 threats, 3 mitigations

### Operations

| Operation | Analyses | Resources | Mitigations |
|---|---|---|---|
| `CreateAccessPoint` | DOCUMENTATION, DATA_PERIMETER | [AccessPoint](#accesspoint) | [Mitigation.EFS.2](#mitigationefs2), [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1) |
| `CreateFileSystem` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.2](#mitigationefs2), [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2), [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7), [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11), [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15) |
| `CreateMountTarget` | DOCUMENTATION, DATA_PERIMETER | [MountTarget](#mounttarget) | [Mitigation.EFS.2](#mitigationefs2), [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1), [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2), [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3) |
| `CreateReplicationConfiguration` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.1](#mitigationefs1), [Mitigation.EFS.2](#mitigationefs2), [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1), [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.5](#mitigationefsfilesystem5), [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7), [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11) |
| `CreateTags` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) |  |
| `DeleteAccessPoint` | DOCUMENTATION, DATA_PERIMETER | [AccessPoint](#accesspoint) |  |
| `DeleteFileSystem` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.1](#mitigationefs1), [Mitigation.EFS.2](#mitigationefs2), [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4), [Mitigation.EFS.FileSystem.5](#mitigationefsfilesystem5), [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11), [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13) |
| `DeleteFileSystemPolicy` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.2](#mitigationefs2), [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3) |
| `DeleteMountTarget` | DOCUMENTATION, DATA_PERIMETER | [MountTarget](#mounttarget) | [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2) |
| `DeleteReplicationConfiguration` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) |  |
| `DeleteTags` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15) |
| `DescribeAccessPoints` | DOCUMENTATION, DATA_PERIMETER | [AccessPoint](#accesspoint) | [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1) |
| `DescribeAccountPreferences` | DOCUMENTATION, DATA_PERIMETER | [AccountPreference](#accountpreference) |  |
| `DescribeBackupPolicy` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4) |
| `DescribeFileSystemPolicy` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9), [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10), [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12), [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13), [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14), [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16) |
| `DescribeFileSystems` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2), [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7), [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8), [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15) |
| `DescribeLifecycleConfiguration` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) |  |
| `DescribeMountTargetSecurityGroups` | DOCUMENTATION, DATA_PERIMETER | [MountTarget](#mounttarget) | [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3) |
| `DescribeMountTargets` | DOCUMENTATION, DATA_PERIMETER | [MountTarget](#mounttarget) | [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14), [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1), [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2), [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3) |
| `DescribeReplicationConfigurations` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1), [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11) |
| `DescribeTags` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) |  |
| `ListTagsForResource` | DOCUMENTATION, DATA_PERIMETER | [AccessPoint](#accesspoint), [FileSystem](#filesystem) | [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1), [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15) |
| `ModifyMountTargetSecurityGroups` | DOCUMENTATION, DATA_PERIMETER | [MountTarget](#mounttarget) | [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3) |
| `PutAccountPreferences` | DOCUMENTATION, DATA_PERIMETER | [AccountPreference](#accountpreference) |  |
| `PutBackupPolicy` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4) |
| `PutFileSystemPolicy` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.1](#mitigationefs1), [Mitigation.EFS.2](#mitigationefs2), [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3), [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9), [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10), [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12), [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13), [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14), [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16) |
| `PutLifecycleConfiguration` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) |  |
| `TagResource` | DOCUMENTATION, DATA_PERIMETER | [AccessPoint](#accesspoint), [FileSystem](#filesystem) | [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1), [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15) |
| `UntagResource` | DOCUMENTATION, DATA_PERIMETER | [AccessPoint](#accesspoint), [FileSystem](#filesystem) | [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1), [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15) |
| `UpdateFileSystem` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) | [Mitigation.EFS.1](#mitigationefs1), [Mitigation.EFS.2](#mitigationefs2), [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3) |
| `UpdateFileSystemProtection` | DOCUMENTATION, DATA_PERIMETER | [FileSystem](#filesystem) |  |

Mitigations attach to an operation directly (the operation appears in the mitigation's request parameters) or through a threat whose request parameters include the operation.

### Goal Coverage (21 of 38)

| Mitigation Goal | Covered | Mitigations |
|---|---|---|
| [AI Services Opt-Out Policy](../mitigation-goals.md#ai-services-opt-out-policy) | — |  |
| [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup) | yes | [Mitigation.EFS.FileSystem.6](#mitigationefsfilesystem6) |
| [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native) | yes | [Mitigation.EFS.FileSystem.4](#mitigationefsfilesystem4) |
| [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery) | — |  |
| [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2) | — |  |
| [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest) | yes | [Mitigation.EFS.FileSystem.2](#mitigationefsfilesystem2) |
| [Cross-Region Replication](../mitigation-goals.md#cross-region-replication) | yes | [Mitigation.EFS.FileSystem.11](#mitigationefsfilesystem11) |
| [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies) | — |  |
| [Customer Code Execution - Govern Code Content](../mitigation-goals.md#customer-code-execution---govern-code-content) | — |  |
| [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage) | — |  |
| [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection) | — |  |
| [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics) | — |  |
| [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics) | — |  |
| [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy) | yes | [Mitigation.EFS.FileSystem.14](#mitigationefsfilesystem14) |
| [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature) | — |  |
| [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication) | yes | [Mitigation.EFS.FileSystem.9](#mitigationefsfilesystem9) |
| [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network) | yes | [Mitigation.EFS.2](#mitigationefs2) |
| [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources) | yes | [Mitigation.EFS.FileSystem.13](#mitigationefsfilesystem13) |
| [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption) | yes | [Mitigation.EFS.FileSystem.8](#mitigationefsfilesystem8) |
| [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment) | yes | [Mitigation.EFS.FileSystem.7](#mitigationefsfilesystem7) |
| [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment) | yes | [Mitigation.EFS.MountTarget.2](#mitigationefsmounttarget2) |
| [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks) | — |  |
| [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks) | yes | [Mitigation.EFS.FileSystem.3](#mitigationefsfilesystem3) |
| [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement) | yes | [Mitigation.EFS.MountTarget.1](#mitigationefsmounttarget1) |
| [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets) | — |  |
| [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources) | yes | [Mitigation.EFS.FileSystem.5](#mitigationefsfilesystem5) |
| [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network) | yes | [Mitigation.EFS.1](#mitigationefs1) |
| [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition) | yes | [Mitigation.EFS.FileSystem.10](#mitigationefsfilesystem10) |
| [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention) | yes | [Mitigation.EFS.FileSystem.16](#mitigationefsfilesystem16) |
| [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose) | yes | [Mitigation.EFS.AccessPoint.1](#mitigationefsaccesspoint1), [Mitigation.EFS.FileSystem.15](#mitigationefsfilesystem15) |
| [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role) | yes | [Mitigation.EFS.FileSystem.1](#mitigationefsfilesystem1) |
| [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access) | yes | [Mitigation.EFS.MountTarget.3](#mitigationefsmounttarget3) |
| [Service Principal Presigned URL](../mitigation-goals.md#service-principal-presigned-url) | — |  |
| [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration) | — |  |
| [Turn on Block Public Access (BPA)](../mitigation-goals.md#turn-on-block-public-access-bpa) | — |  |
| [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention) | yes | [Mitigation.EFS.FileSystem.12](#mitigationefsfilesystem12) |
| [Use Nitro instances](../mitigation-goals.md#use-nitro-instances) | — |  |
| [VPC Deployment](../mitigation-goals.md#vpc-deployment) | — |  |

### Impact Rollup

| Impact | Threats |
|---|---|
| [Account Access Removal](../threat-impacts.md#account-access-removal) | [Threat.EFS.FileSystem.2](#threatefsfilesystem2) |
| [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery) | [Threat.EFS.FileSystem.18](#threatefsfilesystem18), [Threat.EFS.FileSystem.19](#threatefsfilesystem19), [Threat.EFS.MountTarget.3](#threatefsmounttarget3), [Threat.EFS.MountTarget.4](#threatefsmounttarget4) |
| [Data Collection](../threat-impacts.md#data-collection) | [Threat.EFS.1](#threatefs1), [Threat.EFS.2](#threatefs2), [Threat.EFS.FileSystem.6](#threatefsfilesystem6), [Threat.EFS.FileSystem.8](#threatefsfilesystem8), [Threat.EFS.FileSystem.9](#threatefsfilesystem9), [Threat.EFS.FileSystem.10](#threatefsfilesystem10), [Threat.EFS.FileSystem.14](#threatefsfilesystem14), [Threat.EFS.FileSystem.16](#threatefsfilesystem16), [Threat.EFS.FileSystem.18](#threatefsfilesystem18), [Threat.EFS.FileSystem.19](#threatefsfilesystem19) |
| [Data Destruction](../threat-impacts.md#data-destruction) | [Threat.EFS.FileSystem.1](#threatefsfilesystem1), [Threat.EFS.FileSystem.2](#threatefsfilesystem2), [Threat.EFS.FileSystem.3](#threatefsfilesystem3), [Threat.EFS.FileSystem.7](#threatefsfilesystem7), [Threat.EFS.FileSystem.13](#threatefsfilesystem13), [Threat.EFS.FileSystem.14](#threatefsfilesystem14), [Threat.EFS.FileSystem.15](#threatefsfilesystem15) |
| [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Threat.EFS.FileSystem.4](#threatefsfilesystem4) |
| [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Threat.EFS.1](#threatefs1), [Threat.EFS.2](#threatefs2), [Threat.EFS.FileSystem.7](#threatefsfilesystem7), [Threat.EFS.FileSystem.8](#threatefsfilesystem8), [Threat.EFS.FileSystem.9](#threatefsfilesystem9), [Threat.EFS.FileSystem.10](#threatefsfilesystem10), [Threat.EFS.FileSystem.12](#threatefsfilesystem12), [Threat.EFS.FileSystem.13](#threatefsfilesystem13), [Threat.EFS.FileSystem.14](#threatefsfilesystem14), [Threat.EFS.FileSystem.16](#threatefsfilesystem16) |
| [Defense Evasion](../threat-impacts.md#defense-evasion) | [Threat.EFS.AccessPoint.1](#threatefsaccesspoint1), [Threat.EFS.FileSystem.5](#threatefsfilesystem5), [Threat.EFS.FileSystem.6](#threatefsfilesystem6), [Threat.EFS.FileSystem.12](#threatefsfilesystem12) |
| [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Threat.EFS.FileSystem.11](#threatefsfilesystem11), [Threat.EFS.MountTarget.1](#threatefsmounttarget1), [Threat.EFS.MountTarget.5](#threatefsmounttarget5) |
| [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Threat.EFS.FileSystem.4](#threatefsfilesystem4) |
| [Initial Access](../threat-impacts.md#initial-access) | [Threat.EFS.FileSystem.7](#threatefsfilesystem7), [Threat.EFS.FileSystem.12](#threatefsfilesystem12), [Threat.EFS.FileSystem.13](#threatefsfilesystem13), [Threat.EFS.FileSystem.14](#threatefsfilesystem14), [Threat.EFS.FileSystem.16](#threatefsfilesystem16), [Threat.EFS.FileSystem.17](#threatefsfilesystem17), [Threat.EFS.MountTarget.3](#threatefsmounttarget3), [Threat.EFS.MountTarget.4](#threatefsmounttarget4) |
| [Lateral Movement](../threat-impacts.md#lateral-movement) | [Threat.EFS.MountTarget.2](#threatefsmounttarget2) |
| [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Threat.EFS.FileSystem.17](#threatefsfilesystem17) |
