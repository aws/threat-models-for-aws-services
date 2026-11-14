# Amazon Elastic Kubernetes Service (eks) Threat Model

[Threat Summary](#threat-summary) · [Threats](#threats) · [Mitigations](#mitigations) · [Implementations](#implementations) · [Appendix](#appendix)

## Overview

Amazon Elastic Kubernetes Service is a managed Kubernetes service for running containerized applications in the AWS cloud and in on-premises environments. AWS operates a dedicated, highly available Kubernetes control plane for each cluster and handles its scaling, patching, and recovery. Because the service is conformant with upstream Kubernetes, existing applications and community tooling can typically run without refactoring.

64 threats and 45 mitigations across 10 resource types and 70 API operations. 20 of 38 mitigation goals are covered, implemented by 122 implementations.

Source service model: [eks](https://github.com/aws/api-models-aws/blob/3d89dd029ebd13b775ab5d22e5075399f20b35e4/models/eks/service/2017-11-01/eks-2017-11-01.json)

---

## Threat Summary

### [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.22](#threatekscluster22) | Threat actor, with control of an Amazon EKS cluster configured with overly permissive outbound security group rules, can use the cluster to interact with ungoverned network destinations. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Cluster.13](#mitigationekscluster13) |
| [Threat.EKS.Nodegroup.9](#threateksnodegroup9) | Threat actor, with control of an Amazon EKS cluster's nodegroup configured with overly permissive outbound security group rules, can use the nodegroup to interact with ungoverned network destinations. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7) |

### [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.4](#threatekscluster4) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster, can encrypt the Kubernetes API data, such as Secrets and ConfigMaps, with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.EKS.Cluster.9](#mitigationekscluster9) |

### [Data loss through resource deletion](../potential-threats.md#data-loss-through-resource-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.1](#threatekscluster1) | IAM principal, with IAM permissions to delete an Amazon EKS cluster, can delete the cluster and the Kubernetes objects that the cluster contains. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.EKS.Cluster.8](#mitigationekscluster8), [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |

### [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Addon.3](#threateksaddon3) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's addon that supports Container Insights with enhanced observability or Container Network Observability performance metrics, can create or configure the cluster's addon without Container Insights with enhanced observability or Container Network Observability performance metrics enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.6](#mitigationeksaddon6) |
| [Threat.EKS.Cluster.8](#threatekscluster8) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster that supports EKS Auto Mode compute and its EKS node monitoring agent, can create or configure the cluster without EKS Auto Mode compute and its EKS node monitoring agent enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Cluster.5](#mitigationekscluster5) |

### [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Addon.6](#threateksaddon6) | IAM principal, with IAM permissions to use an Amazon EKS cluster, can deploy Pods that initiate network connections without a record of those connections being persisted by Amazon VPC CNI network policy logging. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Addon.1](#mitigationeksaddon1) |
| [Threat.EKS.Capability.5](#threatekscapability5) | IAM principal, with IAM permissions to use an Amazon EKS cluster's capability, can interact with the capability, causing its controllers to act on their behalf, without a record of that controller activity being persisted by vended logs delivery of its EKS_CAPABILITY_ACK_LOGS, EKS_CAPABILITY_KRO_LOGS, or EKS_CAPABILITY_ARGOCD_* controller logs. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Capability.4](#mitigationekscapability4) |
| [Threat.EKS.Cluster.19](#threatekscluster19) | IAM principal, with IAM permissions to use an Amazon EKS cluster, can interact with the Kubernetes API without a record of their activities being persisted by control plane logging. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Cluster.1](#mitigationekscluster1) |

### [Direct data deletion](../potential-threats.md#direct-data-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.14](#threatekscluster14) | IAM principal, with Kubernetes access granted through an Amazon EKS access entry or the aws-auth ConfigMap that permits deleting Kubernetes objects, such as Secrets, ConfigMaps, workloads and PersistentVolumeClaims, from an Amazon EKS cluster, can delete Kubernetes objects, such as Secrets, ConfigMaps, workloads and PersistentVolumeClaims, or whole namespaces from the cluster. | [Data Destruction](../threat-impacts.md#data-destruction) | [Mitigation.EKS.Cluster.8](#mitigationekscluster8) |

### [Flawed authentication model problem](../potential-threats.md#flawed-authentication-model-problem)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Capability.2](#threatekscapability2) | Threat actor, with knowledge of a flawed AWS IAM Identity Center's external provider configuration, can take advantage of the flawed AWS IAM Identity Center's external provider configuration to gain unauthorized access to the Amazon EKS cluster's capability. | [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Capability.2](#mitigationekscapability2) |
| [Threat.EKS.Cluster.6](#threatekscluster6) | Threat actor, with knowledge of a flawed access entry or aws-auth ConfigMap mapping, can take advantage of the flawed access entry or aws-auth ConfigMap mapping to gain unauthorized access to the Amazon EKS cluster. | [Initial Access](../threat-impacts.md#initial-access), [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Mitigation.EKS.Cluster.10](#mitigationekscluster10) |
| [Threat.EKS.IdentityProviderConfig.2](#threateksidentityproviderconfig2) | Threat actor, with knowledge of a flawed OIDC identity provider configuration, can take advantage of the flawed OIDC identity provider configuration to gain unauthorized access to the Amazon EKS cluster. | [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1) |

### [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.1](#threateks1) | Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon EKS service API calls. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.2](#mitigationeks2) |

### [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.13](#threatekscluster13) | Untrusted IAM principal, that has been granted access to my Amazon EKS cluster through its access entries, can use my Amazon EKS cluster. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access), [Malicious Code Execution](../threat-impacts.md#malicious-code-execution) | [Mitigation.EKS.Cluster.11](#mitigationekscluster11) |

### [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Addon.4](#threateksaddon4) | Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's addon's Pod Identity IAM role through its trust policy, can assume my Pod Identity IAM role and act with its permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Addon.3](#mitigationeksaddon3) |
| [Threat.EKS.Capability.4](#threatekscapability4) | Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's capability's IAM role through its trust policy, can assume my capability IAM role and act with its permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access), [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.EKS.Capability.1](#mitigationekscapability1) |
| [Threat.EKS.Cluster.17](#threatekscluster17) | Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's service role, Auto Mode node IAM role, and connector agent IAM role through their trust policies, can assume my service role, Auto Mode node IAM role, and connector agent IAM role and act with their permissions. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Cluster.3](#mitigationekscluster3) |
| [Threat.EKS.FargateProfile.3](#threateksfargateprofile3) | Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's Fargate profile's IAM Pod execution role through its trust policy, can assume my Pod execution role and act with its permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2) |
| [Threat.EKS.Nodegroup.7](#threateksnodegroup7) | Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's nodegroup's node IAM role through its trust policy, can assume my node IAM role and act with its permissions. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2) |
| [Threat.EKS.PodIdentityAssociation.2](#threatekspodidentityassociation2) | Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's pod identity association's IAM EKS Pod Identity role through its trust policy, can assume my EKS Pod Identity role and act with its permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2) |

### [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.11](#threatekscluster11) | Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon EKS cluster, can interact with the cluster's private Kubernetes API server endpoint through overly permissive network rules and move laterally within the VPC. | [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.EKS.Cluster.13](#mitigationekscluster13) |
| [Threat.EKS.Nodegroup.3](#threateksnodegroup3) | Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon EKS cluster's nodegroup, can interact with the nodegroup's worker node network interfaces through overly permissive network rules and move laterally within the VPC. | [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7) |

### [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.7](#threatekscluster7) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's Kubernetes API data, can encrypt the Kubernetes API data with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Cluster.9](#mitigationekscluster9) |

### [Logging disablement](../potential-threats.md#logging-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Addon.7](#threateksaddon7) | IAM principal, with IAM permissions to modify an Amazon EKS cluster's addon, can disable network policy logging for the addon. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Addon.1](#mitigationeksaddon1) |
| [Threat.EKS.Capability.6](#threatekscapability6) | IAM principal, with IAM permissions to modify the log delivery configuration for an Amazon EKS capability, can disable controller log delivery for the capability. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Capability.4](#mitigationekscapability4) |
| [Threat.EKS.Cluster.21](#threatekscluster21) | IAM principal, with IAM permissions to modify an Amazon EKS cluster, can disable control plane logging, including the audit and authenticator log types, for the cluster. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Cluster.1](#mitigationekscluster1) |

### [Monitoring disablement](../potential-threats.md#monitoring-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Addon.1](#threateksaddon1) | IAM principal, with IAM permissions to modify an Amazon EKS cluster's addon, can disable Container Insights with enhanced observability, node health monitoring, or Prometheus metrics for the addon. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.6](#mitigationeksaddon6) |
| [Threat.EKS.Cluster.3](#threatekscluster3) | IAM principal, with IAM permissions to modify an Amazon EKS cluster, can disable the EKS node monitoring agent on EKS Auto Mode nodes for the cluster. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Cluster.5](#mitigationekscluster5) |

### [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.12](#threatekscluster12) | Threat actor, with network access to an Amazon EKS cluster that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the cluster. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Cluster.13](#mitigationekscluster13) |
| [Threat.EKS.Nodegroup.4](#threateksnodegroup4) | Threat actor, with network access to an Amazon EKS cluster's nodegroup that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the nodegroup. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7) |

### [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Addon.5](#threateksaddon5) | Threat actor, with possession of credentials for my Amazon EKS cluster's addon's EKS Pod Identity IAM role or IAM role for service accounts, can use my EKS Pod Identity IAM role or IAM role for service accounts while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.Addon.2](#mitigationeksaddon2) |
| [Threat.EKS.Cluster.18](#threatekscluster18) | Threat actor, with possession of credentials for my Amazon EKS cluster's Auto Mode node IAM role or my registered external cluster's EKS Connector agent IAM role, can use my Auto Mode node IAM role or EKS Connector agent IAM role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.Cluster.2](#mitigationekscluster2) |
| [Threat.EKS.FargateProfile.4](#threateksfargateprofile4) | Threat actor, with possession of credentials for my Amazon EKS cluster's fargate profile's Pod execution role, can use my Pod execution role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.FargateProfile.1](#mitigationeksfargateprofile1) |
| [Threat.EKS.Nodegroup.8](#threateksnodegroup8) | Threat actor, with possession of credentials for my Amazon EKS cluster's nodegroup's node IAM role, can use my node IAM role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.Nodegroup.1](#mitigationeksnodegroup1) |
| [Threat.EKS.PodIdentityAssociation.3](#threatekspodidentityassociation3) | Threat actor, with possession of credentials for my Amazon EKS cluster's pod identity association's EKS Pod Identity role or, when configured, its target IAM role, can use my EKS Pod Identity role or, when configured, its target IAM role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1) |

### [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.15](#threatekscluster15) | Threat actor, with network access to an Amazon EKS cluster that has been placed in a public subnet, can access the cluster directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Cluster.4](#mitigationekscluster4) |
| [Threat.EKS.Nodegroup.5](#threateksnodegroup5) | Threat actor, with network access to an Amazon EKS cluster's nodegroup that has been placed in a public subnet, can access the nodegroup directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3) |

### [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.AccessEntry.1](#threateksaccessentry1) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's access entry, can configure the access entry without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1) |
| [Threat.EKS.Addon.2](#threateksaddon2) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's addon, can configure the addon without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Addon.5](#mitigationeksaddon5) |
| [Threat.EKS.Capability.1](#threatekscapability1) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's capability, can configure the capability without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Capability.3](#mitigationekscapability3) |
| [Threat.EKS.Cluster.5](#threatekscluster5) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster, can configure the cluster without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Cluster.12](#mitigationekscluster12) |
| [Threat.EKS.EksAnywhereSubscription.1](#threatekseksanywheresubscription1) | IAM principal, with IAM permissions to create or modify an Amazon EKS eks anywhere subscription, can configure the eks anywhere subscription without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1) |
| [Threat.EKS.FargateProfile.1](#threateksfargateprofile1) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's fargate profile, can configure the fargate profile without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5) |
| [Threat.EKS.IdentityProviderConfig.1](#threateksidentityproviderconfig1) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's identity provider config, can configure the identity provider config without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2) |
| [Threat.EKS.Nodegroup.1](#threateksnodegroup1) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's nodegroup, can configure the nodegroup without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6) |
| [Threat.EKS.PodIdentityAssociation.1](#threatekspodidentityassociation1) | IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's pod identity association, can configure the pod identity association without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3) |

### [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.2](#threateks2) | IAM principal, with IAM permissions on untrusted Amazon EKS resources, can interact with those untrusted resources from a governed VPC network. | [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.1](#mitigationeks1) |

### [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.10](#threatekscluster10) | My IAM principal, that has been granted access to an untrusted Amazon EKS cluster through an access entry created by that cluster's owner, can interact with the untrusted cluster over its Kubernetes API server endpoint, including sending workloads and data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.Cluster.7](#mitigationekscluster7) |

### [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.FargateProfile.2](#threateksfargateprofile2) | IAM principal, with IAM permissions to create an Amazon EKS cluster's fargate profile, can create the fargate profile in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4) |
| [Threat.EKS.Nodegroup.2](#threateksnodegroup2) | IAM principal, with IAM permissions to create an Amazon EKS cluster's nodegroup, can create the nodegroup in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5) |

### [Service disruption through resource deletion](../potential-threats.md#service-disruption-through-resource-deletion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.2](#threatekscluster2) | IAM principal, with IAM permissions to delete an Amazon EKS cluster, can delete the cluster. | [Service Stop](../threat-impacts.md#service-stop) | [Mitigation.EKS.Cluster.8](#mitigationekscluster8), [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |

### [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.24](#threatekscluster24) | Threat actor, with access to an Amazon EKS cluster, can submit a volume of requests that exceeds the capacity of the cluster or the network it is connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.EKS.Cluster.4](#mitigationekscluster4) |
| [Threat.EKS.FargateProfile.6](#threateksfargateprofile6) | Threat actor, with network access to the Fargate Pods that an Amazon EKS cluster's fargate profile places in its subnets, can submit a volume of requests that exceeds the capacity of the Fargate Pods that the fargate profile places in its subnets or the network they are connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3) |
| [Threat.EKS.Nodegroup.11](#threateksnodegroup11) | Threat actor, with access to an Amazon EKS cluster's nodegroup, can submit a volume of requests that exceeds the capacity of the nodegroup or the network it is connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service), [Network Denial of Service](../threat-impacts.md#network-denial-of-service) | [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3) |

### [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.9](#threatekscluster9) | IAM principal, with unintended IAM permissions to read Kubernetes Secrets and other Kubernetes API data that an access entry or RBAC binding lets it read from an Amazon EKS cluster where the service uses grant-based KMS authorization, can read Kubernetes Secrets and other Kubernetes API data in the cluster because the KMS key policy cannot restrict which IAM principals can decrypt the data. | [Credential Theft](../threat-impacts.md#credential-theft), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.EKS.Cluster.4](#mitigationekscluster4), [Mitigation.EKS.Cluster.11](#mitigationekscluster11), [Mitigation.EKS.Cluster.13](#mitigationekscluster13) |

### [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.23](#threatekscluster23) | Threat actor, who can control the outbound connections of an Amazon EKS cluster using customer-routed control plane egress that is attached to a subnet with a route to an internet gateway, can send the cluster's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Cluster.4](#mitigationekscluster4) |
| [Threat.EKS.FargateProfile.5](#threateksfargateprofile5) | Threat actor, who can execute code in Fargate Pods scheduled through an Amazon EKS cluster's fargate profile that is attached to a subnet with a route to an internet gateway, can send the Pods' outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3) |
| [Threat.EKS.Nodegroup.10](#threateksnodegroup10) | Threat actor, who can execute code in an Amazon EKS cluster's nodegroup that is attached to a subnet with a route to an internet gateway, can send the nodegroup's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3) |

### [Unencrypted instance-to-instance traffic through instance type selection](../potential-threats.md#unencrypted-instance-to-instance-traffic-through-instance-type-selection)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.16](#threatekscluster16) | IAM principal, with IAM permissions to create or update an Amazon EKS cluster's control plane instance and etcd instance, can configure the control plane instance and etcd instance to use an instance type that does not provide automatic encryption of instance-to-instance traffic. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.EKS.Cluster.6](#mitigationekscluster6) |
| [Threat.EKS.Nodegroup.6](#threateksnodegroup6) | IAM principal, with IAM permissions to create or update an Amazon EKS cluster's nodegroup, can configure the nodegroup to use an instance type that does not provide automatic encryption of instance-to-instance traffic. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4) |

### [Vulnerable identity provider problem](../potential-threats.md#vulnerable-identity-provider-problem)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Capability.3](#threatekscapability3) | Threat actor, with knowledge of a vulnerable external identity provider configured in an AWS IAM Identity Center instance used by an Amazon EKS cluster's capability, can gain federated access to the capability's hosted Argo CD UI and API. | [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Capability.2](#mitigationekscapability2) |
| [Threat.EKS.IdentityProviderConfig.3](#threateksidentityproviderconfig3) | Threat actor, with knowledge of a vulnerable Amazon EKS cluster's identity provider config, can gain federated access to the Kubernetes API server. | [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1) |

### [Weak authentication problem](../potential-threats.md#weak-authentication-problem)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.EKS.Cluster.20](#threatekscluster20) | Threat actor, in possession of a long-lived, static Kubernetes service account token issued for an Amazon EKS cluster, can take advantage of the token to access the cluster. | [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.EKS.Cluster.10](#mitigationekscluster10) |

---

## Threats

Each scoping statement below describes the exposure the threat's mapped mitigations address. It assumes those mitigations are not applied and makes no claim about your resources.

### Threat.EKS.1

Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon EKS service API calls.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.2](#mitigationeks2)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)): Amazon EKS is reachable from your VPC through an interface VPC endpoint (powered by AWS PrivateLink), for cluster management API calls. An IAM principal from outside your organization can use that network path to make Amazon EKS API calls with its own credentials.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.configurationValues` |
| `eks:UpdateAddon` | `$.configurationValues` |

### Threat.EKS.2

IAM principal, with IAM permissions on untrusted Amazon EKS resources, can interact with those untrusted resources from a governed VPC network.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.1](#mitigationeks1)

**Scoping** (for [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)): Amazon EKS API calls from your VPCs can target Amazon EKS resources in another account. An IAM principal in that network with permissions on Amazon EKS resources outside your organization can call those operations against them.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UpdateAddon` | `$.clusterName`, `$.configurationValues` |

### Threat.EKS.AccessEntry.1

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's access entry, can configure the access entry without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS cluster's access entry can omit the required tags, or remove them through `UntagResource`. An Amazon EKS cluster's access entry left without those tags still grants cluster access and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateAccessEntry` | `$.clusterName`, `$.principalArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.EKS.Addon.1

IAM principal, with IAM permissions to modify an Amazon EKS cluster's addon, can disable Container Insights with enhanced observability, node health monitoring, or Prometheus metrics for the addon.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.6](#mitigationeksaddon6)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): Your Amazon EKS cluster's addon can provide Container Insights with enhanced observability, node health monitoring, or Prometheus metrics beyond the basic `AWS/EKS` metrics. An IAM principal with permission to modify the addon can disable this monitoring, stopping the detailed metrics and node health status it reports.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:DeleteAddon` | `$.addonName`, `$.clusterName`, `$.preserve` |
| `eks:UpdateAddon` | `$.addonName`, `$.clusterName`, `$.configurationValues` |

### Threat.EKS.Addon.2

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's addon, can configure the addon without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Addon.5](#mitigationeksaddon5)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS cluster's addon can omit the required tags, or remove them through `UntagResource`. An addon left without those tags still runs on your cluster and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.addonName`, `$.clusterName`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.EKS.Addon.3

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's addon that supports Container Insights with enhanced observability or Container Network Observability performance metrics, can create or configure the cluster's addon without Container Insights with enhanced observability or Container Network Observability performance metrics enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.6](#mitigationeksaddon6)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): An IAM principal can create or update your cluster's addon with `$.configurationValues` that leave Container Insights with enhanced observability or Container Network Observability performance metrics disabled. Without node-, pod-, container- and accelerator-level metrics or network performance metrics, anomalous resource use, such as a cryptomining burst or unexpected traffic, is harder to detect.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.addonName`, `$.addonVersion`, `$.configurationValues` |
| `eks:UpdateAddon` | `$.addonName`, `$.configurationValues` |

### Threat.EKS.Addon.4

Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's addon's Pod Identity IAM role through its trust policy, can assume my Pod Identity IAM role and act with its permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Addon.3](#mitigationeksaddon3)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon EKS cluster's addon designates an optional Pod Identity IAM role through `$.podIdentityAssociations[*].roleArn`, and you author its trust policy. That trust policy can allow identities outside your organization to assume the role and act with its permissions, including your addon's access to AWS resources.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.podIdentityAssociations[*].roleArn` |
| `eks:UpdateAddon` | `$.podIdentityAssociations[*].roleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.EKS.Addon.5

Threat actor, with possession of credentials for my Amazon EKS cluster's addon's EKS Pod Identity IAM role or IAM role for service accounts, can use my EKS Pod Identity IAM role or IAM role for service accounts while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.Addon.2](#mitigationeksaddon2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): When you assign an EKS Pod Identity IAM role or an IAM role for service accounts to your add-on, its pods receive temporary credentials for that role. A threat actor who obtains those credentials can use either role while on an unexpected network.

**Request parameters**

| Operation | Parameters |
|---|---|
| `s3:GetObject` | `$.Bucket`, `$.Key` |

### Threat.EKS.Addon.6

IAM principal, with IAM permissions to use an Amazon EKS cluster, can deploy Pods that initiate network connections without a record of those connections being persisted by Amazon VPC CNI network policy logging.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Addon.1](#mitigationeksaddon1)

**Protocols:** `TCP`, `UDP`

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When Amazon VPC CNI network policy logging is not configured on your Amazon EKS cluster's addon, no connection verdict records are written by Amazon VPC CNI network policy logging. An IAM principal with permissions to use the Amazon EKS cluster can deploy Pods that initiate network connections to other Pods and to endpoints outside the cluster, leaving no record of those connections persisted by the VPC CNI network policy logging.

### Threat.EKS.Addon.7

IAM principal, with IAM permissions to modify an Amazon EKS cluster's addon, can disable network policy logging for the addon.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Addon.1](#mitigationeksaddon1)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to modify your Amazon EKS cluster's addon can call `UpdateAddon` with `$.configurationValues` that turn off `nodeAgent.enablePolicyEventLogs` or `nodeAgent.enableCloudWatchLogs`. Either change stops the `vpc-cni` node agent's opt-in network policy logs from reaching CloudWatch Logs while pod traffic continues under your network policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:DeleteAddon` | `$.addonName`, `$.clusterName`, `$.preserve` |
| `eks:UpdateAddon` | `$.addonName`, `$.clusterName`, `$.configurationValues`, `$.resolveConflicts` |

### Threat.EKS.Capability.1

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's capability, can configure the capability without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Capability.3](#mitigationekscapability3)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS cluster's capability can omit the required tags, or remove them through `UntagResource`. An Amazon EKS cluster's capability left without those tags still accrues hourly charges and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateCapability` | `$.capabilityName`, `$.clusterName`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.EKS.Capability.2

Threat actor, with knowledge of a flawed AWS IAM Identity Center's external provider configuration, can take advantage of the flawed AWS IAM Identity Center's external provider configuration to gain unauthorized access to the Amazon EKS cluster's capability.

**Impacts:** [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Capability.2](#mitigationekscapability2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/)

**Protocols:** `HTTPS`

**Scoping** (for [Flawed authentication model problem](../potential-threats.md#flawed-authentication-model-problem)): For an Argo CD capability's UI and API, the AWS IAM Identity Center instance you choose admits the identities your external identity provider asserts. A defect in that provider configuration's trust relationship, signature validation or identity mappings can let a threat actor gain unauthorized access to the Amazon EKS cluster's capability.

### Threat.EKS.Capability.3

Threat actor, with knowledge of a vulnerable external identity provider configured in an AWS IAM Identity Center instance used by an Amazon EKS cluster's capability, can gain federated access to the capability's hosted Argo CD UI and API.

**Impacts:** [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Capability.2](#mitigationekscapability2)

**Frameworks:** MITRE ATT&CK [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `SAML 2.0`, `HTTPS`

**Scoping** (for [Vulnerable identity provider problem](../potential-threats.md#vulnerable-identity-provider-problem)): An Amazon EKS cluster's capability delegates authentication for its Argo CD UI and API to the AWS IAM Identity Center instance named in its configuration. If the external identity provider you connect to that IAM Identity Center instance is vulnerable, a threat actor can gain federated access to the capability's hosted Argo CD UI and API.

### Threat.EKS.Capability.4

Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's capability's IAM role through its trust policy, can assume my capability IAM role and act with its permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access), [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.EKS.Capability.1](#mitigationekscapability1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1550.001](https://attack.mitre.org/techniques/T1550/001/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon EKS cluster's capability designates a capability IAM role through `$.roleArn`, and you author its trust policy. That trust policy can allow identities outside your organization to assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateCapability` | `$.roleArn` |
| `eks:UpdateCapability` | `$.roleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.EKS.Capability.5

IAM principal, with IAM permissions to use an Amazon EKS cluster's capability, can interact with the capability, causing its controllers to act on their behalf, without a record of that controller activity being persisted by vended logs delivery of its EKS_CAPABILITY_ACK_LOGS, EKS_CAPABILITY_KRO_LOGS, or EKS_CAPABILITY_ARGOCD_* controller logs.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Capability.4](#mitigationekscapability4)

**Protocols:** `HTTPS`

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When vended logs delivery is not configured on your Amazon EKS cluster's capability, no controller activity logs are written by vended logs delivery. An IAM principal with permissions to use the capability can create resources that its controllers reconcile on their behalf, such as AWS resources through ACK, composed resources through kro, or applications synchronized from a Git repository through Argo CD, leaving no record of those actions persisted by vended logs delivery.

### Threat.EKS.Capability.6

IAM principal, with IAM permissions to modify the log delivery configuration for an Amazon EKS capability, can disable controller log delivery for the capability.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Capability.4](#mitigationekscapability4)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to modify the CloudWatch Logs deliveries for your Amazon EKS cluster's capability can call `DeleteDelivery` on each delivery for the capability's ACK, kro or Argo CD controller logs, then `DeleteDeliverySource` on its delivery sources. Deleting those deliveries stops the capability's opt-in controller logs from reaching CloudWatch Logs while its controllers keep reconciling resources; calling `DeleteDelivery` for only some of an Argo CD capability's log types stops those records while its other log types continue.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:DeleteDelivery` | `$.id` |
| `cloudwatch-logs:DeleteDeliverySource` | `$.name` |

### Threat.EKS.Cluster.1

IAM principal, with IAM permissions to delete an Amazon EKS cluster, can delete the cluster and the Kubernetes objects that the cluster contains.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.EKS.Cluster.8](#mitigationekscluster8), [Mitigation.EKS.Cluster.14](#mitigationekscluster14)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Scoping** (for [Data loss through resource deletion](../potential-threats.md#data-loss-through-resource-deletion)): An Amazon EKS cluster holds your Kubernetes objects. An IAM principal with permission to delete the cluster can delete that data along with it through the `DeleteCluster` operation, without Kubernetes role-based access control (RBAC) permissions to delete those objects.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:DeleteCluster` | `$.name` |

### Threat.EKS.Cluster.2

IAM principal, with IAM permissions to delete an Amazon EKS cluster, can delete the cluster.

**Impacts:** [Service Stop](../threat-impacts.md#service-stop)

**Mitigations:** [Mitigation.EKS.Cluster.8](#mitigationekscluster8), [Mitigation.EKS.Cluster.14](#mitigationekscluster14)

**Frameworks:** MITRE ATT&CK [T1489](https://attack.mitre.org/techniques/T1489/)

**Scoping** (for [Service disruption through resource deletion](../potential-threats.md#service-disruption-through-resource-deletion)): An IAM principal with permissions to delete an Amazon EKS cluster can call `DeleteCluster` with the cluster's name in `$.name`. That call deletes the cluster and removes its Kubernetes control plane and cluster state, ending orchestration and Kubernetes API access for your workloads.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:DeleteCluster` | `$.name` |
| `eks:DeregisterCluster` | `$.name` |

### Threat.EKS.Cluster.3

IAM principal, with IAM permissions to modify an Amazon EKS cluster, can disable the EKS node monitoring agent on EKS Auto Mode nodes for the cluster.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Cluster.5](#mitigationekscluster5)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): The EKS node monitoring agent on EKS Auto Mode nodes reports kernel, container runtime, networking, storage, and accelerated hardware health beyond default CloudWatch metrics. An IAM principal with permission to modify the cluster can disable it, by turning off EKS Auto Mode compute with `$.computeConfig.enabled` set to false.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:UpdateClusterConfig` | `$.computeConfig.enabled`, `$.name` |

### Threat.EKS.Cluster.4

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster, can encrypt the Kubernetes API data, such as Secrets and ConfigMaps, with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.EKS.Cluster.9](#mitigationekscluster9)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal able to create your Amazon EKS cluster or associate an encryption configuration with it chooses the KMS key that encrypts the cluster's Kubernetes API data by setting `$.encryptionConfig[*].provider.keyArn`. If that key material sits outside your control, its holder can later revoke or delete it and leave that Kubernetes API data unreadable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:AssociateEncryptionConfig` | `$.clusterName`, `$.encryptionConfig[*].provider.keyArn` |
| `eks:CreateCluster` | `$.encryptionConfig[*].provider.keyArn`, `$.name` |

### Threat.EKS.Cluster.5

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster, can configure the cluster without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Cluster.12](#mitigationekscluster12)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS cluster can omit the required tags, or later remove them through `UntagResource`. An Amazon EKS cluster left without those tags still serves the Kubernetes API and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.tags{key}` |
| `eks:RegisterCluster` | `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.EKS.Cluster.6

Threat actor, with knowledge of a flawed access entry or aws-auth ConfigMap mapping, can take advantage of the flawed access entry or aws-auth ConfigMap mapping to gain unauthorized access to the Amazon EKS cluster.

**Impacts:** [Initial Access](../threat-impacts.md#initial-access), [Privilege Escalation](../threat-impacts.md#privilege-escalation)

**Mitigations:** [Mitigation.EKS.Cluster.10](#mitigationekscluster10)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1548.005](https://attack.mitre.org/techniques/T1548/005/)

**Scoping** (for [Flawed authentication model problem](../potential-threats.md#flawed-authentication-model-problem)): Your Amazon EKS cluster maps authenticated IAM principals to Kubernetes usernames and groups through access entries or the `aws-auth` ConfigMap. A defect in these mappings can let a threat actor gain unauthorized access to the Amazon EKS cluster.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |
| `sts:AssumeRole` | `$.RoleArn`, `$.RoleSessionName` |
| `sts:GetCallerIdentity` | *(any parameter)* |

### Threat.EKS.Cluster.7

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's Kubernetes API data, can encrypt the Kubernetes API data with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Cluster.9](#mitigationekscluster9)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal able to create your Amazon EKS cluster's Kubernetes API data chooses whether the AWS owned key protects it at rest. You cannot govern that key's policy or lifecycle, and the service's use of it produces no CloudTrail events in your account.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.encryptionConfig[*].provider.keyArn`, `$.name` |

### Threat.EKS.Cluster.8

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster that supports EKS Auto Mode compute and its EKS node monitoring agent, can create or configure the cluster without EKS Auto Mode compute and its EKS node monitoring agent enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Cluster.5](#mitigationekscluster5)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): An IAM principal can create or modify your Amazon EKS cluster without ever enabling EKS Auto Mode compute and its EKS node monitoring agent. Without the agent's node-level health conditions and metrics, anomalous node activity, such as runaway process creation, an outbound traffic spike, or an unexpected iptables reject rule, blends into standard node readiness and pressure conditions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.computeConfig.enabled`, `$.name` |
| `eks:UpdateClusterConfig` | `$.computeConfig.enabled`, `$.name` |

### Threat.EKS.Cluster.9

IAM principal, with unintended IAM permissions to read Kubernetes Secrets and other Kubernetes API data that an access entry or RBAC binding lets it read from an Amazon EKS cluster where the service uses grant-based KMS authorization, can read Kubernetes Secrets and other Kubernetes API data in the cluster because the KMS key policy cannot restrict which IAM principals can decrypt the data.

**Impacts:** [Credential Theft](../threat-impacts.md#credential-theft), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.Cluster.4](#mitigationekscluster4), [Mitigation.EKS.Cluster.11](#mitigationekscluster11), [Mitigation.EKS.Cluster.13](#mitigationekscluster13)

**Frameworks:** MITRE ATT&CK [T1552.007](https://attack.mitre.org/techniques/T1552/007/)

**Protocols:** `HTTPS`

**Scoping** (for [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)): Amazon EKS decrypts your cluster's Kubernetes Secrets and other Kubernetes API data under a KMS grant it holds, so the key policy cannot restrict which principals read them. An IAM principal with unintended Kubernetes read permissions from an access entry's EKS access policies or an RBAC binding can read the plaintext data over HTTPS through the cluster's public or private Kubernetes API server endpoint.

### Threat.EKS.Cluster.10

My IAM principal, that has been granted access to an untrusted Amazon EKS cluster through an access entry created by that cluster's owner, can interact with the untrusted cluster over its Kubernetes API server endpoint, including sending workloads and data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.Cluster.7](#mitigationekscluster7)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Protocols:** `HTTPS`

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): The owner of an Amazon EKS cluster outside your organization can grant one of your IAM principals access through an access entry. That principal can then move data into or out of the cluster over its Kubernetes API server endpoint.

### Threat.EKS.Cluster.11

Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon EKS cluster, can interact with the cluster's private Kubernetes API server endpoint through overly permissive network rules and move laterally within the VPC.

**Impacts:** [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.EKS.Cluster.13](#mitigationekscluster13)

**Frameworks:** MITRE ATT&CK [T1021](https://attack.mitre.org/techniques/T1021/)

**Protocols:** `HTTPS`

**Scoping** (for [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)): When you enable private endpoint access, Amazon EKS serves the Kubernetes API server endpoint for your cluster through cross-account elastic network interfaces in your VPC. A threat actor controlling another resource in that VPC can reach that endpoint through overly permissive network rules and move laterally.

### Threat.EKS.Cluster.12

Threat actor, with network access to an Amazon EKS cluster that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the cluster.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Cluster.13](#mitigationekscluster13)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1133](https://attack.mitre.org/techniques/T1133/), [T1613](https://attack.mitre.org/techniques/T1613/)

**Protocols:** `HTTPS`

**Scoping** (for [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)): Amazon EKS attaches the cluster security group it creates, and any additional security groups you specify, to the network interfaces in your VPC that serve your cluster's private Kubernetes API server endpoint. An inbound rule permitting an unrestricted source range lets any host that can route to those interfaces reach their listening services.

### Threat.EKS.Cluster.13

Untrusted IAM principal, that has been granted access to my Amazon EKS cluster through its access entries, can use my Amazon EKS cluster.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access), [Malicious Code Execution](../threat-impacts.md#malicious-code-execution)

**Mitigations:** [Mitigation.EKS.Cluster.11](#mitigationekscluster11)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1609](https://attack.mitre.org/techniques/T1609/), [T1610](https://attack.mitre.org/techniques/T1610/)

**Protocols:** `HTTPS`

**Scoping** (for [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)): Access entries on your Amazon EKS cluster can grant Kubernetes API access to principals outside your organization. Such a principal can authenticate to your cluster's Kubernetes API with its own IAM credentials. Depending on its entry's Kubernetes permissions, it can take actions such as reading Secrets or creating pods that run its code on your nodes.

### Threat.EKS.Cluster.14

IAM principal, with Kubernetes access granted through an Amazon EKS access entry or the aws-auth ConfigMap that permits deleting Kubernetes objects, such as Secrets, ConfigMaps, workloads and PersistentVolumeClaims, from an Amazon EKS cluster, can delete Kubernetes objects, such as Secrets, ConfigMaps, workloads and PersistentVolumeClaims, or whole namespaces from the cluster.

**Impacts:** [Data Destruction](../threat-impacts.md#data-destruction)

**Mitigations:** [Mitigation.EKS.Cluster.8](#mitigationekscluster8)

**Frameworks:** MITRE ATT&CK [T1485](https://attack.mitre.org/techniques/T1485/)

**Protocols:** `HTTPS`

**Scoping** (for [Direct data deletion](../potential-threats.md#direct-data-deletion)): Your Amazon EKS cluster's control plane stores Kubernetes objects, such as Secrets, ConfigMaps, workloads and PersistentVolumeClaims, each deletable through the Kubernetes API over HTTPS. An IAM principal with Kubernetes access granted through an Amazon EKS access entry or the `aws-auth` ConfigMap that permits deleting those objects can remove them individually or in whole namespaces while the cluster remains intact and stays active.

### Threat.EKS.Cluster.15

Threat actor, with network access to an Amazon EKS cluster that has been placed in a public subnet, can access the cluster directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Cluster.4](#mitigationekscluster4)

**Frameworks:** MITRE ATT&CK [T1133](https://attack.mitre.org/techniques/T1133/), [T1613](https://attack.mitre.org/techniques/T1613/)

**Protocols:** `HTTPS`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): An Amazon EKS cluster receives network interfaces for its private Kubernetes API server endpoint in the subnets you choose for it, with globally unique IPv6 addresses by default in an IPv6 cluster. An Amazon EKS cluster placed in a subnet that routes to an internet gateway can be reached directly from an ungoverned network when those interfaces have a public IP address.

### Threat.EKS.Cluster.16

IAM principal, with IAM permissions to create or update an Amazon EKS cluster's control plane instance and etcd instance, can configure the control plane instance and etcd instance to use an instance type that does not provide automatic encryption of instance-to-instance traffic.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.EKS.Cluster.6](#mitigationekscluster6)

**Scoping** (for [Unencrypted instance-to-instance traffic through instance type selection](../potential-threats.md#unencrypted-instance-to-instance-traffic-through-instance-type-selection)): An Amazon EKS cluster's AWS Outposts control plane and etcd instance type settings, `$.outpostConfig.controlPlaneInstanceType` and `$.outpostConfig.etcdInstanceType`, accept Amazon EC2 instance types that do not provide automatic encryption of instance-to-instance traffic. If such a type is selected, traffic between the cluster's instances themselves and other instances you operate can travel without that encryption.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.name`, `$.outpostConfig.controlPlaneInstanceType`, `$.outpostConfig.etcdInstanceType` |

### Threat.EKS.Cluster.17

Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's service role, Auto Mode node IAM role, and connector agent IAM role through their trust policies, can assume my service role, Auto Mode node IAM role, and connector agent IAM role and act with their permissions.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Cluster.3](#mitigationekscluster3)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1580](https://attack.mitre.org/techniques/T1580/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon EKS cluster designates its service role, Auto Mode node IAM role, and connector agent IAM role through `$.roleArn`, `$.computeConfig.nodeRoleArn`, `$.connectorConfig.roleArn`, and you author their trust policies. Those trust policies can allow identities outside your organization to assume the roles and act with their permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.computeConfig.nodeRoleArn`, `$.roleArn` |
| `eks:RegisterCluster` | `$.connectorConfig.roleArn` |
| `eks:UpdateClusterConfig` | `$.computeConfig.nodeRoleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.EKS.Cluster.18

Threat actor, with possession of credentials for my Amazon EKS cluster's Auto Mode node IAM role or my registered external cluster's EKS Connector agent IAM role, can use my Auto Mode node IAM role or EKS Connector agent IAM role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.Cluster.2](#mitigationekscluster2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): The instance metadata service on each EC2 managed instance serves credentials for your Auto Mode node IAM role, and AWS Systems Manager delivers credentials for your EKS Connector agent IAM role to the eks-connector agent. A threat actor who obtains those credentials can use either role while on an unexpected network.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecr:BatchGetImage` | `$.imageIds[*].imageDigest`, `$.imageIds[*].imageTag`, `$.registryId`, `$.repositoryName` |
| `ecr:GetAuthorizationToken` | `$.registryIds[*]` |
| `ecr:GetDownloadUrlForLayer` | `$.layerDigest`, `$.registryId`, `$.repositoryName` |
| `eks-auth:AssumeRoleForPodIdentity` | `$.clusterName`, `$.token` |

### Threat.EKS.Cluster.19

IAM principal, with IAM permissions to use an Amazon EKS cluster, can interact with the Kubernetes API without a record of their activities being persisted by control plane logging.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Cluster.1](#mitigationekscluster1)

**Protocols:** `HTTPS`

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When control plane logging is not configured on your Amazon EKS cluster, no control plane logs are written by control plane logging to CloudWatch Logs. An IAM principal with permissions to use the cluster can make Kubernetes API requests that its Kubernetes permissions allow, leaving no record of those requests persisted by control plane logging.

### Threat.EKS.Cluster.20

Threat actor, in possession of a long-lived, static Kubernetes service account token issued for an Amazon EKS cluster, can take advantage of the token to access the cluster.

**Impacts:** [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Cluster.10](#mitigationekscluster10)

**Frameworks:** MITRE ATT&CK [T1078.003](https://attack.mitre.org/techniques/T1078/003/), [T1133](https://attack.mitre.org/techniques/T1133/)

**Protocols:** `HTTPS`

**Scoping** (for [Weak authentication problem](../potential-threats.md#weak-authentication-problem)): Your Amazon EKS cluster's Kubernetes API server endpoint accepts Kubernetes service account tokens presented as bearer tokens in HTTPS requests. If you issue service account tokens to clients outside the cluster, such as CI/CD systems or external tools, anyone who obtains one of these tokens can authenticate as that service account and use its Kubernetes RBAC permissions from any network location that can reach the endpoint, including the internet if the public endpoint is enabled.

### Threat.EKS.Cluster.21

IAM principal, with IAM permissions to modify an Amazon EKS cluster, can disable control plane logging, including the audit and authenticator log types, for the cluster.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Cluster.1](#mitigationekscluster1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to modify your Amazon EKS cluster can set `$.logging.clusterLogging[*].enabled` to `false` for selected log types by calling `UpdateClusterConfig`. That change stops the opt-in control plane logs of those types from reaching CloudWatch Logs while later Kubernetes API activity can continue unrecorded; limiting `$.logging.clusterLogging[*].types[*]` to `audit` and `authenticator` stops only those logs and leaves the other log types enabled.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:UpdateClusterConfig` | `$.logging.clusterLogging[*].enabled`, `$.logging.clusterLogging[*].types[*]`, `$.name` |

### Threat.EKS.Cluster.22

Threat actor, with control of an Amazon EKS cluster configured with overly permissive outbound security group rules, can use the cluster to interact with ungoverned network destinations.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Cluster.13](#mitigationekscluster13)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): Outbound rules on the security groups that Amazon EKS associates with the cluster's cross-account network interfaces in your subnets govern connections from those interfaces, which reach external endpoints only when the cluster's control plane egress mode is `CUSTOMER_ROUTED`. Where those rules permit an unrestricted destination range, a threat actor controlling the cluster can use it to interact with ungoverned network destinations.

### Threat.EKS.Cluster.23

Threat actor, who can control the outbound connections of an Amazon EKS cluster using customer-routed control plane egress that is attached to a subnet with a route to an internet gateway, can send the cluster's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Cluster.4](#mitigationekscluster4)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): With customer-routed control plane egress, your IPv6 cluster connects to external endpoints through network interfaces that Amazon EKS places in the cluster subnets you specify. Where those subnets route to an internet gateway, the control plane's connections can reach the public internet directly when those interfaces have a public IP address.

### Threat.EKS.Cluster.24

Threat actor, with access to an Amazon EKS cluster, can submit a volume of requests that exceeds the capacity of the cluster or the network it is connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.EKS.Cluster.4](#mitigationekscluster4)

**Frameworks:** MITRE ATT&CK [T1499.002](https://attack.mitre.org/techniques/T1499/002/), [T1499.003](https://attack.mitre.org/techniques/T1499/003/)

**Protocols:** `HTTPS`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): With private endpoint access enabled, your Amazon EKS cluster receives Kubernetes API requests over HTTPS through its network interfaces in your subnets and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

### Threat.EKS.EksAnywhereSubscription.1

IAM principal, with IAM permissions to create or modify an Amazon EKS eks anywhere subscription, can configure the eks anywhere subscription without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS eks anywhere subscription can omit the required tags, or later remove them through `UntagResource`. An Amazon EKS eks anywhere subscription left without those tags still incurs charges for its term and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateEksAnywhereSubscription` | `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.EKS.FargateProfile.1

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's fargate profile, can configure the fargate profile without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS cluster's fargate profile can omit the required tags, or remove them later through `UntagResource`. A fargate profile left without those tags still selects pods for AWS Fargate and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateFargateProfile` | `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.EKS.FargateProfile.2

IAM principal, with IAM permissions to create an Amazon EKS cluster's fargate profile, can create the fargate profile in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create an Amazon EKS cluster's fargate profile can set the subnet list in `$.subnets[*]` to place the profile's Pods in a single Availability Zone. An impairment of that zone can then leave those Pods without a healthy zone to run in and interrupt the services that depend on them.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateFargateProfile` | `$.clusterName`, `$.fargateProfileName`, `$.selectors[*].labels{key}`, `$.selectors[*].namespace`, `$.subnets[*]` |

### Threat.EKS.FargateProfile.3

Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's Fargate profile's IAM Pod execution role through its trust policy, can assume my Pod execution role and act with its permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon EKS cluster's Fargate profile designates a Pod execution role through `$.podExecutionRoleArn`, and you author its trust policy. That trust policy can allow identities outside your organization to assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateFargateProfile` | `$.podExecutionRoleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.EKS.FargateProfile.4

Threat actor, with possession of credentials for my Amazon EKS cluster's fargate profile's Pod execution role, can use my Pod execution role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.FargateProfile.1](#mitigationeksfargateprofile1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): Amazon EKS provides temporary credentials for your fargate profile's Pod execution role to its own components that run your Pods on AWS Fargate. A threat actor who obtains those credentials can use that Pod execution role while on an unexpected network.

**Data perimeter:** `CreateFargateProfile` allows you to specify a service role, referred to as the Pod execution role, for your Pod operations. The service uses the role to make AWS API calls from your code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecr:BatchGetImage` | `$.imageIds[*].imageDigest`, `$.imageIds[*].imageTag`, `$.registryId`, `$.repositoryName` |
| `ecr:GetAuthorizationToken` | `$.registryIds[*]` |
| `ecr:GetDownloadUrlForLayer` | `$.layerDigest`, `$.registryId`, `$.repositoryName` |

### Threat.EKS.FargateProfile.5

Threat actor, who can execute code in Fargate Pods scheduled through an Amazon EKS cluster's fargate profile that is attached to a subnet with a route to an internet gateway, can send the Pods' outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): After you call `CreateFargateProfile`, Amazon EKS launches the Pods your Fargate profile selects, with the network interfaces that carry their outbound traffic, in the subnets you specify. Where those subnets route to an internet gateway, code running in those Pods can reach the public internet directly when those interfaces have a public IP address.

### Threat.EKS.FargateProfile.6

Threat actor, with network access to the Fargate Pods that an Amazon EKS cluster's fargate profile places in its subnets, can submit a volume of requests that exceeds the capacity of the Fargate Pods that the fargate profile places in its subnets or the network they are connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3)

**Frameworks:** MITRE ATT&CK [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1499.002](https://attack.mitre.org/techniques/T1499/002/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`, `DNS`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): Each Pod that your fargate profile schedules receives HTTP, HTTPS, TCP, UDP, or DNS requests on its own network interface in your subnets and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

### Threat.EKS.IdentityProviderConfig.1

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's identity provider config, can configure the identity provider config without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS cluster's identity provider config can omit the required tags, or remove them later through `UntagResource`. An Amazon EKS cluster's identity provider config left without those tags still authenticates users to its cluster and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:AssociateIdentityProviderConfig` | `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.EKS.IdentityProviderConfig.2

Threat actor, with knowledge of a flawed OIDC identity provider configuration, can take advantage of the flawed OIDC identity provider configuration to gain unauthorized access to the Amazon EKS cluster.

**Impacts:** [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/)

**Protocols:** `HTTPS`

**Scoping** (for [Flawed authentication model problem](../potential-threats.md#flawed-authentication-model-problem)): Your Amazon EKS cluster's OIDC identity provider configuration maps the identities in ID tokens from the issuer you specify to Kubernetes users and groups. A defect in that configuration can let a threat actor gain unauthorized access to your cluster's Kubernetes API.

### Threat.EKS.IdentityProviderConfig.3

Threat actor, with knowledge of a vulnerable Amazon EKS cluster's identity provider config, can gain federated access to the Kubernetes API server.

**Impacts:** [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/)

**Protocols:** `HTTPS`

**Scoping** (for [Vulnerable identity provider problem](../potential-threats.md#vulnerable-identity-provider-problem)): An Amazon EKS cluster's identity provider config delegates authentication for your cluster to the external OpenID Connect (OIDC) identity provider it names. If that identity provider is vulnerable, a threat actor presenting a signed ID token from that issuer can gain federated access to your Kubernetes API server.

### Threat.EKS.Nodegroup.1

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's nodegroup, can configure the nodegroup without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS cluster's nodegroup can omit the required tags, or remove them through `UntagResource`. A nodegroup left without those tags still runs your cluster's workloads and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.EKS.Nodegroup.2

IAM principal, with IAM permissions to create an Amazon EKS cluster's nodegroup, can create the nodegroup in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create an Amazon EKS cluster's nodegroup can use the subnets in `$.subnets[*]` to place every node and its network interfaces in a single Availability Zone. An impairment of that zone can then leave the nodegroup without a reachable node for the pods that can be scheduled only on it.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.subnets[*]` |

### Threat.EKS.Nodegroup.3

Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon EKS cluster's nodegroup, can interact with the nodegroup's worker node network interfaces through overly permissive network rules and move laterally within the VPC.

**Impacts:** [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)

**Frameworks:** MITRE ATT&CK [T1021](https://attack.mitre.org/techniques/T1021/)

**Protocols:** `SSH`, `RDP`, `HTTPS`, `HTTP`

**Scoping** (for [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)): Your nodegroup's Amazon EC2 worker nodes have network interfaces in your VPC subnets that serve your workloads and any remote access you configure. A threat actor controlling another resource in that VPC can reach those interfaces through overly permissive network rules and move laterally.

### Threat.EKS.Nodegroup.4

Threat actor, with network access to an Amazon EKS cluster's nodegroup that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the nodegroup.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1133](https://attack.mitre.org/techniques/T1133/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `SSH`, `RDP`, `HTTPS`, `HTTP`

**Scoping** (for [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)): The cluster security group or your launch template's security groups, plus any remote access security group, govern inbound traffic to your nodegroup's EC2 instances, which can have public IP addresses. An inbound rule permitting an unrestricted source range lets any host that can route to those instances reach their listening services and administrative ports.

### Threat.EKS.Nodegroup.5

Threat actor, with network access to an Amazon EKS cluster's nodegroup that has been placed in a public subnet, can access the nodegroup directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3)

**Frameworks:** MITRE ATT&CK [T1133](https://attack.mitre.org/techniques/T1133/), [T1190](https://attack.mitre.org/techniques/T1190/), [T1613](https://attack.mitre.org/techniques/T1613/)

**Protocols:** `SSH`, `RDP`, `HTTPS`, `HTTP`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): An Amazon EKS cluster's nodegroup launches its Amazon EC2 worker nodes and their network interfaces into the subnets you specify when you create it. An Amazon EKS cluster's nodegroup placed in a subnet that routes to an internet gateway can be reached directly from an ungoverned network when those nodes have a public IP address.

### Threat.EKS.Nodegroup.6

IAM principal, with IAM permissions to create or update an Amazon EKS cluster's nodegroup, can configure the nodegroup to use an instance type that does not provide automatic encryption of instance-to-instance traffic.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4)

**Scoping** (for [Unencrypted instance-to-instance traffic through instance type selection](../potential-threats.md#unencrypted-instance-to-instance-traffic-through-instance-type-selection)): An Amazon EKS cluster's nodegroup's instance type settings, `$.instanceTypes[*]` and `$.LaunchTemplateData.InstanceType`, accept Amazon EC2 instance types that do not provide automatic encryption of instance-to-instance traffic. If such a type is selected, traffic between the nodegroup's instances themselves and other instances you operate can travel without that encryption.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ec2:CreateLaunchTemplate` | `$.LaunchTemplateData.InstanceType` |
| `ec2:CreateLaunchTemplateVersion` | `$.LaunchTemplateData.InstanceType` |
| `eks:CreateNodegroup` | `$.clusterName`, `$.instanceTypes[*]`, `$.launchTemplate.id`, `$.launchTemplate.name`, `$.launchTemplate.version`, `$.nodegroupName` |
| `eks:UpdateNodegroupVersion` | `$.clusterName`, `$.launchTemplate.version`, `$.nodegroupName` |

### Threat.EKS.Nodegroup.7

Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's nodegroup's node IAM role through its trust policy, can assume my node IAM role and act with its permissions.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1580](https://attack.mitre.org/techniques/T1580/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon EKS cluster's nodegroup designates a node IAM role through `$.nodeRole`, and you author its trust policy. That trust policy can allow identities outside your organization to assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.nodeRole` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.EKS.Nodegroup.8

Threat actor, with possession of credentials for my Amazon EKS cluster's nodegroup's node IAM role, can use my node IAM role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.Nodegroup.1](#mitigationeksnodegroup1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): The node IAM role that you specify on `CreateNodegroup` provides temporary credentials to every instance in your nodegroup through the instance metadata service. A threat actor who obtains those credentials can use that node IAM role while on an unexpected network.

**Data perimeter:** `CreateNodegroup` allows you to specify a service role, referred to as the node IAM role, for your node operations. The service uses the role to make AWS API calls from your code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecr:BatchGetImage` | `$.imageIds[*].imageDigest`, `$.imageIds[*].imageTag`, `$.repositoryName` |
| `ecr:GetDownloadUrlForLayer` | `$.layerDigest`, `$.repositoryName` |

### Threat.EKS.Nodegroup.9

Threat actor, with control of an Amazon EKS cluster's nodegroup configured with overly permissive outbound security group rules, can use the nodegroup to interact with ungoverned network destinations.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): Nodes in the nodegroup and their Pods connect through network interfaces associated with the cluster security group or security groups in the launch template, whose outbound rules govern those connections. Where those rules permit an unrestricted destination range, a threat actor controlling the nodegroup can use it to interact with ungoverned network destinations.

### Threat.EKS.Nodegroup.10

Threat actor, who can execute code in an Amazon EKS cluster's nodegroup that is attached to a subnet with a route to an internet gateway, can send the nodegroup's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): When you call `CreateNodegroup`, Amazon EKS launches your nodegroup's worker nodes, with the network interfaces that carry their outbound traffic, in the subnets you specify. Where those subnets route to an internet gateway, code running on those nodes or in their Pods can reach the public internet directly when those interfaces have a public IP address.

### Threat.EKS.Nodegroup.11

Threat actor, with access to an Amazon EKS cluster's nodegroup, can submit a volume of requests that exceeds the capacity of the nodegroup or the network it is connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service), [Network Denial of Service](../threat-impacts.md#network-denial-of-service)

**Mitigations:** [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3)

**Frameworks:** MITRE ATT&CK [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1499.002](https://attack.mitre.org/techniques/T1499/002/)

**Protocols:** `HTTP`, `HTTPS`, `DNS`, `TCP`, `UDP`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): Each node that your nodegroup launches into your subnets receives HTTP, HTTPS, DNS, TCP, and UDP traffic on its network interfaces and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

### Threat.EKS.PodIdentityAssociation.1

IAM principal, with IAM permissions to create or modify an Amazon EKS cluster's pod identity association, can configure the pod identity association without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon EKS cluster's pod identity association can omit the required tags, or remove them later through `UntagResource`. A pod identity association left without those tags still provides IAM role credentials and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreatePodIdentityAssociation` | `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.EKS.PodIdentityAssociation.2

Untrusted IAM principal, that has been granted access to my Amazon EKS cluster's pod identity association's IAM EKS Pod Identity role through its trust policy, can assume my EKS Pod Identity role and act with its permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon EKS cluster's pod identity association designates an EKS Pod Identity role through `$.roleArn`, and you author its trust policy. That trust policy can allow identities outside your organization to assume the role and act with its permissions, accessing the AWS resources your workloads use.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eks:CreatePodIdentityAssociation` | `$.roleArn` |
| `eks:UpdatePodIdentityAssociation` | `$.roleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.EKS.PodIdentityAssociation.3

Threat actor, with possession of credentials for my Amazon EKS cluster's pod identity association's EKS Pod Identity role or, when configured, its target IAM role, can use my EKS Pod Identity role or, when configured, its target IAM role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): EKS Pod Identity delivers your Pods temporary credentials for the EKS Pod Identity role, or for the target IAM role when one is set. A threat actor who obtains those credentials can use either role while on an unexpected network.

**Data perimeter:** `CreatePodIdentityAssociation` allows you to specify a service role, referred to as the Pod Identity role, for your Pod operations. The service uses the role to make AWS API calls from your code.

**Data perimeter:** `UpdatePodIdentityAssociation` allows you to specify a service role, referred to as the EKS Pod Identity role, for your Pod operations. The service uses the role to make AWS API calls from your code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `s3:GetObject` | `$.Bucket`, `$.Key` |

---

## Mitigations

Each scoping statement below names the capability the mitigation goal governs on the entity and the objective of governing it. Each mitigation's implementations describe how to govern or monitor that capability.

### Mitigation.EKS.1

Amazon EKS service API calls on a governed VPC network should only be made against trusted resources or AWS service-owned resources

**Goal:** [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)

**Addresses:** [Threat.EKS.2](#threateks2)

**Implemented by:** [Implementation.EKS.1](#implementationeks1)

**Scoping** (for [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)): Amazon EKS API calls from your VPCs can target clusters in another account. Restricting those calls to resources in your organization enforces the resource perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.EKS.2

Amazon EKS service API calls on a governed VPC network should only be made by trusted identities or AWS services

**Goal:** [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)

**Addresses:** [Threat.EKS.1](#threateks1)

**Implemented by:** [Implementation.EKS.2](#implementationeks2)

**Scoping** (for [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)): Amazon EKS API calls from your VPCs can be made by identities outside your organization. Restricting those calls to principals in your organization enforces the identity perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.EKS.AccessEntry.1

An Amazon EKS cluster's access entry should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.AccessEntry.1](#threateksaccessentry1)

**Implemented by:** [Implementation.EKS.AccessEntry.1](#implementationeksaccessentry1), [Implementation.EKS.AccessEntry.2](#implementationeksaccessentry2), [Implementation.EKS.AccessEntry.3](#implementationeksaccessentry3)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag an access entry through the `$.tags{key}` key-to-value map on `CreateAccessEntry` or later through `TagResource`, separately from your cluster's tags. Owner and purpose tags help track ownership and the business function each cluster's access entry supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateAccessEntry` | `$.clusterName`, `$.principalArn`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeAccessEntry` | `$.clusterName`, `$.principalArn` |

### Mitigation.EKS.Addon.1

An Amazon EKS cluster's addon should be configured to send network policy logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.EKS.Addon.6](#threateksaddon6), [Threat.EKS.Addon.7](#threateksaddon7)

**Implemented by:** [Implementation.EKS.Addon.1](#implementationeksaddon1), [Implementation.EKS.Addon.2](#implementationeksaddon2), [Implementation.EKS.Addon.3](#implementationeksaddon3)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): The Amazon VPC CNI add-on's network policy agent can record network policy logs of allowed and denied connections, with delivery controlled by the `$.configurationValues` request parameter. Sending those logs to CloudWatch Logs provides security-relevant visibility for audit trails, access monitoring, or error detection.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.addonName`, `$.clusterName`, `$.configurationValues` |
| `eks:UpdateAddon` | `$.addonName`, `$.clusterName`, `$.configurationValues` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeAddon` | `$.addonName`, `$.clusterName` |

### Mitigation.EKS.Addon.2

An Amazon EKS cluster's addon's EKS Pod Identity IAM role, or its IAM role for service accounts where no pod identity association applies, should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.EKS.Addon.5](#threateksaddon5)

**Implemented by:** [Implementation.EKS.Addon.4](#implementationeksaddon4)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): Your add-on binds IAM roles to its Kubernetes service accounts through `$.podIdentityAssociations[*].roleArn` (each EKS Pod Identity IAM role) and `$.serviceAccountRoleArn` (the IAM role for service accounts), and its containers call AWS services with the credentials of whichever of those roles applies. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.addonName`, `$.clusterName`, `$.podIdentityAssociations[*].roleArn`, `$.serviceAccountRoleArn` |
| `eks:UpdateAddon` | `$.addonName`, `$.clusterName`, `$.podIdentityAssociations[*].roleArn`, `$.serviceAccountRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeAddon` | `$.addonName`, `$.clusterName` |

### Mitigation.EKS.Addon.3

An Amazon EKS cluster's addon's Pod Identity IAM role should be assumed only by Amazon EKS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.EKS.Addon.4](#threateksaddon4)

**Implemented by:** [Implementation.EKS.Addon.5](#implementationeksaddon5), [Implementation.EKS.Addon.6](#implementationeksaddon6), [Implementation.EKS.Addon.7](#implementationeksaddon7)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): You can pass a Pod Identity IAM role for your cluster's addon's service account in the `$.podIdentityAssociations[*].roleArn` request parameter, and the role's trust policy specifies which principals can assume it. Allowing only Amazon EKS, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.podIdentityAssociations[*].roleArn` |
| `eks:UpdateAddon` | `$.podIdentityAssociations[*].roleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeAddon` | `$.addonName`, `$.clusterName` |

### Mitigation.EKS.Addon.4

An Amazon EKS cluster's addon should have Prometheus Node Exporter, kube-state-metrics, EKS node monitoring agent, AWS Distro for OpenTelemetry (ADOT), Container Network Observability performance metrics, SR-IOV Network Metrics Exporter, or Amazon SageMaker HyperPod Observability Add-on enabled to collect detailed non-CloudWatch metrics

**Goal:** [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics)

**Addresses:** [Threat.EKS.Addon.1](#threateksaddon1), [Threat.EKS.Addon.3](#threateksaddon3)

**Implemented by:** [Implementation.EKS.Addon.1](#implementationeksaddon1), [Implementation.EKS.Addon.8](#implementationeksaddon8), [Implementation.EKS.Addon.9](#implementationeksaddon9)

**Scoping** (for [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics)): Your Amazon EKS cluster's addon can be Prometheus Node Exporter, kube-state-metrics, EKS node monitoring agent, AWS Distro for OpenTelemetry (ADOT), SR-IOV Network Metrics Exporter, Amazon SageMaker HyperPod Observability Add-on, or the AWS Network Flow Monitor Agent with Container Network Observability performance metrics enabled through its configuration values. These features capture detailed operational telemetry, such as hardware and network-device metrics, Kubernetes object state, and node health signals, kept outside Amazon CloudWatch Metrics.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.addonName`, `$.clusterName`, `$.configurationValues` |
| `eks:UpdateAddon` | `$.addonName`, `$.clusterName`, `$.configurationValues` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeAddon` | `$.addonName`, `$.clusterName` |
| `eks:ListAddons` | `$.clusterName` |

### Mitigation.EKS.Addon.5

An Amazon EKS cluster's addon should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.Addon.2](#threateksaddon2)

**Implemented by:** [Implementation.EKS.Addon.10](#implementationeksaddon10), [Implementation.EKS.Addon.11](#implementationeksaddon11), [Implementation.EKS.Addon.12](#implementationeksaddon12), [ManagedControl.EKS.Addon.1](#managedcontroleksaddon1)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag a cluster's addon with the `$.tags{key}` key-to-value map on `CreateAddon` or through `TagResource`, and these tags are separate from the cluster's tags. Owner and purpose tags help track ownership and the business function each cluster's addon supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.addonName`, `$.clusterName`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeAddon` | `$.addonName`, `$.clusterName` |
| `eks:ListTagsForResource` | `$.resourceArn` |

### Mitigation.EKS.Addon.6

An Amazon EKS cluster's addon should have Container Insights with enhanced observability enabled to collect detailed CloudWatch metrics

**Goal:** [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)

**Addresses:** [Threat.EKS.Addon.1](#threateksaddon1), [Threat.EKS.Addon.3](#threateksaddon3)

**Implemented by:** [Implementation.EKS.Addon.1](#implementationeksaddon1), [Implementation.EKS.Addon.13](#implementationeksaddon13), [Implementation.EKS.Addon.14](#implementationeksaddon14)

**Scoping** (for [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)): Setting `$.addonName`, the add-on name, to `amazon-cloudwatch-observability` installs Container Insights with enhanced observability, which publishes cluster, node, pod, container, and control plane metrics beyond the basic `AWS/EKS` metrics. Collecting those metrics improves security visibility by providing finer-grained data for anomaly detection and incident response.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateAddon` | `$.addonName`, `$.clusterName`, `$.configurationValues` |
| `eks:UpdateAddon` | `$.addonName`, `$.clusterName`, `$.configurationValues` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeAddon` | `$.addonName`, `$.clusterName` |
| `eks:ListAddons` | `$.clusterName` |

### Mitigation.EKS.Capability.1

An Amazon EKS cluster's capability's IAM role should be assumed only by Amazon EKS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.EKS.Capability.4](#threatekscapability4)

**Implemented by:** [Implementation.EKS.Capability.1](#implementationekscapability1), [Implementation.EKS.Capability.2](#implementationekscapability2), [Implementation.EKS.Capability.3](#implementationekscapability3), [Implementation.EKS.Capability.4](#implementationekscapability4)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): The `$.roleArn` request parameter names the capability IAM role that Amazon EKS assumes, and the role's trust policy specifies which principals can assume it. Allowing only Amazon EKS, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCapability` | `$.roleArn` |
| `eks:UpdateCapability` | `$.roleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCapability` | `$.capabilityName`, `$.clusterName` |

### Mitigation.EKS.Capability.2

An Amazon EKS cluster's capability that uses AWS IAM Identity Center to authenticate principals should be configured only with trusted IAM Identity Center instances

**Goal:** [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration)

**Addresses:** [Threat.EKS.Capability.2](#threatekscapability2), [Threat.EKS.Capability.3](#threatekscapability3)

**Implemented by:** [Implementation.EKS.Capability.5](#implementationekscapability5), [Implementation.EKS.Capability.6](#implementationekscapability6)

**Scoping** (for [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration)): An Amazon EKS cluster's capability of type Argo CD authenticates users to its hosted Argo CD UI and API through the AWS IAM Identity Center instance set by the `$.configuration.argoCd.awsIdc.idcInstanceArn` request parameter. Configuring only trusted IAM Identity Center instances helps prevent an untrusted identity provider from being used to gain unintended access.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCapability` | `$.capabilityName`, `$.clusterName`, `$.configuration.argoCd.awsIdc.idcInstanceArn`, `$.configuration.argoCd.awsIdc.idcRegion` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCapability` | `$.capabilityName`, `$.clusterName` |
| `eks:ListCapabilities` | `$.clusterName` |

### Mitigation.EKS.Capability.3

An Amazon EKS cluster's capability should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.Capability.1](#threatekscapability1)

**Implemented by:** [Implementation.EKS.Capability.7](#implementationekscapability7), [Implementation.EKS.Capability.8](#implementationekscapability8), [Implementation.EKS.Capability.9](#implementationekscapability9)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag your cluster's capability through the `$.tags{key}` key-to-value map on `CreateCapability` or by calling `TagResource` on the capability ARN, separately from the cluster. Owner and purpose tags help track ownership and the business function each cluster's capability supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCapability` | `$.capabilityName`, `$.clusterName`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCapability` | `$.capabilityName`, `$.clusterName` |
| `eks:ListTagsForResource` | `$.resourceArn` |

### Mitigation.EKS.Capability.4

An Amazon EKS cluster's capability should be configured to send EKS_CAPABILITY_ACK_LOGS, EKS_CAPABILITY_KRO_LOGS, or EKS_CAPABILITY_ARGOCD_* logs to CloudWatch Logs using vended logs delivery

**Goal:** [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2)

**Addresses:** [Threat.EKS.Capability.5](#threatekscapability5), [Threat.EKS.Capability.6](#threatekscapability6)

**Implemented by:** [Implementation.EKS.Capability.10](#implementationekscapability10), [Implementation.EKS.Capability.11](#implementationekscapability11)

**Scoping** (for [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2)): Controller logs record the activity of the ACK, kro, or Argo CD controllers that Amazon EKS runs for your capability outside your cluster. Sending those logs to CloudWatch Logs using vended logs delivery provides security-relevant visibility.

**Configure via**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:CreateDelivery` | `$.deliveryDestinationArn`, `$.deliverySourceName` |
| `cloudwatch-logs:PutDeliveryDestination` | `$.deliveryDestinationConfiguration.destinationResourceArn`, `$.deliveryDestinationType` |
| `cloudwatch-logs:PutDeliverySource` | `$.logType`, `$.resourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:GetDelivery` | `$.id` |
| `cloudwatch-logs:GetDeliveryDestination` | `$.name` |
| `cloudwatch-logs:GetDeliverySource` | `$.name` |

### Mitigation.EKS.Cluster.1

An Amazon EKS cluster should be configured to send API server, audit, authenticator, controller manager, and scheduler logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.EKS.Cluster.19](#threatekscluster19), [Threat.EKS.Cluster.21](#threatekscluster21)

**Implemented by:** [Implementation.EKS.Cluster.1](#implementationekscluster1), [Implementation.EKS.Cluster.2](#implementationekscluster2), [Implementation.EKS.Cluster.3](#implementationekscluster3), [ManagedControl.EKS.Cluster.1](#managedcontrolekscluster1), [ManagedControl.EKS.Cluster.2](#managedcontrolekscluster2)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): Your Amazon EKS cluster's control plane produces API server, audit, authenticator, controller manager, and scheduler logs that record Kubernetes API activity, IAM-based authentication, control loop activity, and Pod scheduling, each exported only when enabled in its logging configuration. Sending those logs to CloudWatch Logs provides security-relevant visibility for audit trails, access monitoring, or error detection.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.logging.clusterLogging[*].enabled`, `$.logging.clusterLogging[*].types[*]`, `$.name` |
| `eks:UpdateClusterConfig` | `$.logging.clusterLogging[*].enabled`, `$.logging.clusterLogging[*].types[*]`, `$.name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |
| `eks:DescribeUpdate` | `$.name`, `$.updateId` |

### Mitigation.EKS.Cluster.2

An Amazon EKS cluster's Auto Mode node IAM role and EKS Connector agent IAM role should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.EKS.Cluster.18](#threatekscluster18)

**Implemented by:** [Implementation.EKS.Cluster.4](#implementationekscluster4)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): Your EKS Auto Mode cluster's `$.computeConfig.nodeRoleArn` request parameter designates the Auto Mode node IAM role, whose credentials the code on its EC2 managed instances uses to call AWS services, and a registered external cluster's `$.connectorConfig.roleArn` designates the EKS Connector agent IAM role, whose credentials the EKS Connector agent in that connected cluster uses to call AWS services. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.computeConfig.nodeRoleArn`, `$.name` |
| `eks:RegisterCluster` | `$.connectorConfig.roleArn`, `$.name` |
| `eks:UpdateClusterConfig` | `$.computeConfig.nodeRoleArn`, `$.name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |

### Mitigation.EKS.Cluster.3

An Amazon EKS cluster's service role should be assumed only by Amazon EKS, its Auto Mode node IAM role only by Amazon EC2, and its connector agent IAM role only by AWS Systems Manager, acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.EKS.Cluster.17](#threatekscluster17)

**Implemented by:** [Implementation.EKS.Cluster.5](#implementationekscluster5), [Implementation.EKS.Cluster.6](#implementationekscluster6), [Implementation.EKS.Cluster.7](#implementationekscluster7), [Implementation.EKS.Cluster.8](#implementationekscluster8), [Implementation.EKS.Cluster.9](#implementationekscluster9), [Implementation.EKS.Cluster.10](#implementationekscluster10)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): You pass your cluster's service role in the `$.roleArn` request parameter, you can pass an Auto Mode node IAM role in `$.computeConfig.nodeRoleArn` and a registered cluster's connector agent IAM role in `$.connectorConfig.roleArn`, and each role's trust policy specifies which principals can assume it. Allowing only Amazon EKS, Amazon EC2, and AWS Systems Manager, acting on behalf of trusted resources, to assume those roles helps prevent an unintended principal from assuming them.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.computeConfig.nodeRoleArn`, `$.roleArn` |
| `eks:RegisterCluster` | `$.connectorConfig.roleArn` |
| `eks:UpdateClusterConfig` | `$.computeConfig.nodeRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |

### Mitigation.EKS.Cluster.4

An Amazon EKS cluster's control plane network interfaces should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.EKS.Cluster.9](#threatekscluster9), [Threat.EKS.Cluster.15](#threatekscluster15), [Threat.EKS.Cluster.23](#threatekscluster23), [Threat.EKS.Cluster.24](#threatekscluster24)

**Implemented by:** [Implementation.EKS.Cluster.11](#implementationekscluster11), [Implementation.EKS.Cluster.12](#implementationekscluster12)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): The `$.resourcesVpcConfig.subnetIds[*]` request parameter sets your cluster subnets, which hold the cross-account elastic network interfaces connecting your cluster's control plane to your nodes. Private subnet placement helps prevent direct internet routing to your cluster's control plane network interfaces, reducing their network exposure.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.kubernetesNetworkConfig.ipFamily`, `$.name`, `$.resourcesVpcConfig.endpointPrivateAccess`, `$.resourcesVpcConfig.endpointPublicAccess`, `$.resourcesVpcConfig.publicAccessCidrs[*]`, `$.resourcesVpcConfig.subnetIds[*]` |
| `eks:UpdateClusterConfig` | `$.name`, `$.resourcesVpcConfig.endpointPrivateAccess`, `$.resourcesVpcConfig.endpointPublicAccess`, `$.resourcesVpcConfig.publicAccessCidrs[*]`, `$.resourcesVpcConfig.subnetIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |
| `eks:DescribeUpdate` | `$.name`, `$.updateId` |

### Mitigation.EKS.Cluster.5

An Amazon EKS cluster should have the EKS node monitoring agent enabled to collect detailed non-CloudWatch metrics

**Goal:** [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics)

**Addresses:** [Threat.EKS.Cluster.3](#threatekscluster3), [Threat.EKS.Cluster.8](#threatekscluster8)

**Implemented by:** [Implementation.EKS.Cluster.13](#implementationekscluster13), [Implementation.EKS.Cluster.14](#implementationekscluster14), [Implementation.EKS.Cluster.15](#implementationekscluster15)

**Scoping** (for [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics)): With EKS Auto Mode compute enabled, your Amazon EKS cluster includes the EKS node monitoring agent in the AMI of every Auto Mode node. This feature captures detailed operational telemetry, such as kernel, container runtime, networking, storage, and accelerated-hardware health issues, kept outside Amazon CloudWatch Metrics.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.computeConfig.enabled`, `$.name` |
| `eks:UpdateClusterConfig` | `$.computeConfig.enabled`, `$.name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |

### Mitigation.EKS.Cluster.6

An Amazon EKS cluster's control plane instance and etcd instance should use an EC2 instance type that supports automatic encryption of instance-to-instance traffic

**Goal:** [Use Nitro instances](../mitigation-goals.md#use-nitro-instances)

**Addresses:** [Threat.EKS.Cluster.16](#threatekscluster16)

**Implemented by:** [Implementation.EKS.Cluster.16](#implementationekscluster16), [Implementation.EKS.Cluster.17](#implementationekscluster17)

**Scoping** (for [Use Nitro instances](../mitigation-goals.md#use-nitro-instances)): A local Amazon EKS cluster runs its Kubernetes control plane and etcd on Amazon EC2 instances that Amazon EKS launches on your Outpost. The instance type you select for the control plane and etcd instances determines whether traffic between those instances can be automatically encrypted in transit, which applies between supported instance types in the same Region on a direct network path.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.name`, `$.outpostConfig.controlPlaneInstanceType`, `$.outpostConfig.etcdInstanceType` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |

### Mitigation.EKS.Cluster.7

IAM principals should be restricted to accessing only trusted Amazon EKS cluster resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.EKS.Cluster.10](#threatekscluster10)

**Implemented by:** [Implementation.EKS.Cluster.18](#implementationekscluster18)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): An access entry on an Amazon EKS cluster in another account can grant your principals access to that cluster's Kubernetes API. Restricting the clusters your principals can access helps keep data access within trusted accounts.

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

### Mitigation.EKS.Cluster.8

An Amazon EKS cluster should be protected by an AWS Backup backup plan

**Goal:** [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup)

**Addresses:** [Threat.EKS.Cluster.1](#threatekscluster1), [Threat.EKS.Cluster.2](#threatekscluster2), [Threat.EKS.Cluster.14](#threatekscluster14)

**Implemented by:** [Implementation.EKS.Cluster.19](#implementationekscluster19), [Implementation.EKS.Cluster.20](#implementationekscluster20), [Implementation.EKS.Cluster.21](#implementationekscluster21), [ManagedControl.EKS.Cluster.3](#managedcontrolekscluster3)

**Scoping** (for [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup)): An Amazon EKS cluster stores Kubernetes cluster state and persistent volume data. AWS Backup supports the Amazon EKS cluster as an independently protectable resource type and captures its contents in composite recovery points.

**Configure via**

| Operation | Parameters |
|---|---|
| `backup:CreateBackupPlan` | `$.BackupPlan.Rules[*].ScheduleExpression`, `$.BackupPlan.Rules[*].TargetBackupVaultName` |
| `backup:CreateBackupSelection` | `$.BackupPlanId`, `$.BackupSelection.Conditions.StringEquals[*].ConditionKey`, `$.BackupSelection.Conditions.StringEquals[*].ConditionValue`, `$.BackupSelection.ListOfTags[*].ConditionKey`, `$.BackupSelection.ListOfTags[*].ConditionValue`, `$.BackupSelection.NotResources[*]`, `$.BackupSelection.Resources[*]` |
| `backup:UpdateRegionSettings` | `$.ResourceTypeOptInPreference{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `backup:DescribeProtectedResource` | `$.ResourceArn` |
| `backup:DescribeRegionSettings` | *(any parameter)* |
| `backup:GetBackupPlan` | `$.BackupPlanId` |
| `backup:GetBackupSelection` | `$.BackupPlanId`, `$.SelectionId` |
| `backup:ListBackupJobs` | `$.ByResourceArn` |
| `backup:ListBackupSelections` | `$.BackupPlanId` |
| `backup:ListProtectedResources` | *(any parameter)* |
| `backup:ListRecoveryPointsByResource` | `$.ResourceArn` |

### Mitigation.EKS.Cluster.9

An Amazon EKS cluster's Kubernetes API data should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.EKS.Cluster.4](#threatekscluster4), [Threat.EKS.Cluster.7](#threatekscluster7)

**Implemented by:** [Implementation.EKS.Cluster.22](#implementationekscluster22), [Implementation.EKS.Cluster.23](#implementationekscluster23), [Implementation.EKS.Cluster.24](#implementationekscluster24), [ManagedControl.EKS.Cluster.4](#managedcontrolekscluster4), [ManagedControl.EKS.Cluster.5](#managedcontrolekscluster5)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): Envelope encryption helps protect the Kubernetes API data, such as Secrets and ConfigMaps, that your Amazon EKS cluster stores in etcd. Through `CreateCluster` or `AssociateEncryptionConfig`, you can use a customer managed key as the key encryption key instead of an AWS owned key. A customer managed key lets you audit its use and control its lifecycle, including deleting it to make the stored data unavailable.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:AssociateEncryptionConfig` | `$.clusterName`, `$.encryptionConfig[*].provider.keyArn` |
| `eks:CreateCluster` | `$.encryptionConfig[*].provider.keyArn`, `$.name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |
| `eks:DescribeUpdate` | `$.name`, `$.updateId` |

### Mitigation.EKS.Cluster.10

An Amazon EKS cluster should use cluster access management API

**Goal:** [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication)

**Addresses:** [Threat.EKS.Cluster.6](#threatekscluster6), [Threat.EKS.Cluster.20](#threatekscluster20)

**Implemented by:** [Implementation.EKS.Cluster.25](#implementationekscluster25), [Implementation.EKS.Cluster.26](#implementationekscluster26)

**Scoping** (for [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication)): Amazon EKS supports managing the Kubernetes permissions of IAM principals from outside the cluster through access entries, its cluster access management API. The `aws-auth` ConfigMap, whose accidental deletion or syntax errors can lock users out, is deprecated in favor of access entries.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.accessConfig.authenticationMode`, `$.name` |
| `eks:UpdateClusterConfig` | `$.accessConfig.authenticationMode`, `$.name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |
| `eks:ListAccessEntries` | `$.clusterName` |

### Mitigation.EKS.Cluster.11

An Amazon EKS cluster should be accessed only by trusted identities or AWS services acting on behalf of trusted resources

**Goal:** [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)

**Addresses:** [Threat.EKS.Cluster.9](#threatekscluster9), [Threat.EKS.Cluster.13](#threatekscluster13)

**Implemented by:** [Implementation.EKS.Cluster.27](#implementationekscluster27), [Implementation.EKS.Cluster.28](#implementationekscluster28), [Implementation.EKS.Cluster.29](#implementationekscluster29)

**Scoping** (for [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)): Your Amazon EKS cluster supports access entries that specify who can access its Kubernetes API and what actions they can perform. Restricting who can access your clusters helps keep data access within trusted accounts.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateAccessEntry` | `$.principalArn` |
| `eks:CreateCluster` | `$.accessConfig.bootstrapClusterCreatorAdminPermissions` |
| `eks:DeleteAccessEntry` | `$.principalArn` |
| `eks:UpdateAccessEntry` | `$.principalArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeAccessEntry` | `$.clusterName`, `$.principalArn` |
| `eks:DescribeCluster` | `$.name` |
| `eks:ListAccessEntries` | `$.clusterName` |

### Mitigation.EKS.Cluster.12

An Amazon EKS cluster should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.Cluster.5](#threatekscluster5)

**Implemented by:** [Implementation.EKS.Cluster.30](#implementationekscluster30), [Implementation.EKS.Cluster.31](#implementationekscluster31), [Implementation.EKS.Cluster.32](#implementationekscluster32)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag an Amazon EKS cluster through the `$.tags{key}` request parameter, a key-to-value map, when you call `CreateCluster` or later through `TagResource`, and a connected cluster only when you call `RegisterCluster`. Owner and purpose tags help track ownership and the business function each cluster supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.name`, `$.tags{key}` |
| `eks:RegisterCluster` | `$.name`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |
| `eks:ListTagsForResource` | `$.resourceArn` |

### Mitigation.EKS.Cluster.13

An Amazon EKS cluster should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.EKS.Cluster.9](#threatekscluster9), [Threat.EKS.Cluster.11](#threatekscluster11), [Threat.EKS.Cluster.12](#threatekscluster12), [Threat.EKS.Cluster.22](#threatekscluster22)

**Implemented by:** [Implementation.EKS.Cluster.33](#implementationekscluster33), [Implementation.EKS.Cluster.34](#implementationekscluster34), [Implementation.EKS.Cluster.35](#implementationekscluster35)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): Amazon EKS associates the cluster security group it creates with the cross-account elastic network interfaces in your cluster subnets, and you can associate additional groups through `$.resourcesVpcConfig.securityGroupIds[*]`. Restricting the inbound and outbound rules to the required sources, destinations, protocols, and ports helps reduce the network exposure of your cluster.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.name`, `$.resourcesVpcConfig.securityGroupIds[*]` |
| `eks:UpdateClusterConfig` | `$.name`, `$.resourcesVpcConfig.securityGroupIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |

### Mitigation.EKS.Cluster.14

An Amazon EKS cluster should have deletion protection enabled

**Goal:** [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection)

**Addresses:** [Threat.EKS.Cluster.1](#threatekscluster1), [Threat.EKS.Cluster.2](#threatekscluster2)

**Implemented by:** [Implementation.EKS.Cluster.36](#implementationekscluster36), [Implementation.EKS.Cluster.37](#implementationekscluster37), [Implementation.EKS.Cluster.38](#implementationekscluster38), [Implementation.EKS.Cluster.39](#implementationekscluster39)

**Scoping** (for [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection)): An Amazon EKS cluster has its own deletion protection setting, `$.deletionProtection`, which you can turn on at creation or later. This setting helps protect the cluster from unintended deletion.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateCluster` | `$.deletionProtection`, `$.name` |
| `eks:UpdateClusterConfig` | `$.deletionProtection`, `$.name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeCluster` | `$.name` |
| `eks:DescribeUpdate` | `$.name`, `$.updateId` |

### Mitigation.EKS.EksAnywhereSubscription.1

An Amazon EKS eks anywhere subscription should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.EksAnywhereSubscription.1](#threatekseksanywheresubscription1)

**Implemented by:** [Implementation.EKS.EksAnywhereSubscription.1](#implementationekseksanywheresubscription1), [Implementation.EKS.EksAnywhereSubscription.2](#implementationekseksanywheresubscription2)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag an eks anywhere subscription through the `$.tags{key}` key-to-value map on `CreateEksAnywhereSubscription`, or later through `TagResource`. Owner and purpose tags help track ownership and the business function each eks anywhere subscription supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateEksAnywhereSubscription` | `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeEksAnywhereSubscription` | `$.id` |
| `eks:ListEksAnywhereSubscriptions` | *(any parameter)* |
| `eks:ListTagsForResource` | `$.resourceArn` |

### Mitigation.EKS.FargateProfile.1

An Amazon EKS cluster's fargate profile's Pod execution role should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.EKS.FargateProfile.4](#threateksfargateprofile4)

**Implemented by:** [Implementation.EKS.FargateProfile.1](#implementationeksfargateprofile1)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): The `$.podExecutionRoleArn` request parameter designates your Fargate profile's Pod execution role, and the Amazon EKS components on the Fargate infrastructure call AWS services with the role's credentials. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateFargateProfile` | `$.clusterName`, `$.fargateProfileName`, `$.podExecutionRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeFargateProfile` | `$.clusterName`, `$.fargateProfileName` |

### Mitigation.EKS.FargateProfile.2

An Amazon EKS cluster's Fargate profile's Pod execution role should be assumed only by Amazon EKS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.EKS.FargateProfile.3](#threateksfargateprofile3)

**Implemented by:** [Implementation.EKS.FargateProfile.2](#implementationeksfargateprofile2), [Implementation.EKS.FargateProfile.3](#implementationeksfargateprofile3), [Implementation.EKS.FargateProfile.4](#implementationeksfargateprofile4), [Implementation.EKS.FargateProfile.5](#implementationeksfargateprofile5)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): Your cluster's Fargate profile binds a Pod execution role through the required `$.podExecutionRoleArn` request parameter, and the role's trust policy specifies which principals can assume it. Allowing only Amazon EKS, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateFargateProfile` | `$.podExecutionRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeFargateProfile` | `$.clusterName`, `$.fargateProfileName` |

### Mitigation.EKS.FargateProfile.3

An Amazon EKS cluster's fargate profile's Pods should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.EKS.FargateProfile.5](#threateksfargateprofile5), [Threat.EKS.FargateProfile.6](#threateksfargateprofile6)

**Implemented by:** [Implementation.EKS.FargateProfile.6](#implementationeksfargateprofile6), [Implementation.EKS.FargateProfile.7](#implementationeksfargateprofile7)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): The `$.subnets[*]` request parameter on `CreateFargateProfile` sets the subnets that hold the Pods your Fargate profile schedules, each with its own elastic network interface. Private subnet placement helps prevent direct internet routing to those Pods, reducing their network exposure.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateFargateProfile` | `$.clusterName`, `$.fargateProfileName`, `$.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeFargateProfile` | `$.clusterName`, `$.fargateProfileName` |

### Mitigation.EKS.FargateProfile.4

An Amazon EKS cluster's Fargate profile should launch its Pods in subnets in more than one Availability Zone.

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.EKS.FargateProfile.2](#threateksfargateprofile2)

**Implemented by:** [Implementation.EKS.FargateProfile.8](#implementationeksfargateprofile8), [Implementation.EKS.FargateProfile.9](#implementationeksfargateprofile9), [Implementation.EKS.FargateProfile.10](#implementationeksfargateprofile10)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): Amazon EKS launches each Pod that matches your Fargate profile with its own elastic network interface in a subnet from the `$.subnets[*]` subnet list. Placing those network interfaces in more than one Availability Zone provides network-level redundancy and helps keep connectivity available when one zone is disrupted.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateFargateProfile` | `$.clusterName`, `$.fargateProfileName`, `$.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeFargateProfile` | `$.clusterName`, `$.fargateProfileName` |
| `eks:ListFargateProfiles` | `$.clusterName` |

### Mitigation.EKS.FargateProfile.5

An Amazon EKS cluster's fargate profile should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.FargateProfile.1](#threateksfargateprofile1)

**Implemented by:** [Implementation.EKS.FargateProfile.11](#implementationeksfargateprofile11), [Implementation.EKS.FargateProfile.12](#implementationeksfargateprofile12), [Implementation.EKS.FargateProfile.13](#implementationeksfargateprofile13), [ManagedControl.EKS.FargateProfile.1](#managedcontroleksfargateprofile1)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You can tag your Fargate profile through the `$.tags{key}` key-to-value map on `CreateFargateProfile` or later through `TagResource`, independently of your cluster's tags. Owner and purpose tags help track ownership and the business function each cluster's fargate profile supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateFargateProfile` | `$.clusterName`, `$.fargateProfileName`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeFargateProfile` | `$.clusterName`, `$.fargateProfileName` |
| `eks:ListTagsForResource` | `$.resourceArn` |

### Mitigation.EKS.IdentityProviderConfig.1

An Amazon EKS cluster's identity provider config that uses OIDC to authenticate principals should be configured only with trusted OIDC identity providers

**Goal:** [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration)

**Addresses:** [Threat.EKS.IdentityProviderConfig.2](#threateksidentityproviderconfig2), [Threat.EKS.IdentityProviderConfig.3](#threateksidentityproviderconfig3)

**Implemented by:** [Implementation.EKS.IdentityProviderConfig.1](#implementationeksidentityproviderconfig1), [Implementation.EKS.IdentityProviderConfig.2](#implementationeksidentityproviderconfig2)

**Scoping** (for [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration)): An Amazon EKS cluster's identity provider config lets users authenticate to the cluster's Kubernetes API through the OpenID Connect (OIDC) identity provider that the `$.oidc.issuerUrl` request parameter designates. Configuring only trusted OIDC identity providers helps prevent an untrusted identity provider from being used to gain unintended access.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:AssociateIdentityProviderConfig` | `$.clusterName`, `$.oidc.clientId`, `$.oidc.identityProviderConfigName`, `$.oidc.issuerUrl` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeIdentityProviderConfig` | `$.clusterName`, `$.identityProviderConfig.name`, `$.identityProviderConfig.type` |
| `eks:ListIdentityProviderConfigs` | `$.clusterName` |

### Mitigation.EKS.IdentityProviderConfig.2

An Amazon EKS cluster's identity provider config should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.IdentityProviderConfig.1](#threateksidentityproviderconfig1)

**Implemented by:** [Implementation.EKS.IdentityProviderConfig.3](#implementationeksidentityproviderconfig3), [Implementation.EKS.IdentityProviderConfig.4](#implementationeksidentityproviderconfig4), [Implementation.EKS.IdentityProviderConfig.5](#implementationeksidentityproviderconfig5)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You can tag your cluster's identity provider config through the `$.tags{key}` key-to-value map on `AssociateIdentityProviderConfig` or by calling `TagResource`, separately from your cluster's tags. Owner and purpose tags help track ownership and the business function each cluster's identity provider config supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:AssociateIdentityProviderConfig` | `$.clusterName`, `$.oidc.identityProviderConfigName`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeIdentityProviderConfig` | `$.clusterName`, `$.identityProviderConfig.name`, `$.identityProviderConfig.type` |
| `eks:ListTagsForResource` | `$.resourceArn` |

### Mitigation.EKS.Nodegroup.1

An Amazon EKS cluster's nodegroup's node IAM role should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.EKS.Nodegroup.8](#threateksnodegroup8)

**Implemented by:** [Implementation.EKS.Nodegroup.1](#implementationeksnodegroup1)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): The `$.nodeRole` request parameter designates your node group's node IAM role, and code on its instances makes AWS API calls with that role's credentials. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.clusterName`, `$.nodeRole`, `$.nodegroupName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeNodegroup` | `$.clusterName`, `$.nodegroupName` |

### Mitigation.EKS.Nodegroup.2

An Amazon EKS cluster's nodegroup's node IAM role should be assumed only by Amazon EC2 acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.EKS.Nodegroup.7](#threateksnodegroup7)

**Implemented by:** [Implementation.EKS.Nodegroup.2](#implementationeksnodegroup2), [Implementation.EKS.Nodegroup.3](#implementationeksnodegroup3), [Implementation.EKS.Nodegroup.4](#implementationeksnodegroup4), [Implementation.EKS.Nodegroup.5](#implementationeksnodegroup5)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): Your cluster's nodegroup binds its node IAM role through the `$.nodeRole` request parameter, and the role's trust policy specifies which principals can assume it. Allowing only Amazon EC2, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.nodeRole` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeNodegroup` | `$.clusterName`, `$.nodegroupName` |

### Mitigation.EKS.Nodegroup.3

An Amazon EKS cluster's nodegroup should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.EKS.Nodegroup.5](#threateksnodegroup5), [Threat.EKS.Nodegroup.10](#threateksnodegroup10), [Threat.EKS.Nodegroup.11](#threateksnodegroup11)

**Implemented by:** [Implementation.EKS.Nodegroup.6](#implementationeksnodegroup6), [Implementation.EKS.Nodegroup.7](#implementationeksnodegroup7)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): The required `$.subnets[*]` request parameter sets the subnets where your nodegroup's Amazon EC2 Auto Scaling group launches each node and its network interfaces. Private subnet placement helps prevent direct internet routing to your nodegroup's nodes, reducing their network exposure.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.clusterName`, `$.nodegroupName`, `$.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeNodegroup` | `$.clusterName`, `$.nodegroupName` |

### Mitigation.EKS.Nodegroup.4

An Amazon EKS cluster's nodegroup should use an EC2 instance type that supports automatic encryption of instance-to-instance traffic

**Goal:** [Use Nitro instances](../mitigation-goals.md#use-nitro-instances)

**Addresses:** [Threat.EKS.Nodegroup.6](#threateksnodegroup6)

**Implemented by:** [Implementation.EKS.Nodegroup.8](#implementationeksnodegroup8), [Implementation.EKS.Nodegroup.9](#implementationeksnodegroup9)

**Scoping** (for [Use Nitro instances](../mitigation-goals.md#use-nitro-instances)): Your managed node group runs the nodes that host your Pods as Amazon EC2 instances in your AWS account, across the subnets you choose. The instance type you specify for the node group or in its launch template determines whether traffic between your nodes and other instances can be automatically encrypted in transit, which applies between supported instance types in the same Region on a direct network path.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.clusterName`, `$.instanceTypes[*]`, `$.launchTemplate.id`, `$.launchTemplate.name`, `$.launchTemplate.version`, `$.nodegroupName` |
| `eks:UpdateNodegroupVersion` | `$.clusterName`, `$.launchTemplate.version`, `$.nodegroupName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeNodegroup` | `$.clusterName`, `$.nodegroupName` |

### Mitigation.EKS.Nodegroup.5

An Amazon EKS cluster's nodegroup should launch its nodes in subnets in more than one Availability Zone

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.EKS.Nodegroup.2](#threateksnodegroup2)

**Implemented by:** [Implementation.EKS.Nodegroup.10](#implementationeksnodegroup10), [Implementation.EKS.Nodegroup.11](#implementationeksnodegroup11), [Implementation.EKS.Nodegroup.12](#implementationeksnodegroup12)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): The `$.subnets[*]` request parameter lists the subnets where your Amazon EKS managed node group launches its nodes, each with its own network interfaces. Placing those network interfaces in more than one Availability Zone provides network-level redundancy and helps keep connectivity available when one zone is disrupted.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.clusterName`, `$.nodegroupName`, `$.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeNodegroup` | `$.clusterName`, `$.nodegroupName` |

### Mitigation.EKS.Nodegroup.6

An Amazon EKS cluster's nodegroup should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.Nodegroup.1](#threateksnodegroup1)

**Implemented by:** [Implementation.EKS.Nodegroup.13](#implementationeksnodegroup13), [Implementation.EKS.Nodegroup.14](#implementationeksnodegroup14), [Implementation.EKS.Nodegroup.15](#implementationeksnodegroup15)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You can tag your cluster's nodegroup through the `$.tags{key}` key-to-value map on `CreateNodegroup` or `TagResource`, separately from its Auto Scaling group and EC2 instances. Owner and purpose tags help track ownership and the business function each cluster's nodegroup supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.clusterName`, `$.nodegroupName`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeNodegroup` | `$.clusterName`, `$.nodegroupName` |
| `eks:ListTagsForResource` | `$.resourceArn` |

### Mitigation.EKS.Nodegroup.7

An Amazon EKS cluster's nodegroup should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.EKS.Nodegroup.3](#threateksnodegroup3), [Threat.EKS.Nodegroup.4](#threateksnodegroup4), [Threat.EKS.Nodegroup.9](#threateksnodegroup9)

**Implemented by:** [Implementation.EKS.Nodegroup.16](#implementationeksnodegroup16), [Implementation.EKS.Nodegroup.17](#implementationeksnodegroup17), [Implementation.EKS.Nodegroup.18](#implementationeksnodegroup18)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): Your nodegroup's EC2 instances use the cluster security group by default, or can use custom security groups from the launch template referenced through `$.launchTemplate.id` or `$.launchTemplate.name`. Restricting the inbound and outbound rules to the required sources, destinations, protocols, and ports helps reduce the network exposure of your nodegroup.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreateNodegroup` | `$.clusterName`, `$.launchTemplate.id`, `$.launchTemplate.name`, `$.launchTemplate.version`, `$.nodegroupName`, `$.remoteAccess.sourceSecurityGroups[*]` |
| `eks:UpdateNodegroupVersion` | `$.clusterName`, `$.launchTemplate.id`, `$.launchTemplate.name`, `$.launchTemplate.version`, `$.nodegroupName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribeNodegroup` | `$.clusterName`, `$.nodegroupName` |

### Mitigation.EKS.PodIdentityAssociation.1

An Amazon EKS cluster's pod identity association's target IAM role, or its EKS Pod Identity role where no target IAM role is configured, should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.EKS.PodIdentityAssociation.3](#threatekspodidentityassociation3)

**Implemented by:** [Implementation.EKS.PodIdentityAssociation.1](#implementationekspodidentityassociation1)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): Your pod identity association binds a Kubernetes service account to `$.roleArn` (the EKS Pod Identity role), optionally chained to `$.targetRoleArn` (the target IAM role), and your containers in Pods that use that service account call AWS services with the resulting role credentials. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreatePodIdentityAssociation` | `$.clusterName`, `$.policy`, `$.roleArn`, `$.targetRoleArn` |
| `eks:UpdatePodIdentityAssociation` | `$.associationId`, `$.clusterName`, `$.policy`, `$.roleArn`, `$.targetRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribePodIdentityAssociation` | `$.associationId`, `$.clusterName` |
| `eks:ListPodIdentityAssociations` | `$.clusterName` |

### Mitigation.EKS.PodIdentityAssociation.2

An Amazon EKS cluster's pod identity association's EKS Pod Identity role should be assumed only by Amazon EKS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.EKS.PodIdentityAssociation.2](#threatekspodidentityassociation2)

**Implemented by:** [Implementation.EKS.PodIdentityAssociation.2](#implementationekspodidentityassociation2), [Implementation.EKS.PodIdentityAssociation.3](#implementationekspodidentityassociation3), [Implementation.EKS.PodIdentityAssociation.4](#implementationekspodidentityassociation4), [Implementation.EKS.PodIdentityAssociation.5](#implementationekspodidentityassociation5)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): The `$.roleArn` request parameter binds your cluster's pod identity association to an EKS Pod Identity role, whose trust policy specifies which principals can assume it. Allowing only Amazon EKS, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreatePodIdentityAssociation` | `$.roleArn` |
| `eks:UpdatePodIdentityAssociation` | `$.roleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribePodIdentityAssociation` | `$.associationId`, `$.clusterName` |
| `eks:ListPodIdentityAssociations` | `$.clusterName` |

### Mitigation.EKS.PodIdentityAssociation.3

An Amazon EKS cluster's pod identity association should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.EKS.PodIdentityAssociation.1](#threatekspodidentityassociation1)

**Implemented by:** [Implementation.EKS.PodIdentityAssociation.6](#implementationekspodidentityassociation6), [Implementation.EKS.PodIdentityAssociation.7](#implementationekspodidentityassociation7), [Implementation.EKS.PodIdentityAssociation.8](#implementationekspodidentityassociation8)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag your cluster's pod identity association through the `$.tags{key}` key-to-value map on `CreatePodIdentityAssociation` or through `TagResource`, separately from the cluster's own tags. Owner and purpose tags help track ownership and the business function each cluster's pod identity association supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `eks:CreatePodIdentityAssociation` | `$.clusterName`, `$.tags{key}` |
| `eks:TagResource` | `$.resourceArn`, `$.tags{key}` |
| `eks:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `eks:DescribePodIdentityAssociation` | `$.associationId`, `$.clusterName` |

---

## Implementations

### Implementation.EKS.1

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.EKS.1](#mitigationeks1)

**Scoping** (for [Mitigation.EKS.1](#mitigationeks1)): This implementation guidance recommends using your VPC endpoint policy to govern Amazon EKS API calls made through the endpoint with the `aws:ResourceOrgID` condition key. As a result, calls to Amazon EKS resources outside your organization are restricted, while AWS services acting on your behalf (`aws:PrincipalIsAWSService`) remain exempt.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.EKS.2

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.EKS.2](#mitigationeks2)

**Scoping** (for [Mitigation.EKS.2](#mitigationeks2)): This implementation guidance recommends governing Amazon EKS API calls with the `aws:PrincipalOrgID` condition key in your VPC endpoint policy, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, identities outside your organization are restricted from making Amazon EKS API calls through your network.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.EKS.AccessEntry.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::AccessEntry`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1)

**Scoping** (for [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon EKS access entry that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.EKS.AccessEntry.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListAccessEntries` with `$.clusterName`; check `$.accessEntries[*]`
3. Call `eks:DescribeAccessEntry` with `$.clusterName`, `$.principalArn`; check `$.accessEntry.tags{key}`

**Implements:** [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1)

**Scoping** (for [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1)): This implementation guidance recommends reading `$.accessEntry.tags{key}` on each Amazon EKS access entry listed through `$.accessEntries[*]` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon EKS access entry to an accountable owner and the business function it supports.

### Implementation.EKS.AccessEntry.3

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreateAccessEntry` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:access-entry/*/*/*/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:access-entry/*/*/*/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1)

**Scoping** (for [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1)): This implementation guidance recommends restricting `eks:CreateAccessEntry` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS access entry without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EKS.Addon.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Addon`: `$.ConfigurationValues`

**Implements:** [Mitigation.EKS.Addon.1](#mitigationeksaddon1), [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.6](#mitigationeksaddon6)

**Scoping** (for [Mitigation.EKS.Addon.1](#mitigationeksaddon1)): This implementation guidance recommends validating that `$.ConfigurationValues`, the add-on configuration values, set the `enableNetworkPolicy`, `nodeAgent.enablePolicyEventLogs`, and `nodeAgent.enableCloudWatchLogs` keys to `true` before provisioning. That check helps prevent an Amazon EKS cluster's Amazon VPC CNI add-on from being provisioned without configured network policy logging to CloudWatch Logs, which lets you review allow and deny decisions for pod connections.

**Scoping** (for [Mitigation.EKS.Addon.4](#mitigationeksaddon4)): This implementation guidance recommends validating `$.ConfigurationValues`, the add-on's configuration values, to confirm they enable Container Network Observability performance metrics or the AWS Distro for OpenTelemetry (ADOT) Prometheus metrics collector before provisioning. That validation helps prevent an Amazon EKS add-on without its metrics collection setting from being provisioned.

**Scoping** (for [Mitigation.EKS.Addon.6](#mitigationeksaddon6)): This implementation guidance recommends validating `$.ConfigurationValues`, the configuration values of your `amazon-cloudwatch-observability` add-on, for Container Insights with enhanced observability enabled before provisioning. That validation helps address the mitigation by preventing an Amazon EKS add-on from being provisioned without detailed CloudWatch metrics.

### Implementation.EKS.Addon.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `eks:CreateAddon` — `$.addonName` = `vpc-cni`, `$.configurationValues`
- `eks:DeleteAddon` — `$.addonName` = `vpc-cni`
- `eks:UpdateAddon` — `$.addonName` = `vpc-cni`, `$.configurationValues`

**Implements:** [Mitigation.EKS.Addon.1](#mitigationeksaddon1)

**Scoping** (for [Mitigation.EKS.Addon.1](#mitigationeksaddon1)): This implementation guidance recommends recording `UpdateAddon` calls that set `$.configurationValues`, the `vpc-cni` add-on settings that enable network policy logs and their CloudWatch Logs delivery. Recording these calls alongside `DeleteAddon` and the CloudWatch Logs `DeleteLogGroup`, `DeleteLogStream`, and `PutRetentionPolicy` calls helps you detect changes that disable or weaken network policy logging or erase its log records. Recording `CreateAddon` as well lets you detect a `vpc-cni` add-on created without this logging.

### Implementation.EKS.Addon.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListAddons` with `$.clusterName`; check `$.addons[*]`
3. Call `eks:DescribeAddon` with `$.clusterName`, `$.addonName`; check `$.addon.configurationValues`

**Implements:** [Mitigation.EKS.Addon.1](#mitigationeksaddon1)

**Scoping** (for [Mitigation.EKS.Addon.1](#mitigationeksaddon1)): This implementation guidance recommends reading `$.addon.configurationValues`, the configuration values of each Amazon VPC CNI (`vpc-cni`) add-on that `ListAddons` returns for your clusters. Flagging add-ons that do not set `enableNetworkPolicy`, `nodeAgent.enablePolicyEventLogs`, and `nodeAgent.enableCloudWatchLogs` to `"true"` helps you detect gaps in network policy logging to CloudWatch Logs. It does not flag an add-on IAM role that lacks CloudWatch Logs permissions, which stops network policy log delivery while these keys remain configured.

### Implementation.EKS.Addon.4

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.EKS.Addon.2](#mitigationeksaddon2)

**Scoping** (for [Mitigation.EKS.Addon.2](#mitigationeksaddon2)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon EKS cluster's addon's EKS Pod Identity IAM role or, where no pod identity association applies, IAM role for service accounts credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalTag/tag-key` to apply the restriction to principals whose credentials can be used outside your expected networks, tagged with the selected key and value. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)

### Implementation.EKS.Addon.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Addon`: `$.PodIdentityAssociations[*].RoleArn`

**Implements:** [Mitigation.EKS.Addon.3](#mitigationeksaddon3)

**Scoping** (for [Mitigation.EKS.Addon.3](#mitigationeksaddon3)): This implementation guidance recommends validating the trust policy of `$.PodIdentityAssociations[*].RoleArn`, your addon's Pod Identity IAM role, including source conditions on service principals, before provisioning. This helps prevent the addon from being provisioned when that trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.

The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.Addon.6

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `pods.eks.amazonaws.com` | `sts:AssumeRole`, `sts:TagSession` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.EKS.Addon.3](#mitigationeksaddon3)

**Scoping** (for [Mitigation.EKS.Addon.3](#mitigationeksaddon3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon EKS cluster's addon's Pod Identity IAM role, `$.podIdentityAssociations[*].roleArn`, to the `pods.eks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon EKS from assuming the role on behalf of clusters outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.EKS.Addon.7

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.EKS.Addon.3](#mitigationeksaddon3)

**Scoping** (for [Mitigation.EKS.Addon.3](#mitigationeksaddon3)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon EKS cluster's addon's Pod Identity IAM role, `$.podIdentityAssociations[*].roleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.EKS.Addon.8

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `eks:CreateAddon` — `$.addonName`, `$.configurationValues`
- `eks:DeleteAddon` — `$.addonName`, `$.preserve`
- `eks:UpdateAddon` — `$.addonName`, `$.configurationValues`

**Implements:** [Mitigation.EKS.Addon.4](#mitigationeksaddon4)

**Scoping** (for [Mitigation.EKS.Addon.4](#mitigationeksaddon4)): This implementation guidance recommends recording the `UpdateAddon`, `DeleteAddon`, and `CreateAddon` calls on your Amazon EKS add-ons that rewrite or set `$.configurationValues`, the add-on's metrics collection mode such as Container Network Observability performance metrics, or remove the Prometheus Node Exporter, kube-state-metrics, EKS node monitoring agent, AWS Distro for OpenTelemetry (ADOT), SR-IOV Network Metrics Exporter, or Amazon SageMaker HyperPod Observability Add-on. Recording those calls and their calling identity helps you detect a change that stops the detailed telemetry collected outside Amazon CloudWatch Metrics.

### Implementation.EKS.Addon.9

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListAddons` with `$.clusterName`; check `$.addons[*]`
3. Call `eks:DescribeAddon` with `$.clusterName`, `$.addonName`; check `$.addon.status`, `$.addon.configurationValues`

**Implements:** [Mitigation.EKS.Addon.4](#mitigationeksaddon4)

**Scoping** (for [Mitigation.EKS.Addon.4](#mitigationeksaddon4)): This implementation guidance recommends flagging each Amazon EKS cluster whose `ListAddons` response `$.addons[*]`, the cluster's installed add-ons, and `DescribeAddon` response `$.addon.status`, each add-on's status, and `$.addon.configurationValues`, its configuration values, show no active Prometheus Node Exporter, kube-state-metrics, EKS node monitoring agent, SR-IOV Network Metrics Exporter, or SageMaker HyperPod Observability add-on, and no active ADOT or Network Flow Monitor Agent add-on with its Prometheus collector or Container Network Observability OpenMetrics endpoint on, respectively. Flagging them helps you detect resources with no detailed telemetry collected outside Amazon CloudWatch Metrics.

### Implementation.EKS.Addon.10

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Addon`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.EKS.Addon.5](#mitigationeksaddon5)

**Scoping** (for [Mitigation.EKS.Addon.5](#mitigationeksaddon5)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon EKS add-on that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.EKS.Addon.11

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::Addon`: `$.configuration.Tags[*].Key`

**Implements:** [Mitigation.EKS.Addon.5](#mitigationeksaddon5)

**Scoping** (for [Mitigation.EKS.Addon.5](#mitigationeksaddon5)): This implementation guidance recommends reading `$.configuration.Tags[*].Key` on each Amazon EKS add-on and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon EKS add-on to an accountable owner and the business function it supports.

### Implementation.EKS.Addon.12

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreateAddon` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:addon/*/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:addon/*/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.Addon.5](#mitigationeksaddon5)

**Scoping** (for [Mitigation.EKS.Addon.5](#mitigationeksaddon5)): This implementation guidance recommends restricting `eks:CreateAddon` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS add-on without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EKS.Addon.13

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `eks:CreateAddon` — `$.addonName` = `amazon-cloudwatch-observability`, `$.configurationValues`
- `eks:DeleteAddon` — `$.addonName` = `amazon-cloudwatch-observability`
- `eks:UpdateAddon` — `$.addonName` = `amazon-cloudwatch-observability`, `$.configurationValues`

**Implements:** [Mitigation.EKS.Addon.6](#mitigationeksaddon6)

**Scoping** (for [Mitigation.EKS.Addon.6](#mitigationeksaddon6)): This implementation guidance recommends recording `UpdateAddon`, `DeleteAddon`, and `CreateAddon` calls carrying `$.addonName`, the `amazon-cloudwatch-observability` add-on name, and, on `UpdateAddon` and `CreateAddon`, `$.configurationValues`, the add-on's Container Insights settings. Watching those calls helps address the mitigation by surfacing attempts to stop the pod-, node-, and cluster-level metrics of Container Insights with enhanced observability.

### Implementation.EKS.Addon.14

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListAddons` with `$.clusterName`; check `$.addons[*]`
3. Call `eks:DescribeAddon` with `$.clusterName`, `$.addonName`; check `$.addon.addonVersion`, `$.addon.configurationValues`

**Implements:** [Mitigation.EKS.Addon.6](#mitigationeksaddon6)

**Scoping** (for [Mitigation.EKS.Addon.6](#mitigationeksaddon6)): This implementation guidance recommends reading `$.addons[*]`, the add-ons installed on each cluster, `$.addon.addonVersion`, the version of the `amazon-cloudwatch-observability` add-on, and `$.addon.configurationValues`, its configuration values. Flagging any Amazon EKS cluster that lacks this add-on, or whose add-on configuration turns off Container Insights with enhanced observability, helps address the mitigation by identifying where only the default CloudWatch metrics are collected rather than detailed cluster, node, pod, and container metrics.

### Implementation.EKS.Capability.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Capability`: `$.RoleArn`

**Implements:** [Mitigation.EKS.Capability.1](#mitigationekscapability1)

**Scoping** (for [Mitigation.EKS.Capability.1](#mitigationekscapability1)): This implementation guidance recommends validating the trust policy of `$.RoleArn`, your capability's IAM role, including source conditions on service principals, before provisioning. This helps prevent the capability from being provisioned when that trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one sts:AssumeRole statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.Capability.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListCapabilities` with `$.clusterName`; check `$.capabilities[*].capabilityName`
3. Call `eks:DescribeCapability` with `$.clusterName`, `$.capabilityName`; check `$.capability.roleArn`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
7. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.EKS.Capability.1](#mitigationekscapability1)

**Scoping** (for [Mitigation.EKS.Capability.1](#mitigationekscapability1)): This implementation guidance recommends reading `$.capability.roleArn`, your capability's IAM role, then evaluating `$.Role.AssumeRolePolicyDocument`, that role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one sts:AssumeRole statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.Capability.3

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `capabilities.eks.amazonaws.com` | `sts:AssumeRole`, `sts:TagSession` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.EKS.Capability.1](#mitigationekscapability1)

**Scoping** (for [Mitigation.EKS.Capability.1](#mitigationekscapability1)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon EKS cluster's capability's IAM role, `$.roleArn` on `CreateCapability` and `UpdateCapability`, to the `capabilities.eks.amazonaws.com` service principal with the `aws:SourceAccount` and `aws:SourceArn` condition keys. These conditions help prevent Amazon EKS from assuming the role on behalf of capabilities outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.EKS.Capability.4

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.EKS.Capability.1](#mitigationekscapability1)

**Scoping** (for [Mitigation.EKS.Capability.1](#mitigationekscapability1)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon EKS cluster's capability's IAM role, `$.roleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.EKS.Capability.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Capability`: `$.Configuration.ArgoCd.AwsIdc.IdcInstanceArn`, `$.Configuration.ArgoCd.AwsIdc.IdcRegion`

**Implements:** [Mitigation.EKS.Capability.2](#mitigationekscapability2)

**Scoping** (for [Mitigation.EKS.Capability.2](#mitigationekscapability2)): This implementation guidance recommends validating `$.Configuration.ArgoCd.AwsIdc.IdcInstanceArn` and `$.Configuration.ArgoCd.AwsIdc.IdcRegion`, the IAM Identity Center instance and Region that your capability uses to authenticate users to the hosted Argo CD UI and API, before provisioning. Requiring trusted IAM Identity Center instances helps prevent an Amazon EKS cluster's capability that designates an untrusted provider from being provisioned.

### Implementation.EKS.Capability.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListCapabilities` with `$.clusterName`; check `$.capabilities[*].capabilityName`, `$.capabilities[*].type`
3. Call `eks:DescribeCapability` with `$.clusterName`, `$.capabilityName`; check `$.capability.configuration.argoCd.awsIdc.idcInstanceArn`, `$.capability.configuration.argoCd.awsIdc.idcRegion`

**Implements:** [Mitigation.EKS.Capability.2](#mitigationekscapability2)

**Scoping** (for [Mitigation.EKS.Capability.2](#mitigationekscapability2)): This implementation guidance recommends reading `$.capability.configuration.argoCd.awsIdc.idcInstanceArn` and `$.capability.configuration.argoCd.awsIdc.idcRegion`, the ARN and Region of the IAM Identity Center instance that authenticates users to your Argo CD UI and API. Flagging an Argo CD capability whose recorded provider you do not trust helps you detect an authentication path that could allow unintended access.

### Implementation.EKS.Capability.7

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Capability`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.EKS.Capability.3](#mitigationekscapability3)

**Scoping** (for [Mitigation.EKS.Capability.3](#mitigationekscapability3)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon EKS capability that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.EKS.Capability.8

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListCapabilities` with `$.clusterName`; check `$.capabilities[*].arn`
3. Call `eks:ListTagsForResource` with `$.resourceArn`; check `$.tags{key}`

**Implements:** [Mitigation.EKS.Capability.3](#mitigationekscapability3)

**Scoping** (for [Mitigation.EKS.Capability.3](#mitigationekscapability3)): This implementation guidance recommends reading `$.tags{key}` on each Amazon EKS capability listed through `$.capabilities[*].arn` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon EKS capability to an accountable owner and the business function it supports.

### Implementation.EKS.Capability.9

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreateCapability` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:capability/*/*/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:capability/*/*/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.Capability.3](#mitigationekscapability3)

**Scoping** (for [Mitigation.EKS.Capability.3](#mitigationekscapability3)): This implementation guidance recommends restricting `eks:CreateCapability` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS capability without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EKS.Capability.10

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteDelivery` — `$.id`
- `cloudwatch-logs:DeleteDeliveryDestination` — `$.name`
- `cloudwatch-logs:DeleteDeliverySource` — `$.name`
- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteResourcePolicy`
- `cloudwatch-logs:PutDeliveryDestination` — `$.deliveryDestinationConfiguration.destinationResourceArn`, `$.name`
- `cloudwatch-logs:PutDeliverySource` — `$.logType`, `$.name`, `$.resourceArn`
- `cloudwatch-logs:PutResourcePolicy`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `cloudwatch-logs:UpdateDeliveryConfiguration` — `$.id`, `$.recordFields[*]`

**Implements:** [Mitigation.EKS.Capability.4](#mitigationekscapability4)

**Scoping** (for [Mitigation.EKS.Capability.4](#mitigationekscapability4)): This implementation guidance recommends recording the CloudWatch Logs operations that can disable or weaken the vended logs delivery for your Amazon EKS cluster's capability. Recording calls such as `DeleteDelivery`, `DeleteDeliverySource`, and `PutRetentionPolicy`, together with the calling identity, helps you detect and attribute those changes.

### Implementation.EKS.Capability.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListCapabilities` with `$.clusterName`; check `$.capabilities[*].arn`, `$.capabilities[*].type`
3. Call `cloudwatch-logs:DescribeDeliverySources` with `$.resourceArns[*]`; check `$.deliverySources[*].resourceArns[*]`, `$.deliverySources[*].name`, `$.deliverySources[*].logType`
4. Call `cloudwatch-logs:DescribeDeliveries` with `$.deliverySourceNames[*]`; check `$.deliveries[*].deliverySourceName`, `$.deliveries[*].deliveryDestinationType` = `CWL`

**Implements:** [Mitigation.EKS.Capability.4](#mitigationekscapability4)

**Scoping** (for [Mitigation.EKS.Capability.4](#mitigationekscapability4)): This implementation guidance recommends flagging an Amazon EKS capability with no vended logs delivery to CloudWatch Logs, read from `$.deliveries[*].deliveryDestinationType`, the delivery's destination type. Surfacing that gap helps you detect ACK, kro, and Argo CD controller activity that is not being recorded for review.

### Implementation.EKS.Cluster.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.Logging.ClusterLogging.EnabledTypes[*].Type`

**Implements:** [Mitigation.EKS.Cluster.1](#mitigationekscluster1)

**Scoping** (for [Mitigation.EKS.Cluster.1](#mitigationekscluster1)): This implementation guidance recommends validating that `$.Logging.ClusterLogging.EnabledTypes[*].Type`, the control plane log types your cluster exports, includes `api`, `audit`, `authenticator`, `controllerManager`, and `scheduler` before provisioning. That check helps prevent an Amazon EKS cluster from being provisioned without configured control plane logging to CloudWatch Logs, for auditing access and changes.

### Implementation.EKS.Cluster.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `eks:CreateCluster` — `$.logging.clusterLogging[*].enabled`, `$.logging.clusterLogging[*].types[*]`
- `eks:UpdateClusterConfig` — `$.logging.clusterLogging[*].enabled`, `$.logging.clusterLogging[*].types[*]`

**Implements:** [Mitigation.EKS.Cluster.1](#mitigationekscluster1)

**Scoping** (for [Mitigation.EKS.Cluster.1](#mitigationekscluster1)): This implementation guidance recommends recording `UpdateClusterConfig` calls that set `$.logging.clusterLogging[*].enabled`, whether logs export to CloudWatch Logs, and `$.logging.clusterLogging[*].types[*]`, the log types that setting covers. Recording these calls alongside `DeleteLogGroup`, `DeleteLogStream`, and `PutRetentionPolicy` on the destination log group helps you detect changes that disable or weaken control plane logging or erase its log records. Recording `CreateCluster` as well lets you detect an Amazon EKS cluster created without this logging.

### Implementation.EKS.Cluster.3

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::Cluster`: `$.configuration.Logging.ClusterLogging.EnabledTypes[*].Type`

**Implements:** [Mitigation.EKS.Cluster.1](#mitigationekscluster1)

**Scoping** (for [Mitigation.EKS.Cluster.1](#mitigationekscluster1)): This implementation guidance recommends reading `$.configuration.Logging.ClusterLogging.EnabledTypes[*].Type`, the control plane log types enabled for export, from each Amazon EKS cluster's configuration item. Flagging clusters with any log type disabled helps you detect gaps in API server, audit, authenticator, controller manager, and scheduler logging to CloudWatch Logs.

### Implementation.EKS.Cluster.4

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |
| `*` | `*` | `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService`, `ec2:SourceInstanceARN` (alternatives: `aws:PrincipalTag/dp:include:network`) |

**Implements:** [Mitigation.EKS.Cluster.2](#mitigationekscluster2)

**Scoping** (for [Mitigation.EKS.Cluster.2](#mitigationekscluster2)): This implementation guidance recommends applying the `ec2:SourceInstanceARN`, `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon EKS cluster's Auto Mode node IAM role and EKS Connector agent IAM role credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `ec2:SourceInstanceARN` to apply the restriction to the role sessions whose credentials can be used outside your expected networks. Alternatively, you can use `aws:PrincipalTag/tag-key`, tagging the roles with the tag key and value your IAM policy management strategy uses. For credentials delivered to an Amazon EC2 instance, you can use `aws:Ec2InstanceSourceVpc` and `aws:Ec2InstanceSourcePrivateIPv4` to bind them to the instance they were issued to. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [How to use policies to restrict where EC2 instance credentials can be used from](https://aws.amazon.com/blogs/security/how-to-use-policies-to-restrict-where-ec2-instance-credentials-can-be-used-from/)
- [SCP sample: network perimeter policy for Amazon EC2](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/network_perimeter_ec2_scp.json)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)

### Implementation.EKS.Cluster.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.RoleArn`, `$.ComputeConfig.NodeRoleArn`

**Implements:** [Mitigation.EKS.Cluster.3](#mitigationekscluster3)

**Scoping** (for [Mitigation.EKS.Cluster.3](#mitigationekscluster3)): This implementation guidance recommends validating the trust policies of `$.RoleArn` and `$.ComputeConfig.NodeRoleArn`, your cluster's service role and Auto Mode node IAM role, including source conditions on service principals, before provisioning. This helps prevent the cluster from being provisioned when any trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.Cluster.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters` with `$.include[*]`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.roleArn`, `$.cluster.computeConfig.nodeRoleArn`, `$.cluster.connectorConfig.roleArn`, `$.cluster.outpostConfig.outpostArns[*]`
3. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
5. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.EKS.Cluster.3](#mitigationekscluster3)

**Scoping** (for [Mitigation.EKS.Cluster.3](#mitigationekscluster3)): This implementation guidance recommends reading `$.cluster.roleArn`, `$.cluster.computeConfig.nodeRoleArn`, and `$.cluster.connectorConfig.roleArn`, your cluster's service role, Auto Mode node IAM role, and connector agent IAM role, then evaluating `$.Role.AssumeRolePolicyDocument`, each role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys on your service role and connector agent IAM role, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.Cluster.7

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ec2.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.EKS.Cluster.3](#mitigationekscluster3)

**Scoping** (for [Mitigation.EKS.Cluster.3](#mitigationekscluster3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon EKS cluster's Auto Mode node IAM role, `$.computeConfig.nodeRoleArn`, to the `ec2.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.EKS.Cluster.8

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ssm.amazonaws.com` | `sts:AssumeRole` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.EKS.Cluster.3](#mitigationekscluster3)

**Scoping** (for [Mitigation.EKS.Cluster.3](#mitigationekscluster3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon EKS cluster's connector agent IAM role, `$.connectorConfig.roleArn`, to the `ssm.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent AWS Systems Manager from assuming the role on behalf of Systems Manager resources in accounts outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.EKS.Cluster.9

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `eks.amazonaws.com` | `sts:AssumeRole`, `sts:TagSession` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.EKS.Cluster.3](#mitigationekscluster3)

**Scoping** (for [Mitigation.EKS.Cluster.3](#mitigationekscluster3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon EKS cluster's service role, `$.roleArn`, to the `eks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon EKS from assuming the role on behalf of clusters outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.EKS.Cluster.10

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.EKS.Cluster.3](#mitigationekscluster3)

**Scoping** (for [Mitigation.EKS.Cluster.3](#mitigationekscluster3)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon EKS cluster's service role, Auto Mode node IAM role, and connector agent IAM role, `$.roleArn` and `$.computeConfig.nodeRoleArn` and `$.connectorConfig.roleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.EKS.Cluster.11

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.ResourcesVpcConfig.SubnetIds[*]`, `$.KubernetesNetworkConfig.IpFamily`

**Implements:** [Mitigation.EKS.Cluster.4](#mitigationekscluster4)

**Scoping** (for [Mitigation.EKS.Cluster.4](#mitigationekscluster4)): This implementation guidance recommends validating `$.ResourcesVpcConfig.SubnetIds[*]`, the subnets that hold the cluster's network interfaces, by resolving each subnet's route table for a default route to an internet gateway before provisioning. Subnets carrying that route fail validation, which helps prevent an Amazon EKS cluster from being provisioned in a public subnet.

### Implementation.EKS.Cluster.12

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.resourcesVpcConfig.subnetIds[*]`, `$.cluster.resourcesVpcConfig.vpcId`
3. Call `ec2:DescribeRouteTables` with `$.Filter`; check `$.routeTableSet[*].routeSet[*].gatewayId`, `$.routeTableSet[*].routeSet[*].destinationCidrBlock` = `0.0.0.0/0`, `$.routeTableSet[*].routeSet[*].destinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.EKS.Cluster.4](#mitigationekscluster4)

**Scoping** (for [Mitigation.EKS.Cluster.4](#mitigationekscluster4)): This implementation guidance recommends reading `$.cluster.resourcesVpcConfig.subnetIds[*]`, the subnets hosting the cluster's network interfaces, then resolving each subnet's route table. Flagging an Amazon EKS cluster with a subnet whose default route targets an internet gateway, read at `$.routeTableSet[*].routeSet[*].gatewayId`, helps you detect placement in a public subnet.

### Implementation.EKS.Cluster.13

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.ComputeConfig.Enabled`

**Implements:** [Mitigation.EKS.Cluster.5](#mitigationekscluster5)

**Scoping** (for [Mitigation.EKS.Cluster.5](#mitigationekscluster5)): This implementation guidance recommends validating that `$.ComputeConfig.Enabled`, the EKS Auto Mode compute capability that includes the EKS node monitoring agent, is `true` before provisioning. That validation helps prevent an Amazon EKS cluster without the EKS Auto Mode compute capability from being provisioned.

### Implementation.EKS.Cluster.14

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `eks:CreateCluster` — `$.computeConfig.enabled`
- `eks:UpdateClusterConfig` — `$.computeConfig.enabled` = `false`

**Implements:** [Mitigation.EKS.Cluster.5](#mitigationekscluster5)

**Scoping** (for [Mitigation.EKS.Cluster.5](#mitigationekscluster5)): This implementation guidance recommends recording the `UpdateClusterConfig` and `CreateCluster` calls on your Amazon EKS cluster that set `$.computeConfig.enabled`, the EKS Auto Mode compute capability whose nodes run the EKS node monitoring agent. Recording those calls and their calling identity helps you detect a change that stops the detailed telemetry collected outside Amazon CloudWatch Metrics.

### Implementation.EKS.Cluster.15

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.computeConfig.enabled`

**Implements:** [Mitigation.EKS.Cluster.5](#mitigationekscluster5)

**Scoping** (for [Mitigation.EKS.Cluster.5](#mitigationekscluster5)): This implementation guidance recommends flagging each Amazon EKS cluster whose `DescribeCluster` response `$.cluster.computeConfig.enabled`, the setting that turns on EKS Auto Mode compute and its built-in EKS node monitoring agent, is absent or false. Flagging them helps you detect resources with no detailed telemetry collected outside Amazon CloudWatch Metrics.

### Implementation.EKS.Cluster.16

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.OutpostConfig.ControlPlaneInstanceType`, `$.OutpostConfig.EtcdInstanceType`

**Implements:** [Mitigation.EKS.Cluster.6](#mitigationekscluster6)

**Scoping** (for [Mitigation.EKS.Cluster.6](#mitigationekscluster6)): This implementation guidance recommends validating `$.OutpostConfig.ControlPlaneInstanceType` and `$.OutpostConfig.EtcdInstanceType`, the instance types of your local cluster's control plane and etcd instances, before provisioning, which helps prevent an Amazon EKS cluster configured with instance types without automatic instance-to-instance encryption from being provisioned. That helps provide an additional layer of encryption in transit between your control plane and etcd instances without application changes.

### Implementation.EKS.Cluster.17

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.outpostConfig.controlPlaneInstanceType`, `$.cluster.outpostConfig.etcdInstanceType`
3. Call `ec2:DescribeInstanceTypes` with `$.InstanceType[*]`; check `$.instanceTypeSet[*].instanceType`, `$.instanceTypeSet[*].networkInfo.encryptionInTransitSupported`

**Implements:** [Mitigation.EKS.Cluster.6](#mitigationekscluster6)

**Scoping** (for [Mitigation.EKS.Cluster.6](#mitigationekscluster6)): This implementation guidance recommends comparing the control plane and etcd instance types of your local Amazon EKS cluster at `$.cluster.outpostConfig.controlPlaneInstanceType` and `$.cluster.outpostConfig.etcdInstanceType` with EC2 encryption-in-transit support at `$.instanceTypeSet[*].networkInfo.encryptionInTransitSupported`. That comparison flags a local Amazon EKS cluster configured with instance types without automatic encryption of instance-to-instance traffic.

### Implementation.EKS.Cluster.18

*Guidance — Directive*

An Amazon EKS cluster in another account can map one of your IAM principals through an access entry or the `aws-auth` ConfigMap. Your principal can then send requests to that cluster's Kubernetes API server endpoint. The AWS IAM Authenticator for Kubernetes authenticates your principal from a presigned AWS STS `GetCallerIdentity` token. This handshake needs no IAM permissions. Kubernetes RBAC and Amazon EKS access policies, not IAM, then authorize each request. No IAM action is evaluated on this access path once access has been granted, so IAM policy conditions cannot restrict which resources your identities access. Use infrastructure security controls such as route tables, egress proxies, firewalls, and DNS filtering so that requests from your networks can reach only trusted resources.

**Implements:** [Mitigation.EKS.Cluster.7](#mitigationekscluster7)

**Scoping** (for [Mitigation.EKS.Cluster.7](#mitigationekscluster7)): This implementation guidance recommends infrastructure controls such as route tables, egress proxies, firewalls, and DNS filtering for requests to the Kubernetes API server endpoint of an Amazon EKS cluster in another account. Once access is granted, no IAM action is evaluated on that path, so those controls, not policy conditions, help your identities access only trusted resources.

### Implementation.EKS.Cluster.19

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `backup:DeleteBackupPlan` — `$.BackupPlanId`
- `backup:DeleteBackupSelection` — `$.BackupPlanId`, `$.SelectionId`
- `backup:DeleteRecoveryPoint` — `$.RecoveryPointArn`
- `backup:DisassociateRecoveryPointFromParent` — `$.RecoveryPointArn`
- `backup:StopBackupJob`
- `backup:UpdateBackupPlan` — `$.BackupPlan.Rules[*].Lifecycle.DeleteAfterDays`, `$.BackupPlan.Rules[*].ScheduleExpression`, `$.BackupPlan.Rules[*].TargetBackupVaultName`, `$.BackupPlanId`
- `backup:UpdateRecoveryPointLifecycle` — `$.Lifecycle.DeleteAfterDays`, `$.RecoveryPointArn`
- `backup:UpdateRegionSettings` — `$.ResourceTypeOptInPreference{key}`
- `eks:TagResource` — `$.resourceArn`, `$.tags{key}`
- `eks:UntagResource` — `$.resourceArn`, `$.tagKeys[*]`

**Implements:** [Mitigation.EKS.Cluster.8](#mitigationekscluster8)

**Scoping** (for [Mitigation.EKS.Cluster.8](#mitigationekscluster8)): This implementation guidance recommends recording the AWS Backup operations `DeleteBackupSelection`, `DeleteBackupPlan`, `DeleteRecoveryPoint`, `DisassociateRecoveryPointFromParent`, `StopBackupJob`, `UpdateBackupPlan`, `UpdateRecoveryPointLifecycle`, and `UpdateRegionSettings`, and Amazon EKS `UntagResource` and `TagResource` calls that change the tags your backup selection matches. Recording these calls helps you detect deleted plans, selections, or recovery points, stopped jobs, weakened rules or retention, or lost tag-based selection for your Amazon EKS cluster.

### Implementation.EKS.Cluster.20

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.arn`, `$.cluster.tags{key}`
3. Call `backup:ListBackupPlans`; check `$.BackupPlansList[*].BackupPlanId`
4. Call `backup:ListBackupSelections` with `$.BackupPlanId`; check `$.BackupSelectionsList[*].SelectionId`
5. Call `backup:GetBackupSelection` with `$.BackupPlanId`, `$.SelectionId`; check `$.BackupSelection.Resources[*]`, `$.BackupSelection.NotResources[*]`, `$.BackupSelection.ListOfTags[*]`, `$.BackupSelection.Conditions`
6. Call `backup:DescribeRegionSettings`; check `$.ResourceTypeOptInPreference{key}`

**Implements:** [Mitigation.EKS.Cluster.8](#mitigationekscluster8)

**Scoping** (for [Mitigation.EKS.Cluster.8](#mitigationekscluster8)): This implementation guidance recommends comparing your Amazon EKS clusters against each backup selection's `$.BackupSelection.Resources[*]` (resource ARN patterns), `$.BackupSelection.NotResources[*]` (excluded ARN patterns), `$.BackupSelection.ListOfTags[*]` (tag conditions), and `$.BackupSelection.Conditions` (tag conditions). Flagging any Amazon EKS cluster that no selection covers helps you detect Kubernetes cluster state and persistent volumes left without a centrally governed recovery point.

### Implementation.EKS.Cluster.21

*Declarative policy — Preventive*

Governs the `BACKUP_POLICY` declarative policy attributes below.

- `$.plans.EKS_Backup_Plan.selections.resources.EKS_Cluster_Selection.not_resource_types`
- `$.plans.EKS_Backup_Plan.selections.resources.EKS_Cluster_Selection.resource_types`

**Implements:** [Mitigation.EKS.Cluster.8](#mitigationekscluster8)

**Scoping** (for [Mitigation.EKS.Cluster.8](#mitigationekscluster8)): This implementation guidance recommends an AWS Organizations backup policy that declares `$.plans.EKS_Backup_Plan.selections.resources.EKS_Cluster_Selection.resource_types`, the list of resource types a backup plan protects, and `$.plans.EKS_Backup_Plan.selections.resources.EKS_Cluster_Selection.not_resource_types`, the resource types it excludes. Listing the Amazon EKS cluster resource type without excluding it assigns matching resources to the plan for scheduled, centrally governed recovery points.

### Implementation.EKS.Cluster.22

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.EncryptionConfig[*].Provider.KeyArn`

**Implements:** [Mitigation.EKS.Cluster.9](#mitigationekscluster9)

**Scoping** (for [Mitigation.EKS.Cluster.9](#mitigationekscluster9)): This implementation guidance recommends validating `$.EncryptionConfig[*].Provider.KeyArn`, the KMS key used for envelope encryption of your Amazon EKS cluster's Kubernetes API data, before provisioning. Requiring a customer managed key helps prevent an Amazon EKS cluster from being provisioned without a key whose policy and lifecycle you control.

### Implementation.EKS.Cluster.23

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::Cluster`: `$.configuration.EncryptionConfig[*].Provider.KeyArn`, `$.configuration.EncryptionConfigKeyArn`

**Implements:** [Mitigation.EKS.Cluster.9](#mitigationekscluster9)

**Scoping** (for [Mitigation.EKS.Cluster.9](#mitigationekscluster9)): This implementation guidance recommends reading `$.configuration.EncryptionConfig[*].Provider.KeyArn` and `$.configuration.EncryptionConfigKeyArn`, the envelope encryption key for your cluster's Kubernetes API data, and resolving `$.KeyMetadata.KeyManager` for that key. Flagging Amazon EKS clusters not protected by a customer managed key helps you detect where you do not control the key policy and lifecycle.

### Implementation.EKS.Cluster.24

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:AssociateEncryptionConfig`, `eks:CreateCluster` | `*` | `eks:encryptionConfigProviderKeyArns` |
| `eks:CreateCluster` | `*` | `eks:encryptionConfigProviderKeyArns` |

**Implements:** [Mitigation.EKS.Cluster.9](#mitigationekscluster9)

**Scoping** (for [Mitigation.EKS.Cluster.9](#mitigationekscluster9)): This implementation guidance recommends applying the `eks:encryptionConfigProviderKeyArns` condition key in a service control policy (SCP) to govern `eks:CreateCluster` and `eks:AssociateEncryptionConfig`. This condition helps prevent creating or modifying an Amazon EKS cluster without an approved customer managed key at `$.encryptionConfig[*].provider.keyArn`, your cluster's envelope encryption key.

### Implementation.EKS.Cluster.25

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.AccessConfig.AuthenticationMode`

**Implements:** [Mitigation.EKS.Cluster.10](#mitigationekscluster10)

**Scoping** (for [Mitigation.EKS.Cluster.10](#mitigationekscluster10)): This implementation guidance recommends validating that `$.AccessConfig.AuthenticationMode`, your cluster's authentication mode, is set to `API` or `API_AND_CONFIG_MAP` to enable EKS access entries before provisioning. This validation helps prevent an Amazon EKS cluster that relies only on the `aws-auth` ConfigMap for IAM principal access from being provisioned.

### Implementation.EKS.Cluster.26

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.accessConfig.authenticationMode`

**Implements:** [Mitigation.EKS.Cluster.10](#mitigationekscluster10)

**Scoping** (for [Mitigation.EKS.Cluster.10](#mitigationekscluster10)): This implementation guidance recommends flagging an Amazon EKS cluster whose `$.cluster.accessConfig.authenticationMode`, the cluster's authentication mode, selects only the in-cluster `aws-auth` ConfigMap method (`CONFIG_MAP`). Flagging these clusters helps you detect where IAM principals can be granted Kubernetes API permissions through `aws-auth` ConfigMap edits instead of EKS access entries.

### Implementation.EKS.Cluster.27

*Guidance — Directive*

Once a caller is authenticated to your cluster's Kubernetes API server endpoint, no IAM action is evaluated on that access path. Amazon EKS access policies and Kubernetes RBAC authorize requests on this path. As a result, IAM policy conditions cannot restrict which identities access your cluster through this endpoint. Use infrastructure security controls such as route tables, security groups, network access control lists, and firewalls so that only requests from expected networks reach it. Amazon EKS provides its own endpoint access settings for this path. Endpoint public access (`endpointPublicAccess`) is enabled by default and makes the API server accessible from the internet. If you keep it enabled, set the public access CIDRs (`publicAccessCidrs`) to the CIDR blocks of your expected networks. If you specify no CIDR blocks, the public endpoint receives requests from all IP addresses. Without endpoint private access, these CIDR blocks must also include the egress sources from your VPC, such as your NAT gateway addresses. Endpoint private access (`endpointPrivateAccess`) lets Kubernetes API requests from within your cluster's VPC use the private endpoint. You must enable private access before you can disable public access. With public access disabled, all traffic to the API server must come from within your cluster's VPC or a connected network. You use the rules on the cluster security group to control access to the private endpoint. Amazon EKS also associates any security groups that you specify (`securityGroupIds`) with the network interfaces that it creates for your cluster. Route tables, network access control lists, and firewalls in your VPC and connected networks can also govern which sources reach the private endpoint. The public access CIDRs and the cluster security group each affect only their respective endpoint. These network controls complement, and do not replace, the access entries, access policies, and Kubernetes RBAC that govern access to your cluster.

**Implements:** [Mitigation.EKS.Cluster.11](#mitigationekscluster11)

**Scoping** (for [Mitigation.EKS.Cluster.11](#mitigationekscluster11)): This implementation guidance recommends infrastructure controls such as route tables, security groups, network access control lists, and firewalls for your cluster's Kubernetes API server endpoint. Once the caller is authenticated, no IAM action is evaluated on that path, so those controls, not policy conditions, help restrict who can reach your resource.

### Implementation.EKS.Cluster.28

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::AccessEntry`: `$.PrincipalArn`

**Implements:** [Mitigation.EKS.Cluster.11](#mitigationekscluster11)

**Scoping** (for [Mitigation.EKS.Cluster.11](#mitigationekscluster11)): This implementation guidance recommends validating `$.PrincipalArn`, the IAM principal named in an Amazon EKS access entry, before provisioning the access entry. The hook helps prevent an access entry that grants a principal outside your zone of trust access to your cluster from being provisioned.

### Implementation.EKS.Cluster.29

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.arn`, `$.cluster.accessConfig.authenticationMode`, `$.cluster.accessConfig.bootstrapClusterCreatorAdminPermissions`
3. Call `eks:ListAccessEntries` with `$.clusterName`; check `$.accessEntries[*]`
4. Call `eks:DescribeAccessEntry` with `$.clusterName`, `$.principalArn`; check `$.accessEntry.principalArn`, `$.accessEntry.type`, `$.accessEntry.kubernetesGroups[*]`, `$.accessEntry.username`

**Implements:** [Mitigation.EKS.Cluster.11](#mitigationekscluster11)

**Scoping** (for [Mitigation.EKS.Cluster.11](#mitigationekscluster11)): This implementation guidance recommends reading `$.accessEntry.principalArn`, the IAM principal that each access entry allows to access your cluster, and `$.cluster.accessConfig.authenticationMode`, the setting that determines which methods can grant IAM principals access, and evaluating it for access entries that grant access from outside your zone of trust. Flagging those grants helps you detect Kubernetes API access to your Amazon EKS cluster that extends beyond trusted identities.

### Implementation.EKS.Cluster.30

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.EKS.Cluster.12](#mitigationekscluster12)

**Scoping** (for [Mitigation.EKS.Cluster.12](#mitigationekscluster12)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon EKS cluster that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.EKS.Cluster.31

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::Cluster`: `$.tags[*].key`

**Implements:** [Mitigation.EKS.Cluster.12](#mitigationekscluster12)

**Scoping** (for [Mitigation.EKS.Cluster.12](#mitigationekscluster12)): This implementation guidance recommends reading `$.tags[*].key` on each Amazon EKS cluster and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon EKS cluster to an accountable owner and the business function it supports.

### Implementation.EKS.Cluster.32

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreateCluster`, `eks:RegisterCluster` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:cluster/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:cluster/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.Cluster.12](#mitigationekscluster12)

**Scoping** (for [Mitigation.EKS.Cluster.12](#mitigationekscluster12)): This implementation guidance recommends restricting `eks:CreateCluster` and `eks:RegisterCluster` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS cluster without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EKS.Cluster.33

*Guidance — Directive*

Configure the security groups associated with your Amazon EKS cluster following least-privilege best practices. These include the cluster security group that Amazon EKS creates. They also include any additional security groups that you specify in `$.resourcesVpcConfig.securityGroupIds[*]` when you call `CreateCluster` or `UpdateClusterConfig`. Amazon EKS associates these security groups with the cluster's cross-account elastic network interfaces. Their rules control connections to the private Kubernetes API server endpoint. They also control the control plane's connections through your VPC to the kubelet API, webhooks, and other endpoints it calls. They don't control access to the cluster's public endpoint. Restrict both inbound and outbound rules to only the required sources and destinations (IP address ranges, referenced security groups, or prefix lists), protocols, and ports. The cluster security group includes a default outbound rule to 0.0.0.0/0 (IPv4) or ::/0 (IPv6). If your cluster doesn't need that rule, you can remove it while keeping the minimum rules that Amazon EKS requires. Where the peer is an AWS resource in a security group, such as your nodes, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change.

To govern security groups at scale, use AWS Firewall Manager security group policies across your organization in AWS Organizations. Scope each policy by account, organizational unit (OU), or tag. A common security group policy applies a baseline security group. Firewall Manager doesn't support updating security groups on requester-managed network interfaces, such as the ones that Amazon EKS creates for your cluster. A content audit security group policy audits inbound and outbound rules against an allow list or a block list. It flags noncompliant security groups in a central dashboard, sends notifications, and can automatically remediate noncompliant rules. A usage audit security group policy removes unused or redundant security groups.

To verify unintended access that your security groups permit, use Amazon VPC Network Access Analyzer. Define a Network Access Scope (`CreateNetworkInsightsAccessScope`) for the access that you consider unintended. For example, match any path that reaches administrative ports such as 22 or 3389 from an unrestricted address. Then run an analysis (`StartNetworkInsightsAccessScopeAnalysis`) and review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.EKS.Cluster.13](#mitigationekscluster13)

**Scoping** (for [Mitigation.EKS.Cluster.13](#mitigationekscluster13)): This implementation guidance recommends restricting the inbound and outbound rules on your Amazon EKS cluster's security groups to the required addresses, protocols, and ports. Excluding unrestricted addresses such as 0.0.0.0/0 or ::/0 from those inbound and outbound rules helps reduce the network exposure of your cluster's cross-account elastic network interfaces.

### Implementation.EKS.Cluster.34

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.ResourcesVpcConfig.SecurityGroupIds[*]`

**Implements:** [Mitigation.EKS.Cluster.13](#mitigationekscluster13)

**Scoping** (for [Mitigation.EKS.Cluster.13](#mitigationekscluster13)): This implementation guidance recommends validating the inbound and outbound rules of the security groups resolved from `$.ResourcesVpcConfig.SecurityGroupIds[*]` before provisioning. Checking for rules that allow any address on unintended ports, directly or through a prefix list, helps prevent an Amazon EKS cluster from being provisioned with such groups.

### Implementation.EKS.Cluster.35

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.resourcesVpcConfig.clusterSecurityGroupId`, `$.cluster.resourcesVpcConfig.securityGroupIds[*]`
3. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
4. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr`

**Implements:** [Mitigation.EKS.Cluster.13](#mitigationekscluster13)

**Scoping** (for [Mitigation.EKS.Cluster.13](#mitigationekscluster13)): This implementation guidance recommends resolving the security groups recorded at `$.cluster.resourcesVpcConfig.clusterSecurityGroupId` and `$.cluster.resourcesVpcConfig.securityGroupIds[*]` and inspecting their inbound and outbound rules. Flagging rules that admit 0.0.0.0/0 or ::/0 on sensitive ports, broad port ranges, or all protocols helps you detect your Amazon EKS clusters with broader network exposure than they need.

### Implementation.EKS.Cluster.36

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Cluster`: `$.DeletionProtection`

**Implements:** [Mitigation.EKS.Cluster.14](#mitigationekscluster14)

**Scoping** (for [Mitigation.EKS.Cluster.14](#mitigationekscluster14)): This implementation guidance recommends validating, before provisioning, that `$.DeletionProtection`, the deletion protection setting, is enabled. Stack operations that leave it disabled fail, helping prevent a cluster from being provisioned without deletion protection.

### Implementation.EKS.Cluster.37

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `eks:DeleteCluster`
- `eks:UpdateClusterConfig` — `$.deletionProtection` = `false`

**Implements:** [Mitigation.EKS.Cluster.14](#mitigationekscluster14)

**Scoping** (for [Mitigation.EKS.Cluster.14](#mitigationekscluster14)): This implementation guidance recommends recording `UpdateClusterConfig` calls that set `$.deletionProtection`, the deletion protection setting, to false on your cluster, and recording the `DeleteCluster` calls that follow. Because deletion protection must be disabled before a cluster can be deleted, those update calls give you an early signal to investigate.

### Implementation.EKS.Cluster.38

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.deletionProtection`

**Implements:** [Mitigation.EKS.Cluster.14](#mitigationekscluster14)

**Scoping** (for [Mitigation.EKS.Cluster.14](#mitigationekscluster14)): This implementation guidance recommends reading `$.cluster.deletionProtection`, the deletion protection setting, on each cluster, by calling `DescribeCluster` for each cluster name that `ListClusters` returns. Flagging any cluster where that setting is disabled helps you find those that `DeleteCluster` would accept.

### Implementation.EKS.Cluster.39

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreateCluster` | `*` | `eks:deletionProtection` |
| `eks:DeleteCluster`, `eks:DeregisterCluster` | `arn:aws:eks:*:*:cluster/*` | `aws:PrincipalArn` (alternatives: `aws:PrincipalTag/${TagKey}`) |
| `eks:UpdateClusterConfig` | `arn:aws:eks:*:*:cluster/*` | `aws:PrincipalArn` (alternatives: `aws:PrincipalTag/${TagKey}`), `eks:deletionProtection` |

**Implements:** [Mitigation.EKS.Cluster.14](#mitigationekscluster14)

**Scoping** (for [Mitigation.EKS.Cluster.14](#mitigationekscluster14)): This implementation guidance recommends applying the `eks:deletionProtection` and `aws:PrincipalArn` condition keys in a service control policy (SCP) to govern `eks:CreateCluster`, `eks:UpdateClusterConfig`, `eks:DeleteCluster`, and `eks:DeregisterCluster`. These conditions help prevent cluster creation unless `$.deletionProtection`, the cluster's deletion protection setting, is enabled. They also restrict disabling that setting, and deleting or deregistering a cluster, to your trusted principals.

**Reference:** To restrict access to select principals, use the `aws:PrincipalArn` or `aws:PrincipalTag/tag-key` condition keys. Use `aws:PrincipalArn` to specify the Amazon Resource Names (ARNs) of the principals. Those ARNs can follow a naming convention or an IAM path that scopes the restriction to a set of principals without enumerating each one. Use `aws:PrincipalTag/tag-key` to identify those principals by a tag attached to them. Choose the condition key that matches your IAM policy management strategy.

### Implementation.EKS.EksAnywhereSubscription.1

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListEksAnywhereSubscriptions`; check `$.subscriptions[*].arn`, `$.subscriptions[*].tags{key}`

**Implements:** [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1)

**Scoping** (for [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1)): This implementation guidance recommends reading `$.subscriptions[*].tags{key}` on each Amazon EKS Anywhere subscription listed through `$.subscriptions[*].arn` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon EKS Anywhere subscription to an accountable owner and the business function it supports.

### Implementation.EKS.EksAnywhereSubscription.2

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreateEksAnywhereSubscription` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:eks-anywhere-subscription/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:eks-anywhere-subscription/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1)

**Scoping** (for [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1)): This implementation guidance recommends restricting `eks:CreateEksAnywhereSubscription` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS Anywhere subscription without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EKS.FargateProfile.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.EKS.FargateProfile.1](#mitigationeksfargateprofile1)

**Scoping** (for [Mitigation.EKS.FargateProfile.1](#mitigationeksfargateprofile1)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon EKS cluster's fargate profile's Pod execution role credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Data perimeter** (for [Mitigation.EKS.FargateProfile.1](#mitigationeksfargateprofile1)): The default network perimeter service control policy (SCP) samples apply their restriction only to IAM principals tagged with `dp:include:network` set to `true`. Tag your Pod execution role with this tag to apply the network perimeter to its calls. If that role assumes another role during its operations, restrict the assumed role to expected networks as well.
- [Network perimeter SCP sample using aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)
- [Network perimeter SCP sample using aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [Network perimeter guidance for the SCP samples](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/README.md#sidenforcenetworkperimetervpceorgid)

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalTag/tag-key` to apply the restriction to principals whose credentials can be used outside your expected networks, tagged with the selected key and value. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)

### Implementation.EKS.FargateProfile.2

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::FargateProfile`: `$.PodExecutionRoleArn`

**Implements:** [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)

**Scoping** (for [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)): This implementation guidance recommends validating the trust policy of `$.PodExecutionRoleArn`, your Fargate profile's Pod execution role, including source conditions on service principals, before provisioning. This helps prevent the Fargate profile from being provisioned when that trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.

The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.FargateProfile.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListFargateProfiles` with `$.clusterName`; check `$.fargateProfileNames[*]`
3. Call `eks:DescribeFargateProfile` with `$.clusterName`, `$.fargateProfileName`; check `$.fargateProfile.podExecutionRoleArn`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
7. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)

**Scoping** (for [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)): This implementation guidance recommends reading `$.fargateProfile.podExecutionRoleArn`, your Fargate profile's Pod execution role, then evaluating `$.Role.AssumeRolePolicyDocument`, that role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.FargateProfile.4

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `eks-fargate-pods.amazonaws.com` | `sts:AssumeRole` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)

**Scoping** (for [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)): This implementation guidance recommends a trust policy that restricts assumption of your Fargate profile's Pod execution role, `$.podExecutionRoleArn`, to the `eks-fargate-pods.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon EKS from assuming the role on behalf of Fargate profiles outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.EKS.FargateProfile.5

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)

**Scoping** (for [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Fargate profile's Pod execution role, `$.podExecutionRoleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.EKS.FargateProfile.6

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::FargateProfile`: `$.Subnets[*]`

**Implements:** [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3)

**Scoping** (for [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3)): This implementation guidance recommends validating `$.Subnets[*]`, the subnets your Fargate profile's Pods launch into, by resolving each subnet's route table for a default route to an internet gateway before provisioning. Subnets carrying that route fail validation, which helps prevent an Amazon EKS cluster's Fargate profile from being provisioned in a public subnet.

### Implementation.EKS.FargateProfile.7

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListFargateProfiles` with `$.clusterName`; check `$.fargateProfileNames[*]`
3. Call `eks:DescribeFargateProfile` with `$.clusterName`, `$.fargateProfileName`; check `$.fargateProfile.subnets[*]`
4. Call `ec2:DescribeSubnets` with `$.SubnetId[*]`; check `$.subnetSet[*].vpcId`
5. Call `ec2:DescribeRouteTables` with `$.Filter[*].Name`, `$.Filter[*].Value[*]`; check `$.routeTableSet[*].routeSet[*].gatewayId`, `$.routeTableSet[*].routeSet[*].destinationCidrBlock` = `0.0.0.0/0`, `$.routeTableSet[*].routeSet[*].destinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3)

**Scoping** (for [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3)): This implementation guidance recommends reading `$.fargateProfile.subnets[*]`, the subnets where the profile's Pods launch, then resolving each subnet's route table. Flagging a Fargate profile with a subnet whose default route targets an internet gateway, read at `$.routeTableSet[*].routeSet[*].gatewayId`, helps you detect placement in a public subnet.

### Implementation.EKS.FargateProfile.8

*Guidance — Directive*

Create each Fargate profile with two or more private subnets that are in different Availability Zones. Supply these subnet IDs in the `subnets` parameter when you call `CreateFargateProfile`, or in the `Subnets` property of `AWS::EKS::FargateProfile`. Each Pod that runs on the profile has its own elastic network interface in one of these subnets. Spreading the subnets across zones places your Pods' network interfaces in more than one Availability Zone. This helps your workload's network connectivity survive the loss of a single Availability Zone.

Each subnet resides entirely within one Availability Zone. Zone placement therefore comes from the subnets you choose, not from the profile. Before you supply subnet IDs, call `DescribeSubnets` and confirm that their `AvailabilityZone` values differ. Standardize on infrastructure-as-code review for Fargate profiles. Confirm that each template and module that creates a profile supplies at least two subnets from different zones. Fargate profiles are immutable, so you can't add subnets to an existing profile. To move an existing profile to multiple zones, create a replacement profile with matching selectors and the new subnets. Delete the original after the replacement finishes creating. Deleting the original stops its running Pods. Pods that match the replacement can then be rescheduled onto it.

Amazon EKS and Fargate spread Pods across the profile's subnets, but the spread can be uneven. Fargate doesn't currently support Kubernetes topology spread constraints. If a workload needs an even spread, such as two replicas that must avoid downtime, you can use two Fargate profiles. Give each of those profiles one subnet in a different Availability Zone.

To verify the deployed state, call `DescribeFargateProfile` and confirm that the returned subnets resolve to more than one Availability Zone. You can also call `DescribeNetworkInterfaces` to confirm that the network interfaces of your running Pods span more than one Availability Zone. To assess and track multi-AZ posture at scale, use AWS Resilience Hub, which supports Amazon EKS on AWS Fargate. Include your Amazon EKS cluster and the workloads that run on the profile in a Resilience Hub application, then run a resiliency assessment. The assessment can flag workloads concentrated in a single Availability Zone against your resiliency policy.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4)

**Scoping** (for [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4)): This implementation guidance recommends supplying two or more subnets from different Availability Zones in `$.subnets[*]`, the private subnets your Fargate profile's Pods launch into. Spreading the network interfaces Amazon EKS creates for your Fargate Pods across zones helps Pod connectivity survive the loss of a single Availability Zone.

### Implementation.EKS.FargateProfile.9

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::FargateProfile`: `$.Subnets[*]`

**Implements:** [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4)

**Scoping** (for [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4)): This implementation guidance recommends validating, before provisioning, that `$.Subnets[*]`, your Fargate profile's VPC subnets, lists two or more subnet entries. That check helps prevent a single-subnet Fargate profile from being provisioned, so its Pods' network interfaces can span more than one Availability Zone.

### Implementation.EKS.FargateProfile.10

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::FargateProfile`: `$.configuration.Subnets[*]`

**Implements:** [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4)

**Scoping** (for [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4)): This implementation guidance recommends reading `$.configuration.Subnets[*]`, the subnets into which your Fargate profile's Pods are launched, then resolving each subnet's Availability Zone. Flagging a Fargate profile confined to a single Availability Zone helps you detect the elastic network interfaces of its Pods that lack multi-zone redundancy.

### Implementation.EKS.FargateProfile.11

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::FargateProfile`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5)

**Scoping** (for [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon EKS Fargate profile that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.EKS.FargateProfile.12

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::FargateProfile`: `$.tags[*].key`

**Implements:** [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5)

**Scoping** (for [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5)): This implementation guidance recommends reading `$.tags[*].key` on each Amazon EKS Fargate profile and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon EKS Fargate profile to an accountable owner and the business function it supports.

### Implementation.EKS.FargateProfile.13

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreateFargateProfile` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:fargateprofile/*/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:fargateprofile/*/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5)

**Scoping** (for [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5)): This implementation guidance recommends restricting `eks:CreateFargateProfile` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS Fargate profile without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EKS.IdentityProviderConfig.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::IdentityProviderConfig`: `$.Oidc.IssuerUrl`, `$.Oidc.ClientId`

**Implements:** [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1)

**Scoping** (for [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1)): This implementation guidance recommends validating `$.Oidc.IssuerUrl` and `$.Oidc.ClientId`, the OpenID Connect (OIDC) identity provider and audience that your Amazon EKS cluster's Kubernetes API server trusts for user authentication, before provisioning. Requiring trusted OIDC identity providers helps prevent an identity provider config that designates an untrusted provider from being provisioned.

### Implementation.EKS.IdentityProviderConfig.2

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::IdentityProviderConfig`: `$.configuration.Oidc.IssuerUrl`, `$.configuration.Oidc.ClientId`

**Implements:** [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1)

**Scoping** (for [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1)): This implementation guidance recommends reading `$.configuration.Oidc.IssuerUrl` and `$.configuration.Oidc.ClientId`, the OpenID Connect (OIDC) identity provider and client ID that your cluster trusts to authenticate users. Flagging an Amazon EKS identity provider configuration whose recorded provider you do not trust helps you detect an authentication path that could allow unintended access.

### Implementation.EKS.IdentityProviderConfig.3

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::IdentityProviderConfig`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2)

**Scoping** (for [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon EKS identity provider configuration that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.EKS.IdentityProviderConfig.4

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::IdentityProviderConfig`: `$.tags[*].key`

**Implements:** [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2)

**Scoping** (for [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2)): This implementation guidance recommends reading `$.tags[*].key` on each Amazon EKS identity provider config and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon EKS identity provider config to an accountable owner and the business function it supports.

### Implementation.EKS.IdentityProviderConfig.5

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:AssociateIdentityProviderConfig` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:identityproviderconfig/*/*/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:identityproviderconfig/*/*/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2)

**Scoping** (for [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2)): This implementation guidance recommends restricting `eks:AssociateIdentityProviderConfig` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS identity provider configuration without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EKS.Nodegroup.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |
| `*` | `*` | `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService`, `ec2:SourceInstanceARN` (alternatives: `aws:PrincipalTag/dp:include:network`) |

**Implements:** [Mitigation.EKS.Nodegroup.1](#mitigationeksnodegroup1)

**Scoping** (for [Mitigation.EKS.Nodegroup.1](#mitigationeksnodegroup1)): This implementation guidance recommends applying the `ec2:SourceInstanceARN`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon EKS cluster's nodegroup's node IAM role credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Data perimeter** (for [Mitigation.EKS.Nodegroup.1](#mitigationeksnodegroup1)): The default network perimeter service control policy (SCP) samples restrict calls to expected networks for IAM principals with the `dp:include:network` tag set to `true`. This implementation guidance recommends `ec2:SourceInstanceARN` instead, which applies the restriction to role sessions created by an Amazon EC2 instance, including your node IAM role. It records `aws:PrincipalTag/dp:include:network` as an alternative when you prefer to restrict only the roles you tag. If your node IAM role assumes another role as part of its operations, consider applying the same network perimeter restriction to the assumed role.
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [Network perimeter guidance for service control policies](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/README.md#sidenforcenetworkperimetervpceorgid)

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `ec2:SourceInstanceARN` to apply the restriction to the role sessions whose credentials can be used outside your expected networks. Alternatively, you can use `aws:PrincipalTag/tag-key`, tagging the roles with the tag key and value your IAM policy management strategy uses. For credentials delivered to an Amazon EC2 instance, you can use `aws:Ec2InstanceSourceVpc` and `aws:Ec2InstanceSourcePrivateIPv4` to bind them to the instance they were issued to. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [How to use policies to restrict where EC2 instance credentials can be used from](https://aws.amazon.com/blogs/security/how-to-use-policies-to-restrict-where-ec2-instance-credentials-can-be-used-from/)
- [SCP sample: network perimeter policy for Amazon EC2](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/network_perimeter_ec2_scp.json)

### Implementation.EKS.Nodegroup.2

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Nodegroup`: `$.NodeRole`

**Implements:** [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)

**Scoping** (for [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)): This implementation guidance recommends validating the trust policy of `$.NodeRole`, your nodegroup's node IAM role, before provisioning. This helps prevent the nodegroup from being provisioned when that trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.Nodegroup.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListNodegroups` with `$.clusterName`; check `$.nodegroups[*]`
3. Call `eks:DescribeNodegroup` with `$.clusterName`, `$.nodegroupName`; check `$.nodegroup.nodeRole`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)

**Scoping** (for [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)): This implementation guidance recommends reading `$.nodegroup.nodeRole`, your nodegroup's node IAM role, then evaluating `$.Role.AssumeRolePolicyDocument`, that role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.Nodegroup.4

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ec2.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)

**Scoping** (for [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon EKS cluster's nodegroup's node IAM role, `$.nodeRole`, to the `ec2.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.EKS.Nodegroup.5

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)

**Scoping** (for [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon EKS cluster's nodegroup's node IAM role, `$.nodeRole`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.EKS.Nodegroup.6

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Nodegroup`: `$.Subnets[*]`

**Implements:** [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3)

**Scoping** (for [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3)): This implementation guidance recommends validating `$.Subnets[*]`, the subnets your node group's nodes launch into, by resolving each subnet's route table for a default route to an internet gateway before provisioning. Subnets carrying that route fail validation, which helps prevent an Amazon EKS managed node group from being provisioned in a public subnet.

### Implementation.EKS.Nodegroup.7

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListNodegroups` with `$.clusterName`; check `$.nodegroups[*]`
3. Call `eks:DescribeNodegroup` with `$.clusterName`, `$.nodegroupName`; check `$.nodegroup.subnets[*]`
4. Call `ec2:DescribeSubnets` with `$.SubnetId`; check `$.subnetSet[*].vpcId`
5. Call `ec2:DescribeRouteTables` with `$.Filter` = `association.subnet-id`; check `$.routeTableSet[*].routeSet[*].gatewayId`, `$.routeTableSet[*].routeSet[*].destinationCidrBlock`, `$.routeTableSet[*].routeSet[*].destinationIpv6CidrBlock`, `$.routeTableSet[*].associationSet[*].subnetId`
6. Call `ec2:DescribeRouteTables` with `$.Filter` = `association.main`, `$.Filter` = `vpc-id`; check `$.routeTableSet[*].associationSet[*].main`, `$.routeTableSet[*].routeSet[*].gatewayId`, `$.routeTableSet[*].routeSet[*].destinationCidrBlock` = `0.0.0.0/0`, `$.routeTableSet[*].routeSet[*].destinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3)

**Scoping** (for [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3)): This implementation guidance recommends reading `$.nodegroup.subnets[*]`, the subnets your managed nodes launch into, then resolving each subnet's route table. Flagging a managed node group with a subnet whose default route targets an internet gateway, read at `$.routeTableSet[*].routeSet[*].gatewayId`, helps you detect placement in a public subnet.

### Implementation.EKS.Nodegroup.8

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Nodegroup`: `$.InstanceTypes[*]`, `$.LaunchTemplate.Id`, `$.LaunchTemplate.Name`, `$.LaunchTemplate.Version`

**Implements:** [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4)

**Scoping** (for [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4)): This implementation guidance recommends validating `$.InstanceTypes[*]` and `$.launchTemplateVersionSet[*].launchTemplateData.instanceType`, the instance types set on your node group or in its launch template, before provisioning, which helps prevent a managed node group configured with instance types without automatic instance-to-instance encryption from being provisioned. That helps provide an additional layer of encryption in transit between your nodes without application changes.

### Implementation.EKS.Nodegroup.9

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::Nodegroup`: `$.configuration.InstanceTypes[*]`, `$.configuration.LaunchTemplate.Id`, `$.configuration.LaunchTemplate.Name`, `$.configuration.LaunchTemplate.Version`

**Implements:** [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4)

**Scoping** (for [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4)): This implementation guidance recommends comparing the instance types of your Amazon EKS nodegroup at `$.configuration.InstanceTypes[*]` and `$.launchTemplateVersionSet[*].launchTemplateData.instanceType` with EC2 encryption-in-transit support at `$.instanceTypeSet[*].networkInfo.encryptionInTransitSupported`. That comparison flags an Amazon EKS nodegroup configured with instance types without automatic encryption of instance-to-instance traffic.

### Implementation.EKS.Nodegroup.10

*Guidance — Directive*

Create each Amazon EKS managed node group with two or more subnets that belong to different Availability Zones. Supply these subnets in the `subnets` parameter of `CreateNodegroup`, not in a launch template. Node group deployment fails if your launch template specifies a subnet. The node group's Auto Scaling group spans every subnet that you specify. Amazon EKS can then launch your nodes, and the network interfaces attached to them, in more than one Availability Zone. This helps your node group's network connectivity survive the loss of a single zone. You can set the subnet list only when you create the node group. `UpdateNodegroupConfig` can't change it. To move an existing single-AZ node group to multiple zones, create a replacement node group with multi-AZ subnets. Then drain the existing nodes and delete the old node group.

Subnet-to-Availability Zone membership is a property of each subnet, not of the node group. Before you supply subnet IDs, confirm that they belong to distinct Availability Zones. To do this, call the Amazon EC2 `DescribeSubnets` operation and check the `AvailabilityZone` field in the response. Standardize on infrastructure-as-code review for the templates and modules that create your node groups. Confirm that each one supplies at least two subnets from different Availability Zones.

To verify the deployed state, call `DescribeNodegroup`. It returns the subnets specified for the node group's Auto Scaling group and the name of that Auto Scaling group. Then call the Amazon EC2 `DescribeNetworkInterfaces` operation for the instances in that Auto Scaling group. Confirm that the subnets of the attached network interfaces span more than one Availability Zone.

To assess and track multi-AZ posture at scale, use AWS Resilience Hub. Add your Amazon EKS cluster and the namespaces whose workloads run on your node groups to a Resilience Hub application. Then run a resiliency assessment. The assessment reports whether your application meets the Availability Zone disruption targets in your resiliency policy. This helps you find single-AZ concentration in the application.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5)

**Scoping** (for [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5)): This implementation guidance recommends supplying two or more subnets from different Availability Zones in `$.subnets[*]`, the subnets for your node group's Auto Scaling group. Spreading the network interfaces of the nodes Amazon EKS launches across zones helps your node group's connectivity survive the loss of a single Availability Zone.

### Implementation.EKS.Nodegroup.11

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Nodegroup`: `$.Subnets[*]`

**Implements:** [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5)

**Scoping** (for [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5)): This implementation guidance recommends validating, before provisioning, that `$.Subnets[*]`, your managed node group's VPC subnets, lists two or more subnet entries. That check helps prevent a single-subnet managed node group from being provisioned, so its nodes' network interfaces can span more than one Availability Zone.

### Implementation.EKS.Nodegroup.12

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::Nodegroup`: `$.configuration.Subnets[*]`

**Implements:** [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5)

**Scoping** (for [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5)): This implementation guidance recommends reading `$.configuration.Subnets[*]`, the subnets where your managed node group launches its nodes, then resolving each subnet's Availability Zone. Flagging a managed node group confined to a single Availability Zone helps you detect the network interfaces of its nodes that lack multi-zone redundancy.

### Implementation.EKS.Nodegroup.13

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Nodegroup`: `$.Tags{key}`

**Implements:** [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6)

**Scoping** (for [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6)): This implementation guidance recommends validating, before provisioning, that `$.Tags{key}` declares owner and purpose tags. Requiring those tags helps prevent an Amazon EKS managed node group that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.EKS.Nodegroup.14

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::Nodegroup`: `$.configuration.Tags{key}`

**Implements:** [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6)

**Scoping** (for [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6)): This implementation guidance recommends reading `$.configuration.Tags{key}` on each Amazon EKS managed node group and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon EKS managed node group to an accountable owner and the business function it supports.

### Implementation.EKS.Nodegroup.15

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreateNodegroup` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:nodegroup/*/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:nodegroup/*/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6)

**Scoping** (for [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6)): This implementation guidance recommends restricting `eks:CreateNodegroup` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS managed node group without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.EKS.Nodegroup.16

*Guidance — Directive*

Configure the security groups on your managed node group's instances following least-privilege best practices. Restrict both inbound and outbound rules to the required sources and destinations (IP address ranges, referenced security groups, or prefix lists), protocols, and ports. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change.

By default, Amazon EKS applies the cluster security group to your node group's instances. Changes to the cluster security group also affect your cluster's network interfaces and other managed node groups that use it. If your launch template specifies custom security groups, Amazon EKS doesn't add the cluster security group. To change which custom security groups your nodes use, specify them in a new launch template version. Then apply that version by calling `UpdateNodegroupVersion` with the new version in `$.launchTemplate.version`. Keep the minimum rules that your nodes need to communicate with your cluster. Nodes with incorrect security group rules can't join the cluster. If you enable remote access, specify source security groups in `$.remoteAccess.sourceSecurityGroups` when you call `CreateNodegroup`. If you specify an SSH key without source security groups, port 22 (port 3389 for Windows) on your nodes is opened to 0.0.0.0/0.

To govern security groups at scale, use AWS Firewall Manager security group policies across your organization in AWS Organizations. You can scope each policy by account, organizational unit (OU), or tag. A common security group policy applies a baseline security group to your accounts and resources. A content audit security group policy audits inbound and outbound rules against an allow list or a block list. It flags noncompliant security groups in a central dashboard and sends notifications. It can also update or remove noncompliant rules automatically. Review the findings before you turn on automatic remediation. A usage audit security group policy finds unused or redundant security groups and can clean them up.

To verify unintended access that your security groups permit, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended. For example, match any path that reaches administrative ports 22 or 3389 on your nodes from an unrestricted address. Create the scope by calling `CreateNetworkInsightsAccessScope`. Then start an analysis by calling `StartNetworkInsightsAccessScopeAnalysis` and review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)

**Scoping** (for [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)): This implementation guidance recommends restricting the inbound and outbound rules on your nodegroup's security groups to the required addresses, protocols, and ports. Excluding unrestricted addresses such as 0.0.0.0/0 or ::/0 from those inbound and outbound rules helps reduce the network exposure of your nodegroup.

### Implementation.EKS.Nodegroup.17

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::Nodegroup`: `$.LaunchTemplate.Id`, `$.LaunchTemplate.Name`, `$.LaunchTemplate.Version`, `$.ClusterName`, `$.RemoteAccess.SourceSecurityGroups[*]`, `$.RemoteAccess.Ec2SshKey`

**Implements:** [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)

**Scoping** (for [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)): This implementation guidance recommends validating the inbound and outbound rules of the security groups resolved from its launch template, its cluster, or `$.RemoteAccess.SourceSecurityGroups[*]`, and that `$.RemoteAccess.Ec2SshKey`, the SSH key for remote access to the nodes, is declared only alongside them, before provisioning. Checking for rules that allow any address on unintended ports, directly or through a prefix list, helps prevent an Amazon EKS managed node group from being provisioned with such groups.

### Implementation.EKS.Nodegroup.18

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListNodegroups` with `$.clusterName`; check `$.nodegroups[*]`
3. Call `eks:DescribeNodegroup` with `$.clusterName`, `$.nodegroupName`; check `$.nodegroup.resources.remoteAccessSecurityGroup`, `$.nodegroup.launchTemplate.id`, `$.nodegroup.launchTemplate.version`
4. Call `ec2:DescribeLaunchTemplateVersions` with `$.LaunchTemplateId`, `$.LaunchTemplateVersion[*]`; check `$.launchTemplateVersionSet[*].launchTemplateData.securityGroupIdSet[*]`, `$.launchTemplateVersionSet[*].launchTemplateData.networkInterfaceSet[*].groupSet[*]`
5. Call `eks:DescribeCluster` with `$.name`; check `$.cluster.resourcesVpcConfig.clusterSecurityGroupId`
6. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
7. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)

**Scoping** (for [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7)): This implementation guidance recommends resolving the security groups recorded at `$.nodegroup.resources.remoteAccessSecurityGroup`, named in the launch template at `$.launchTemplateVersionSet[*].launchTemplateData.securityGroupIdSet[*]` or behind its interface at `$.launchTemplateVersionSet[*].launchTemplateData.networkInterfaceSet[*].groupSet[*]`, or in effect through the cluster at `$.cluster.resourcesVpcConfig.clusterSecurityGroupId` and inspecting their inbound and outbound rules. Flagging rules that admit 0.0.0.0/0 or ::/0 on sensitive ports, broad port ranges, or all protocols helps you detect your nodegroups with broader network exposure than they need.

### Implementation.EKS.PodIdentityAssociation.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1)

**Scoping** (for [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon EKS cluster's pod identity association's target IAM role or, where no target IAM role is configured, EKS Pod Identity role credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Data perimeter** (for [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1)): The default network perimeter service control policy (SCP) samples apply to IAM roles that you tag with `dp:include:network` set to `true`. Tag the target IAM role when the association specifies one, because its credentials are the ones delivered to your Pods; otherwise tag the EKS Pod Identity role.
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [Network perimeter guidance for the data perimeter SCP examples](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/README.md#sidenforcenetworkperimetervpceorgid)

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalTag/tag-key` to apply the restriction to principals whose credentials can be used outside your expected networks, tagged with the selected key and value. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)

### Implementation.EKS.PodIdentityAssociation.2

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::PodIdentityAssociation`: `$.RoleArn`

**Implements:** [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)

**Scoping** (for [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)): This implementation guidance recommends validating the trust policy of `$.RoleArn`, your pod identity association's EKS Pod Identity role, including source conditions on service principals, before provisioning. This helps prevent the pod identity association from being provisioned when that trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.PodIdentityAssociation.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `eks:ListClusters`; check `$.clusters[*]`
2. Call `eks:ListPodIdentityAssociations` with `$.clusterName`; check `$.associations[*].associationId`
3. Call `eks:DescribePodIdentityAssociation` with `$.clusterName`, `$.associationId`; check `$.association.roleArn`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
7. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)

**Scoping** (for [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)): This implementation guidance recommends reading `$.association.roleArn`, your pod identity association's EKS Pod Identity role, then evaluating `$.Role.AssumeRolePolicyDocument`, that role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.EKS.PodIdentityAssociation.4

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `pods.eks.amazonaws.com` | `sts:AssumeRole`, `sts:TagSession` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)

**Scoping** (for [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)): This implementation guidance recommends a trust policy that restricts assumption of your Pod Identity association's EKS Pod Identity role, `$.roleArn`, to the `pods.eks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon EKS from assuming the role on behalf of clusters outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.EKS.PodIdentityAssociation.5

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)

**Scoping** (for [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your pod identity association's EKS Pod Identity role, `$.roleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.EKS.PodIdentityAssociation.6

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::EKS::PodIdentityAssociation`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3)

**Scoping** (for [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon EKS cluster's pod identity association that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.EKS.PodIdentityAssociation.7

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::EKS::PodIdentityAssociation`: `$.configuration.Tags[*].Key`, `$.configuration.Tags[*].Value`

**Implements:** [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3)

**Scoping** (for [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3)): This implementation guidance recommends reading `$.configuration.Tags[*].Key` on each EKS Pod Identity association and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each EKS Pod Identity association to an accountable owner and the business function it supports.

### Implementation.EKS.PodIdentityAssociation.8

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `eks:CreatePodIdentityAssociation` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:TagResource` | `arn:aws:eks:*:*:podidentityassociation/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `eks:UntagResource` | `arn:aws:eks:*:*:podidentityassociation/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3)

**Scoping** (for [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3)): This implementation guidance recommends restricting `eks:CreatePodIdentityAssociation` and `eks:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon EKS Pod Identity association without the owner and purpose tag keys, or removing them through `eks:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### ManagedControl.EKS.Addon.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon EKS add-ons have tags

**Aliases:** `CONFIG.EKS.DT.5`

**Identifier:** `EKS_ADDON_TAGGED`

**ARN:** `arn:aws:controlcatalog:::control/bgcqq4ll1ct4a2nzld2xahn8c`

Checks if Amazon EKS add-ons have tags. Optionally, you can specify tag keys. The rule is NON_COMPLIANT if there are no tags or if the specified tag keys are not present. The rule does not check for tags starting with 'aws:'.

**Implements:** [Mitigation.EKS.Addon.5](#mitigationeksaddon5)

### ManagedControl.EKS.Cluster.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if an Amazon Elastic Kubernetes Service (Amazon EKS) cluster is configured with logging enabled

**Aliases:** `CONFIG.EKS.DT.1`

**Identifier:** `EKS_CLUSTER_LOG_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/8i4rci08epasf0b9h94gj1zt8`

Checks if an Amazon Elastic Kubernetes Service (Amazon EKS) cluster is configured with logging enabled. The rule is NON_COMPLIANT if logging for Amazon EKS clusters is not enabled or if logging is not enabled with the log type mentioned.

**Implements:** [Mitigation.EKS.Cluster.1](#mitigationekscluster1)

### ManagedControl.EKS.Cluster.2

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if an Amazon Elastic Kubernetes Service (Amazon EKS) cluster is configured with logging enabled

**Aliases:** `CONFIG.EKS.DT.4`

**Identifier:** `EKS_CLUSTER_LOGGING_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/n761ztipe2x0pp5mgaid4dnn`

Checks if an Amazon Elastic Kubernetes Service (Amazon EKS) cluster is configured with logging enabled. The rule is NON_COMPLIANT if logging for Amazon EKS clusters is not enabled for all log types.

**Implements:** [Mitigation.EKS.Cluster.1](#mitigationekscluster1)

### ManagedControl.EKS.Cluster.3

*AWS Config rule — Detective*

AWS managed control from General controls (service-agnostic — no governed resource).

**Title:** Checks if a recovery point expires no earlier than after the specified period

**Aliases:** `BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK`

**Identifier:** `BACKUP_RECOVERY_POINT_MINIMUM_RETENTION_CHECK`

**ARN:** `arn:aws:controlcatalog:::control/m7a5gbdf08wg2o0en010mkng`

Checks if a recovery point expires no earlier than after the specified period. The rule is NON_COMPLIANT if the recovery point has a retention point that is less than the required retention period.

**Implements:** [Mitigation.EKS.Cluster.8](#mitigationekscluster8)

### ManagedControl.EKS.Cluster.4

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require an Amazon EKS cluster to be configured with secret encryption using AWS Key Management Service (KMS) keys

**Aliases:** `CT.EKS.PR.2`

**ARN:** `arn:aws:controlcatalog:::control/1xg698zqw921vpmp4o9gn191n`

This control checks whether Amazon Elastic Kubernetes Service (Amazon EKS) clusters are configured to use Kubernetes secrets encrypted with AWS Key Management Service (KMS) keys.

**Implements:** [Mitigation.EKS.Cluster.9](#mitigationekscluster9)

### ManagedControl.EKS.Cluster.5

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon EKS clusters are configured to have Kubernetes secrets encrypted using AWS KMS

**Aliases:** `CONFIG.EKS.DT.2`

**Identifier:** `EKS_CLUSTER_SECRETS_ENCRYPTED`

**ARN:** `arn:aws:controlcatalog:::control/acauoj4kp0ksvufhr5lnaeolk`

Checks if Amazon EKS clusters are configured to have Kubernetes secrets encrypted using AWS KMS. The rule is NON_COMPLIANT if an EKS cluster does not have an encryptionConfig resource or if encryptionConfig does not name secrets as a resource.

**Implements:** [Mitigation.EKS.Cluster.9](#mitigationekscluster9)

### ManagedControl.EKS.FargateProfile.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon EKS fargate profiles have tags

**Aliases:** `CONFIG.EKS.DT.7`

**Identifier:** `EKS_FARGATE_PROFILE_TAGGED`

**ARN:** `arn:aws:controlcatalog:::control/251z9lrx66pg5v89ncv5hhy57`

Checks if Amazon EKS fargate profiles have tags. Optionally, you can specify tag keys. The rule is NON_COMPLIANT if there are no tags or if the specified tag keys are not present. The rule does not check for tags starting with 'aws:'.

**Implements:** [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5)

---

## Appendix

### Resources

#### AccessEntry

*ACCESS_CONTROL*

An Amazon EKS access entry grants an AWS IAM user or role access to the Kubernetes API of a specific cluster. It links that IAM identity to a Kubernetes username and optional group memberships. IAM authentication then carries through to authorization inside the cluster.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:access-entry/${ClusterName}/${IamIdentityType}/${IamIdentityAccountID}/${IamIdentityName}/${UUID}`
- CloudFormation: `AWS::EKS::AccessEntry`
- 11 operations, 1 threats, 1 mitigations

#### Addon

*COMPUTE_RESOURCE*

An Amazon EKS add-on is a managed installation of operational software on an Amazon EKS cluster. It covers supporting components such as networking plugins, cluster DNS, storage drivers, and observability or security agents that applications depend on.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:addon/${ClusterName}/${AddonName}/${UUID}`
- CloudFormation: `AWS::EKS::Addon`
- AWS Config: `AWS::EKS::Addon`
- 13 operations, 7 threats, 6 mitigations

#### Capability

*COMPUTE_RESOURCE*

A capability adds a fully managed platform feature to an Amazon EKS cluster. It runs controller software in AWS-managed infrastructure outside the cluster, so worker nodes do not host or scale these components. AWS manages patching, version upgrades, scaling, and availability for the controllers.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:capability/${ClusterName}/${CapabilityType}/${CapabilityName}/${UUID}`
- CloudFormation: `AWS::EKS::Capability`
- 11 operations, 6 threats, 4 mitigations

#### CertificateAuthority

*CRYPTOGRAPHIC_MATERIAL*

An Amazon EKS certificate authority is the root of trust for a cluster's Kubernetes control plane. It signs the certificates that secure API server connections, so worker nodes and clients can verify the API server's identity.

- CloudFormation: `AWS::EKS::CertificateAuthority`
- 5 operations, 0 threats, 0 mitigations

#### Cluster

*COMPUTE_RESOURCE*

An Amazon EKS cluster is a managed Kubernetes environment for deploying, scaling, and operating containerized applications. AWS operates a dedicated, highly available control plane for each cluster across multiple Availability Zones. The control plane includes the Kubernetes API server and its backing data store.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:cluster/${ClusterName}`
- CloudFormation: `AWS::EKS::Cluster`
- AWS Config: `AWS::EKS::Cluster`
- 20 operations, 24 threats, 14 mitigations

#### EksAnywhereSubscription

*ACCESS_CONTROL*

An EKS Anywhere subscription entitles an AWS account to AWS support for Amazon EKS Anywhere clusters running on customer-managed infrastructure. It also gives the account access to EKS Anywhere Curated Packages, which add capabilities such as load balancing, observability, and autoscaling.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:eks-anywhere-subscription/${UUID}`
- CloudFormation: `AWS::EKS::EksAnywhereSubscription`
- 8 operations, 1 threats, 1 mitigations

#### FargateProfile

*CONFIGURATION*

A Fargate profile tells an Amazon EKS cluster which Kubernetes pods run on AWS Fargate serverless compute instead of on nodes you manage. Administrators define selectors that match pods by namespace and optional labels, including simple wildcard patterns. When a pod being scheduled matches a selector, EKS runs it on Fargate with that profile's settings.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:fargateprofile/${ClusterName}/${FargateProfileName}/${UUID}`
- CloudFormation: `AWS::EKS::FargateProfile`
- AWS Config: `AWS::EKS::FargateProfile`
- 7 operations, 6 threats, 5 mitigations

#### IdentityProviderConfig

*IDENTITY*

An identity provider config links an Amazon EKS cluster to an external OpenID Connect identity provider. Users can then authenticate to the Kubernetes API with tokens from an existing identity system, alongside or instead of IAM identities.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:identityproviderconfig/${ClusterName}/${IdentityProviderType}/${IdentityProviderConfigName}/${UUID}`
- CloudFormation: `AWS::EKS::IdentityProviderConfig`
- AWS Config: `AWS::EKS::IdentityProviderConfig`
- 7 operations, 3 threats, 2 mitigations

#### Nodegroup

*COMPUTE_RESOURCE*

A managed node group provides the Amazon EC2 worker capacity that runs Kubernetes workloads in an Amazon EKS cluster. Amazon EKS provisions the instances through an Auto Scaling group in your account and registers them with the cluster automatically. Each group spans the subnets and Availability Zones you choose.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:nodegroup/${ClusterName}/${NodegroupName}/${UUID}`
- CloudFormation: `AWS::EKS::Nodegroup`
- AWS Config: `AWS::EKS::Nodegroup`
- 12 operations, 11 threats, 7 mitigations

#### PodIdentityAssociation

*IDENTITY*

A pod identity association links an IAM role to a Kubernetes service account in a specific namespace of an Amazon EKS cluster. Pods that run under that service account receive short-lived AWS credentials for the role, which are rotated automatically. This works much like an instance profile does for an EC2 instance, but at the granularity of individual workloads.

- IAM resource ARN: `arn:${Partition}:eks:${Region}:${Account}:podidentityassociation/${ClusterName}/${UUID}`
- CloudFormation: `AWS::EKS::PodIdentityAssociation`
- AWS Config: `AWS::EKS::PodIdentityAssociation`
- 7 operations, 3 threats, 3 mitigations

### Operations

| Operation | Analyses | Resources | Mitigations |
|---|---|---|---|
| `ActivateCertificateAuthority` | DOCUMENTATION | [CertificateAuthority](#certificateauthority) |  |
| `AssociateAccessPolicy` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) |  |
| `AssociateEncryptionConfig` | DOCUMENTATION | [Cluster](#cluster) | [Mitigation.EKS.Cluster.9](#mitigationekscluster9) |
| `AssociateIdentityProviderConfig` | DOCUMENTATION | [IdentityProviderConfig](#identityproviderconfig) | [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1), [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2) |
| `CancelUpdate` | DOCUMENTATION | [Addon](#addon), [Capability](#capability), [Cluster](#cluster), [Nodegroup](#nodegroup) |  |
| `CreateAccessEntry` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) | [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1), [Mitigation.EKS.Cluster.11](#mitigationekscluster11) |
| `CreateAddon` | DOCUMENTATION | [Addon](#addon) | [Mitigation.EKS.2](#mitigationeks2), [Mitigation.EKS.Addon.1](#mitigationeksaddon1), [Mitigation.EKS.Addon.2](#mitigationeksaddon2), [Mitigation.EKS.Addon.3](#mitigationeksaddon3), [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.5](#mitigationeksaddon5), [Mitigation.EKS.Addon.6](#mitigationeksaddon6) |
| `CreateCapability` | DOCUMENTATION | [Capability](#capability) | [Mitigation.EKS.Capability.1](#mitigationekscapability1), [Mitigation.EKS.Capability.2](#mitigationekscapability2), [Mitigation.EKS.Capability.3](#mitigationekscapability3) |
| `CreateCertificateAuthority` | DOCUMENTATION | [CertificateAuthority](#certificateauthority) |  |
| `CreateCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.EKS.Cluster.1](#mitigationekscluster1), [Mitigation.EKS.Cluster.2](#mitigationekscluster2), [Mitigation.EKS.Cluster.3](#mitigationekscluster3), [Mitigation.EKS.Cluster.4](#mitigationekscluster4), [Mitigation.EKS.Cluster.5](#mitigationekscluster5), [Mitigation.EKS.Cluster.6](#mitigationekscluster6), [Mitigation.EKS.Cluster.9](#mitigationekscluster9), [Mitigation.EKS.Cluster.10](#mitigationekscluster10), [Mitigation.EKS.Cluster.11](#mitigationekscluster11), [Mitigation.EKS.Cluster.12](#mitigationekscluster12), [Mitigation.EKS.Cluster.13](#mitigationekscluster13), [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |
| `CreateEksAnywhereSubscription` | DOCUMENTATION | [EksAnywhereSubscription](#eksanywheresubscription) | [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1) |
| `CreateFargateProfile` | DOCUMENTATION, DATA_PERIMETER | [FargateProfile](#fargateprofile) | [Mitigation.EKS.FargateProfile.1](#mitigationeksfargateprofile1), [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2), [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3), [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4), [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5) |
| `CreateNodegroup` | DOCUMENTATION, DATA_PERIMETER | [Nodegroup](#nodegroup) | [Mitigation.EKS.Nodegroup.1](#mitigationeksnodegroup1), [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2), [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3), [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4), [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5), [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6), [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7) |
| `CreatePodIdentityAssociation` | DOCUMENTATION, DATA_PERIMETER | [PodIdentityAssociation](#podidentityassociation) | [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1), [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2), [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3) |
| `DeleteAccessEntry` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) | [Mitigation.EKS.Cluster.11](#mitigationekscluster11) |
| `DeleteAddon` | DOCUMENTATION, DATA_PERIMETER | [Addon](#addon) | [Mitigation.EKS.Addon.1](#mitigationeksaddon1), [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.6](#mitigationeksaddon6) |
| `DeleteCapability` | DOCUMENTATION | [Capability](#capability) |  |
| `DeleteCertificateAuthority` | DOCUMENTATION | [CertificateAuthority](#certificateauthority) |  |
| `DeleteCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.EKS.Cluster.8](#mitigationekscluster8), [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |
| `DeleteEksAnywhereSubscription` | DOCUMENTATION | [EksAnywhereSubscription](#eksanywheresubscription) |  |
| `DeleteFargateProfile` | DOCUMENTATION, DATA_PERIMETER | [FargateProfile](#fargateprofile) |  |
| `DeleteNodegroup` | DOCUMENTATION, DATA_PERIMETER | [Nodegroup](#nodegroup) |  |
| `DeletePodIdentityAssociation` | DOCUMENTATION, DATA_PERIMETER | [PodIdentityAssociation](#podidentityassociation) |  |
| `DeregisterCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.EKS.Cluster.8](#mitigationekscluster8), [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |
| `DescribeAccessEntry` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) | [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1), [Mitigation.EKS.Cluster.11](#mitigationekscluster11) |
| `DescribeAddon` | DOCUMENTATION, DATA_PERIMETER | [Addon](#addon) | [Mitigation.EKS.Addon.1](#mitigationeksaddon1), [Mitigation.EKS.Addon.2](#mitigationeksaddon2), [Mitigation.EKS.Addon.3](#mitigationeksaddon3), [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.5](#mitigationeksaddon5), [Mitigation.EKS.Addon.6](#mitigationeksaddon6) |
| `DescribeAddonConfiguration` | DOCUMENTATION, DATA_PERIMETER | [Addon](#addon) |  |
| `DescribeAddonVersions` | DOCUMENTATION, DATA_PERIMETER | [Addon](#addon) |  |
| `DescribeCapability` | DOCUMENTATION | [Capability](#capability) | [Mitigation.EKS.Capability.1](#mitigationekscapability1), [Mitigation.EKS.Capability.2](#mitigationekscapability2), [Mitigation.EKS.Capability.3](#mitigationekscapability3) |
| `DescribeCertificateAuthority` | DOCUMENTATION | [CertificateAuthority](#certificateauthority) |  |
| `DescribeCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.EKS.Cluster.1](#mitigationekscluster1), [Mitigation.EKS.Cluster.2](#mitigationekscluster2), [Mitigation.EKS.Cluster.3](#mitigationekscluster3), [Mitigation.EKS.Cluster.4](#mitigationekscluster4), [Mitigation.EKS.Cluster.5](#mitigationekscluster5), [Mitigation.EKS.Cluster.6](#mitigationekscluster6), [Mitigation.EKS.Cluster.9](#mitigationekscluster9), [Mitigation.EKS.Cluster.10](#mitigationekscluster10), [Mitigation.EKS.Cluster.11](#mitigationekscluster11), [Mitigation.EKS.Cluster.12](#mitigationekscluster12), [Mitigation.EKS.Cluster.13](#mitigationekscluster13), [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |
| `DescribeClusterVersions` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `DescribeEksAnywhereSubscription` | DOCUMENTATION, DATA_PERIMETER | [EksAnywhereSubscription](#eksanywheresubscription) | [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1) |
| `DescribeFargateProfile` | DOCUMENTATION, DATA_PERIMETER | [FargateProfile](#fargateprofile) | [Mitigation.EKS.FargateProfile.1](#mitigationeksfargateprofile1), [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2), [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3), [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4), [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5) |
| `DescribeIdentityProviderConfig` | DOCUMENTATION | [IdentityProviderConfig](#identityproviderconfig) | [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1), [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2) |
| `DescribeInsight` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `DescribeInsightsRefresh` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `DescribeNodegroup` | DOCUMENTATION, DATA_PERIMETER | [Nodegroup](#nodegroup) | [Mitigation.EKS.Nodegroup.1](#mitigationeksnodegroup1), [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2), [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3), [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4), [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5), [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6), [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7) |
| `DescribePodIdentityAssociation` | DOCUMENTATION, DATA_PERIMETER | [PodIdentityAssociation](#podidentityassociation) | [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1), [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2), [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3) |
| `DescribeUpdate` | DOCUMENTATION, DATA_PERIMETER | [Addon](#addon), [Capability](#capability), [Cluster](#cluster), [Nodegroup](#nodegroup) | [Mitigation.EKS.Cluster.1](#mitigationekscluster1), [Mitigation.EKS.Cluster.4](#mitigationekscluster4), [Mitigation.EKS.Cluster.9](#mitigationekscluster9), [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |
| `DisassociateAccessPolicy` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) |  |
| `DisassociateIdentityProviderConfig` | DOCUMENTATION | [IdentityProviderConfig](#identityproviderconfig) |  |
| `ListAccessEntries` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) | [Mitigation.EKS.Cluster.10](#mitigationekscluster10), [Mitigation.EKS.Cluster.11](#mitigationekscluster11) |
| `ListAccessPolicies` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) |  |
| `ListAddons` | DOCUMENTATION, DATA_PERIMETER | [Addon](#addon) | [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.6](#mitigationeksaddon6) |
| `ListAssociatedAccessPolicies` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) |  |
| `ListCapabilities` | DOCUMENTATION | [Capability](#capability) | [Mitigation.EKS.Capability.2](#mitigationekscapability2) |
| `ListCertificateAuthorities` | DOCUMENTATION | [CertificateAuthority](#certificateauthority) |  |
| `ListClusters` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `ListEksAnywhereSubscriptions` | DOCUMENTATION, DATA_PERIMETER | [EksAnywhereSubscription](#eksanywheresubscription) | [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1) |
| `ListFargateProfiles` | DOCUMENTATION, DATA_PERIMETER | [FargateProfile](#fargateprofile) | [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4) |
| `ListIdentityProviderConfigs` | DOCUMENTATION, DATA_PERIMETER | [IdentityProviderConfig](#identityproviderconfig) | [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1) |
| `ListInsights` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `ListNodegroups` | DOCUMENTATION, DATA_PERIMETER | [Nodegroup](#nodegroup) |  |
| `ListPodIdentityAssociations` | DOCUMENTATION, DATA_PERIMETER | [PodIdentityAssociation](#podidentityassociation) | [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1), [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2) |
| `ListTagsForResource` | DOCUMENTATION, DATA_PERIMETER | [Addon](#addon), [Capability](#capability), [Cluster](#cluster), [EksAnywhereSubscription](#eksanywheresubscription), [FargateProfile](#fargateprofile), [IdentityProviderConfig](#identityproviderconfig), [Nodegroup](#nodegroup) | [Mitigation.EKS.Addon.5](#mitigationeksaddon5), [Mitigation.EKS.Capability.3](#mitigationekscapability3), [Mitigation.EKS.Cluster.12](#mitigationekscluster12), [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1), [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5), [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2), [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6) |
| `ListUpdates` | DOCUMENTATION, DATA_PERIMETER | [Addon](#addon), [Capability](#capability), [Cluster](#cluster), [Nodegroup](#nodegroup) |  |
| `RegisterCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.EKS.Cluster.2](#mitigationekscluster2), [Mitigation.EKS.Cluster.3](#mitigationekscluster3), [Mitigation.EKS.Cluster.12](#mitigationekscluster12) |
| `StartInsightsRefresh` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `TagResource` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry), [Addon](#addon), [Capability](#capability), [Cluster](#cluster), [EksAnywhereSubscription](#eksanywheresubscription), [FargateProfile](#fargateprofile), [IdentityProviderConfig](#identityproviderconfig), [Nodegroup](#nodegroup), [PodIdentityAssociation](#podidentityassociation) | [Mitigation.EKS.1](#mitigationeks1), [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1), [Mitigation.EKS.Addon.5](#mitigationeksaddon5), [Mitigation.EKS.Capability.3](#mitigationekscapability3), [Mitigation.EKS.Cluster.12](#mitigationekscluster12), [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1), [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5), [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2), [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6), [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3) |
| `UntagResource` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry), [Addon](#addon), [Capability](#capability), [Cluster](#cluster), [EksAnywhereSubscription](#eksanywheresubscription), [FargateProfile](#fargateprofile), [IdentityProviderConfig](#identityproviderconfig), [Nodegroup](#nodegroup), [PodIdentityAssociation](#podidentityassociation) | [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1), [Mitigation.EKS.Addon.5](#mitigationeksaddon5), [Mitigation.EKS.Capability.3](#mitigationekscapability3), [Mitigation.EKS.Cluster.12](#mitigationekscluster12), [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1), [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5), [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2), [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6), [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3) |
| `UpdateAccessEntry` | DOCUMENTATION, DATA_PERIMETER | [AccessEntry](#accessentry) | [Mitigation.EKS.Cluster.11](#mitigationekscluster11) |
| `UpdateAddon` | DOCUMENTATION | [Addon](#addon) | [Mitigation.EKS.1](#mitigationeks1), [Mitigation.EKS.2](#mitigationeks2), [Mitigation.EKS.Addon.1](#mitigationeksaddon1), [Mitigation.EKS.Addon.2](#mitigationeksaddon2), [Mitigation.EKS.Addon.3](#mitigationeksaddon3), [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Addon.6](#mitigationeksaddon6) |
| `UpdateCapability` | DOCUMENTATION | [Capability](#capability) | [Mitigation.EKS.Capability.1](#mitigationekscapability1) |
| `UpdateClusterConfig` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.EKS.Cluster.1](#mitigationekscluster1), [Mitigation.EKS.Cluster.2](#mitigationekscluster2), [Mitigation.EKS.Cluster.3](#mitigationekscluster3), [Mitigation.EKS.Cluster.4](#mitigationekscluster4), [Mitigation.EKS.Cluster.5](#mitigationekscluster5), [Mitigation.EKS.Cluster.10](#mitigationekscluster10), [Mitigation.EKS.Cluster.13](#mitigationekscluster13), [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |
| `UpdateClusterVersion` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `UpdateEksAnywhereSubscription` | DOCUMENTATION | [EksAnywhereSubscription](#eksanywheresubscription) |  |
| `UpdateNodegroupConfig` | DOCUMENTATION, DATA_PERIMETER | [Nodegroup](#nodegroup) |  |
| `UpdateNodegroupVersion` | DOCUMENTATION | [Nodegroup](#nodegroup) | [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4), [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7) |
| `UpdatePodIdentityAssociation` | DOCUMENTATION, DATA_PERIMETER | [PodIdentityAssociation](#podidentityassociation) | [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1), [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2) |

Mitigations attach to an operation directly (the operation appears in the mitigation's request parameters) or through a threat whose request parameters include the operation.

### Goal Coverage (20 of 38)

| Mitigation Goal | Covered | Mitigations |
|---|---|---|
| [AI Services Opt-Out Policy](../mitigation-goals.md#ai-services-opt-out-policy) | — |  |
| [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup) | yes | [Mitigation.EKS.Cluster.8](#mitigationekscluster8) |
| [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native) | — |  |
| [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery) | yes | [Mitigation.EKS.Addon.1](#mitigationeksaddon1), [Mitigation.EKS.Cluster.1](#mitigationekscluster1) |
| [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2) | yes | [Mitigation.EKS.Capability.4](#mitigationekscapability4) |
| [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest) | — |  |
| [Cross-Region Replication](../mitigation-goals.md#cross-region-replication) | — |  |
| [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies) | — |  |
| [Customer Code Execution - Govern Code Content](../mitigation-goals.md#customer-code-execution---govern-code-content) | — |  |
| [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage) | — |  |
| [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection) | yes | [Mitigation.EKS.Cluster.14](#mitigationekscluster14) |
| [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics) | yes | [Mitigation.EKS.Addon.6](#mitigationeksaddon6) |
| [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics) | yes | [Mitigation.EKS.Addon.4](#mitigationeksaddon4), [Mitigation.EKS.Cluster.5](#mitigationekscluster5) |
| [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy) | — |  |
| [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature) | — |  |
| [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication) | yes | [Mitigation.EKS.Cluster.10](#mitigationekscluster10) |
| [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network) | yes | [Mitigation.EKS.2](#mitigationeks2) |
| [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources) | yes | [Mitigation.EKS.Cluster.11](#mitigationekscluster11) |
| [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption) | yes | [Mitigation.EKS.Cluster.9](#mitigationekscluster9) |
| [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment) | — |  |
| [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment) | yes | [Mitigation.EKS.FargateProfile.4](#mitigationeksfargateprofile4), [Mitigation.EKS.Nodegroup.5](#mitigationeksnodegroup5) |
| [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks) | yes | [Mitigation.EKS.Addon.2](#mitigationeksaddon2), [Mitigation.EKS.Cluster.2](#mitigationekscluster2), [Mitigation.EKS.FargateProfile.1](#mitigationeksfargateprofile1), [Mitigation.EKS.Nodegroup.1](#mitigationeksnodegroup1), [Mitigation.EKS.PodIdentityAssociation.1](#mitigationekspodidentityassociation1) |
| [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks) | — |  |
| [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement) | yes | [Mitigation.EKS.Cluster.4](#mitigationekscluster4), [Mitigation.EKS.FargateProfile.3](#mitigationeksfargateprofile3), [Mitigation.EKS.Nodegroup.3](#mitigationeksnodegroup3) |
| [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets) | — |  |
| [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources) | yes | [Mitigation.EKS.Cluster.7](#mitigationekscluster7) |
| [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network) | yes | [Mitigation.EKS.1](#mitigationeks1) |
| [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition) | — |  |
| [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention) | — |  |
| [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose) | yes | [Mitigation.EKS.AccessEntry.1](#mitigationeksaccessentry1), [Mitigation.EKS.Addon.5](#mitigationeksaddon5), [Mitigation.EKS.Capability.3](#mitigationekscapability3), [Mitigation.EKS.Cluster.12](#mitigationekscluster12), [Mitigation.EKS.EksAnywhereSubscription.1](#mitigationekseksanywheresubscription1), [Mitigation.EKS.FargateProfile.5](#mitigationeksfargateprofile5), [Mitigation.EKS.IdentityProviderConfig.2](#mitigationeksidentityproviderconfig2), [Mitigation.EKS.Nodegroup.6](#mitigationeksnodegroup6), [Mitigation.EKS.PodIdentityAssociation.3](#mitigationekspodidentityassociation3) |
| [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role) | yes | [Mitigation.EKS.Addon.3](#mitigationeksaddon3), [Mitigation.EKS.Capability.1](#mitigationekscapability1), [Mitigation.EKS.Cluster.3](#mitigationekscluster3), [Mitigation.EKS.FargateProfile.2](#mitigationeksfargateprofile2), [Mitigation.EKS.Nodegroup.2](#mitigationeksnodegroup2), [Mitigation.EKS.PodIdentityAssociation.2](#mitigationekspodidentityassociation2) |
| [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access) | yes | [Mitigation.EKS.Cluster.13](#mitigationekscluster13), [Mitigation.EKS.Nodegroup.7](#mitigationeksnodegroup7) |
| [Service Principal Presigned URL](../mitigation-goals.md#service-principal-presigned-url) | — |  |
| [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration) | yes | [Mitigation.EKS.Capability.2](#mitigationekscapability2), [Mitigation.EKS.IdentityProviderConfig.1](#mitigationeksidentityproviderconfig1) |
| [Turn on Block Public Access (BPA)](../mitigation-goals.md#turn-on-block-public-access-bpa) | — |  |
| [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention) | — |  |
| [Use Nitro instances](../mitigation-goals.md#use-nitro-instances) | yes | [Mitigation.EKS.Cluster.6](#mitigationekscluster6), [Mitigation.EKS.Nodegroup.4](#mitigationeksnodegroup4) |
| [VPC Deployment](../mitigation-goals.md#vpc-deployment) | — |  |

### Impact Rollup

| Impact | Threats |
|---|---|
| [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery) | [Threat.EKS.Cluster.12](#threatekscluster12), [Threat.EKS.Cluster.15](#threatekscluster15), [Threat.EKS.Cluster.17](#threatekscluster17), [Threat.EKS.Nodegroup.4](#threateksnodegroup4), [Threat.EKS.Nodegroup.5](#threateksnodegroup5), [Threat.EKS.Nodegroup.7](#threateksnodegroup7) |
| [Credential Theft](../threat-impacts.md#credential-theft) | [Threat.EKS.Cluster.9](#threatekscluster9) |
| [Data Collection](../threat-impacts.md#data-collection) | [Threat.EKS.1](#threateks1), [Threat.EKS.Addon.4](#threateksaddon4), [Threat.EKS.Addon.5](#threateksaddon5), [Threat.EKS.Capability.4](#threatekscapability4), [Threat.EKS.Cluster.9](#threatekscluster9), [Threat.EKS.Cluster.10](#threatekscluster10), [Threat.EKS.Cluster.13](#threatekscluster13), [Threat.EKS.Cluster.16](#threatekscluster16), [Threat.EKS.Cluster.17](#threatekscluster17), [Threat.EKS.Cluster.18](#threatekscluster18), [Threat.EKS.FargateProfile.3](#threateksfargateprofile3), [Threat.EKS.FargateProfile.4](#threateksfargateprofile4), [Threat.EKS.Nodegroup.6](#threateksnodegroup6), [Threat.EKS.Nodegroup.7](#threateksnodegroup7), [Threat.EKS.Nodegroup.8](#threateksnodegroup8), [Threat.EKS.PodIdentityAssociation.2](#threatekspodidentityassociation2), [Threat.EKS.PodIdentityAssociation.3](#threatekspodidentityassociation3) |
| [Data Destruction](../threat-impacts.md#data-destruction) | [Threat.EKS.Cluster.1](#threatekscluster1), [Threat.EKS.Cluster.14](#threatekscluster14) |
| [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Threat.EKS.Cluster.4](#threatekscluster4) |
| [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Threat.EKS.1](#threateks1), [Threat.EKS.2](#threateks2), [Threat.EKS.Addon.4](#threateksaddon4), [Threat.EKS.Addon.5](#threateksaddon5), [Threat.EKS.Capability.4](#threatekscapability4), [Threat.EKS.Cluster.9](#threatekscluster9), [Threat.EKS.Cluster.10](#threatekscluster10), [Threat.EKS.Cluster.13](#threatekscluster13), [Threat.EKS.Cluster.17](#threatekscluster17), [Threat.EKS.Cluster.18](#threatekscluster18), [Threat.EKS.Cluster.22](#threatekscluster22), [Threat.EKS.Cluster.23](#threatekscluster23), [Threat.EKS.FargateProfile.3](#threateksfargateprofile3), [Threat.EKS.FargateProfile.4](#threateksfargateprofile4), [Threat.EKS.FargateProfile.5](#threateksfargateprofile5), [Threat.EKS.Nodegroup.7](#threateksnodegroup7), [Threat.EKS.Nodegroup.8](#threateksnodegroup8), [Threat.EKS.Nodegroup.9](#threateksnodegroup9), [Threat.EKS.Nodegroup.10](#threateksnodegroup10), [Threat.EKS.PodIdentityAssociation.2](#threatekspodidentityassociation2), [Threat.EKS.PodIdentityAssociation.3](#threatekspodidentityassociation3) |
| [Defense Evasion](../threat-impacts.md#defense-evasion) | [Threat.EKS.AccessEntry.1](#threateksaccessentry1), [Threat.EKS.Addon.1](#threateksaddon1), [Threat.EKS.Addon.2](#threateksaddon2), [Threat.EKS.Addon.3](#threateksaddon3), [Threat.EKS.Addon.6](#threateksaddon6), [Threat.EKS.Addon.7](#threateksaddon7), [Threat.EKS.Capability.1](#threatekscapability1), [Threat.EKS.Capability.5](#threatekscapability5), [Threat.EKS.Capability.6](#threatekscapability6), [Threat.EKS.Cluster.3](#threatekscluster3), [Threat.EKS.Cluster.5](#threatekscluster5), [Threat.EKS.Cluster.7](#threatekscluster7), [Threat.EKS.Cluster.8](#threatekscluster8), [Threat.EKS.Cluster.19](#threatekscluster19), [Threat.EKS.Cluster.21](#threatekscluster21), [Threat.EKS.Cluster.22](#threatekscluster22), [Threat.EKS.Cluster.23](#threatekscluster23), [Threat.EKS.EksAnywhereSubscription.1](#threatekseksanywheresubscription1), [Threat.EKS.FargateProfile.1](#threateksfargateprofile1), [Threat.EKS.FargateProfile.5](#threateksfargateprofile5), [Threat.EKS.IdentityProviderConfig.1](#threateksidentityproviderconfig1), [Threat.EKS.Nodegroup.1](#threateksnodegroup1), [Threat.EKS.Nodegroup.9](#threateksnodegroup9), [Threat.EKS.Nodegroup.10](#threateksnodegroup10), [Threat.EKS.PodIdentityAssociation.1](#threatekspodidentityassociation1) |
| [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Threat.EKS.Cluster.24](#threatekscluster24), [Threat.EKS.FargateProfile.2](#threateksfargateprofile2), [Threat.EKS.FargateProfile.6](#threateksfargateprofile6), [Threat.EKS.Nodegroup.2](#threateksnodegroup2), [Threat.EKS.Nodegroup.11](#threateksnodegroup11) |
| [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Threat.EKS.Cluster.4](#threatekscluster4) |
| [Initial Access](../threat-impacts.md#initial-access) | [Threat.EKS.Addon.4](#threateksaddon4), [Threat.EKS.Capability.2](#threatekscapability2), [Threat.EKS.Capability.3](#threatekscapability3), [Threat.EKS.Capability.4](#threatekscapability4), [Threat.EKS.Cluster.6](#threatekscluster6), [Threat.EKS.Cluster.12](#threatekscluster12), [Threat.EKS.Cluster.13](#threatekscluster13), [Threat.EKS.Cluster.15](#threatekscluster15), [Threat.EKS.Cluster.17](#threatekscluster17), [Threat.EKS.Cluster.20](#threatekscluster20), [Threat.EKS.FargateProfile.3](#threateksfargateprofile3), [Threat.EKS.IdentityProviderConfig.2](#threateksidentityproviderconfig2), [Threat.EKS.IdentityProviderConfig.3](#threateksidentityproviderconfig3), [Threat.EKS.Nodegroup.4](#threateksnodegroup4), [Threat.EKS.Nodegroup.5](#threateksnodegroup5), [Threat.EKS.Nodegroup.7](#threateksnodegroup7), [Threat.EKS.PodIdentityAssociation.2](#threatekspodidentityassociation2) |
| [Lateral Movement](../threat-impacts.md#lateral-movement) | [Threat.EKS.Capability.4](#threatekscapability4), [Threat.EKS.Cluster.11](#threatekscluster11), [Threat.EKS.Nodegroup.3](#threateksnodegroup3) |
| [Malicious Code Execution](../threat-impacts.md#malicious-code-execution) | [Threat.EKS.Cluster.13](#threatekscluster13) |
| [Network Denial of Service](../threat-impacts.md#network-denial-of-service) | [Threat.EKS.Nodegroup.11](#threateksnodegroup11) |
| [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Threat.EKS.Cluster.6](#threatekscluster6) |
| [Service Stop](../threat-impacts.md#service-stop) | [Threat.EKS.Cluster.2](#threatekscluster2) |
