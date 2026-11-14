# Amazon Elastic Container Service (ecs) Threat Model

[Threat Summary](#threat-summary) · [Threats](#threats) · [Mitigations](#mitigations) · [Implementations](#implementations) · [Appendix](#appendix)

## Overview

Amazon Elastic Container Service is a fully managed container orchestration service. It runs and scales containerized applications without requiring you to operate a control plane. You describe your application once, then let the service place, monitor, and replace containers to match the state you asked for.

93 threats and 59 mitigations across 11 resource types and 77 API operations. 17 of 38 mitigation goals are covered, implemented by 160 implementations.

Source service model: [ecs](https://github.com/aws/api-models-aws/blob/3d89dd029ebd13b775ab5d22e5075399f20b35e4/models/ecs/service/2014-11-13/ecs-2014-11-13.json)

---

## Threat Summary

### [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.9](#threatecscapacityprovider9) | Threat actor, with control of an Amazon ECS capacity provider configured with overly permissive outbound security group rules, can use the capacity provider to interact with ungoverned network destinations. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8) |
| [Threat.ECS.ExpressGatewayService.9](#threatecsexpressgatewayservice9) | Threat actor, with control of an Amazon ECS express gateway service configured with overly permissive outbound security group rules, can use the express gateway service to interact with ungoverned network destinations over HTTPS, HTTP, or DNS. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7) |
| [Threat.ECS.Service.17](#threatecsservice17) | Threat actor, with control of an Amazon ECS service configured with overly permissive outbound security group rules, can use the service to interact with ungoverned network destinations over HTTPS or HTTP. | [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.Service.11](#mitigationecsservice11) |
| [Threat.ECS.Task.12](#threatecstask12) | Threat actor, with control of an Amazon ECS task configured with overly permissive outbound security group rules, can use the task to interact with ungoverned network destinations. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Task.8](#mitigationecstask8) |
| [Threat.ECS.TaskSet.7](#threatecstaskset7) | Threat actor, with control of an Amazon ECS task set configured with overly permissive outbound security group rules, can use the task set to interact with ungoverned network destinations over HTTPS, HTTP, or DNS. | [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5) |

### [Data exposure from missing encryption at rest](../potential-threats.md#data-exposure-from-missing-encryption-at-rest)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Cluster.6](#threatecscluster6) | IAM principal, with IAM permissions to create or modify an Amazon ECS cluster, can create or modify the cluster to not use encryption at rest. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.Cluster.2](#mitigationecscluster2), [Mitigation.ECS.Cluster.4](#mitigationecscluster4) |
| [Threat.ECS.Service.6](#threatecsservice6) | IAM principal, with IAM permissions to create or modify an Amazon ECS service, can create or modify the service to not use encryption at rest. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.Service.3](#mitigationecsservice3), [Mitigation.ECS.Service.6](#mitigationecsservice6) |
| [Threat.ECS.Task.4](#threatecstask4) | IAM principal, with IAM permissions to create an Amazon ECS task's managed EBS volume, can configure the managed EBS volume on creation to not use encryption at rest. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.Task.3](#mitigationecstask3), [Mitigation.ECS.Task.5](#mitigationecstask5) |

### [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Service.18](#threatecsservice18) | Threat actor, with network adjacency to the connection path between clients and an Amazon ECS service that has not been configured to encrypt data in transit, can capture the service's network traffic and read data in transit that has not been encrypted. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.ECS.Service.8](#mitigationecsservice8) |
| [Threat.ECS.TaskDefinition.9](#threatecstaskdefinition9) | Threat actor, with network adjacency to the connection path between clients and an Amazon ECS task definition's Amazon EFS volume that has not been configured to encrypt data in transit, can capture Amazon EFS volume network traffic and read data in transit that has not been encrypted. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5) |

### [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Cluster.2](#threatecscluster2) | IAM principal, with IAM permissions to create or modify an Amazon ECS cluster's managed storage and Fargate ephemeral storage, can encrypt the Amazon ECS managed storage and Fargate ephemeral storage for tasks in the cluster, including the Amazon EBS volumes attached to tasks, with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Mitigation.ECS.Cluster.2](#mitigationecscluster2) |
| [Threat.ECS.Service.2](#threatecsservice2) | IAM principal, with IAM permissions to create or modify an Amazon ECS service's managed EBS volume and Service Connect TLS private key, can encrypt the data stored on the managed EBS volume that Amazon ECS creates for each task in the service, any snapshots created from it, and the Service Connect TLS private keys that Amazon ECS stores in AWS Secrets Manager, with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.ECS.Service.6](#mitigationecsservice6) |
| [Threat.ECS.Task.1](#threatecstask1) | IAM principal, with IAM permissions to create an Amazon ECS task's managed EBS volume, can encrypt the managed EBS volume attached to the task with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Mitigation.ECS.Task.5](#mitigationecstask5) |

### [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.AccountSetting.2](#threatecsaccountsetting2) | IAM principal, with IAM permissions to create or modify an Amazon ECS account setting that supports Container Insights with enhanced observability, can create or configure the account setting without Container Insights with enhanced observability enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1) |
| [Threat.ECS.CapacityProvider.3](#threatecscapacityprovider3) | IAM principal, with IAM permissions to create or modify an Amazon ECS capacity provider that supports detailed monitoring for Amazon ECS Managed Instances, can create or configure the capacity provider without detailed monitoring enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7) |
| [Threat.ECS.Cluster.5](#threatecscluster5) | IAM principal, with IAM permissions to create or modify an Amazon ECS cluster that supports Container Insights with enhanced observability, can create or configure the cluster without Container Insights with enhanced observability enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Cluster.5](#mitigationecscluster5) |
| [Threat.ECS.Service.5](#threatecsservice5) | IAM principal, with IAM permissions to create or modify an Amazon ECS service that supports high-resolution service metrics, can create or configure the service without high-resolution service metrics enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Service.10](#mitigationecsservice10) |
| [Threat.ECS.TaskDefinition.3](#threatecstaskdefinition3) | IAM principal, with IAM permissions to create or modify an Amazon ECS task definition that supports an AWS Distro for OpenTelemetry (ADOT) sidecar container for metric collection, can create or configure the task definition without an ADOT sidecar container for metric collection enabled, leaving tasks launched from that revision without task-level CPU, memory, network, and storage metrics or application metrics in Amazon Managed Service for Prometheus. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4) |

### [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Cluster.7](#threatecscluster7) | IAM principal, with IAM permissions to use an Amazon ECS container within a task running in the cluster, can run commands in an ECS Exec session without a record of those commands and their output being persisted by ECS Exec session logging to Amazon CloudWatch Logs or Amazon S3. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Cluster.1](#mitigationecscluster1) |
| [Threat.ECS.DaemonTaskDefinition.4](#threatecsdaemontaskdefinition4) | IAM principal, with IAM permissions to use an Amazon ECS daemon task definition, can run daemon containers whose `ENTRYPOINT` commands write output to the `STDOUT` and `STDERR` I/O streams without a record of that command output being persisted by container log delivery to Amazon CloudWatch Logs through the `awslogs` log driver. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1) |
| [Threat.ECS.Service.13](#threatecsservice13) | IAM principal, with IAM permissions to use an Amazon ECS service, can interact with the service without a record of their activities being persisted by Service Connect proxy logging and access logging to Amazon CloudWatch Logs. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Service.1](#mitigationecsservice1) |
| [Threat.ECS.TaskDefinition.6](#threatecstaskdefinition6) | IAM principal, with IAM permissions to use an Amazon ECS task definition, can run commands inside containers launched from the task definition without a record of the output of those commands being persisted by container log delivery to Amazon CloudWatch Logs through the awslogs log driver. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1) |

### [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.1](#threatecs1) | Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon ECS service API calls. | [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.2](#mitigationecs2) |

### [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.7](#threatecscapacityprovider7) | Untrusted IAM principal, that has been granted access to my Amazon ECS capacity provider's infrastructure IAM role and instance profile role through their trust policies, can assume my infrastructure role and instance profile role and act with their permissions. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2) |
| [Threat.ECS.DaemonTaskDefinition.2](#threatecsdaemontaskdefinition2) | Untrusted IAM principal, that has been granted access to my Amazon ECS daemon task definition's IAM task role and task execution role through their trust policies, can assume my task role and task execution role and act with their permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3) |
| [Threat.ECS.ExpressGatewayService.6](#threatecsexpressgatewayservice6) | Untrusted IAM principal, that has been granted access to my Amazon ECS express gateway service's IAM infrastructure role, task execution role, and task role through their trust policies, can assume my infrastructure role, task execution role, and task role and act with their permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3) |
| [Threat.ECS.Service.12](#threatecsservice12) | Untrusted IAM principal, that has been granted access to my Amazon ECS service's IAM infrastructure role, infrastructure role for load balancers, deployment lifecycle hook role, and service role through their trust policies, can assume my infrastructure role, infrastructure role for load balancers, deployment lifecycle hook role, and service role and act with their permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.Service.2](#mitigationecsservice2) |
| [Threat.ECS.Task.10](#threatecstask10) | Untrusted IAM principal, that has been granted access to my Amazon ECS task's IAM task role, IAM execution role, and IAM infrastructure role through their trust policies, can assume my task role, execution role, and infrastructure role, and act with their permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.Task.2](#mitigationecstask2) |
| [Threat.ECS.TaskDefinition.4](#threatecstaskdefinition4) | Untrusted IAM principal, that has been granted access to my Amazon ECS task definition's IAM task role and IAM task execution role through their trust policies, can assume my task role and task execution role and act with their permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3) |
| [Threat.ECS.TaskSet.6](#threatecstaskset6) | Untrusted IAM principal, that has been granted access to my Amazon ECS task set's IAM infrastructure role for load balancers through its trust policy, can assume my infrastructure role for load balancers and act with its permissions. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1) |

### [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.ExpressGatewayService.3](#threatecsexpressgatewayservice3) | Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon ECS express gateway service, can interact with the express gateway service's Fargate task elastic network interface on the listening container port over HTTP through overly permissive network rules and move laterally within the VPC. | [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7) |
| [Threat.ECS.Service.9](#threatecsservice9) | Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon ECS service, can interact with the task network interfaces the service maintains, over HTTP, HTTPS, gRPC or TCP, through overly permissive network rules and move laterally within the VPC. | [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.ECS.Service.11](#mitigationecsservice11) |
| [Threat.ECS.Task.7](#threatecstask7) | Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon ECS task, can interact with the task's elastic network interface and its container ports through overly permissive network rules and move laterally within the VPC. | [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.ECS.Task.8](#mitigationecstask8) |
| [Threat.ECS.TaskSet.3](#threatecstaskset3) | Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon ECS task set, such as a neighbouring task, an Amazon EC2 container instance, or a bastion host, can interact with the task set's tasks over their dedicated task elastic network interfaces, on the container ports the application listens on, through overly permissive network rules and move laterally within the VPC. | [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5) |

### [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Cluster.4](#threatecscluster4) | IAM principal, with IAM permissions to create or modify an Amazon ECS cluster, can encrypt its tasks' Amazon EBS volumes and Fargate ephemeral storage with an AWS owned or AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle. | [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Cluster.2](#mitigationecscluster2), [Mitigation.ECS.Cluster.4](#mitigationecscluster4) |
| [Threat.ECS.Service.4](#threatecsservice4) | IAM principal, with IAM permissions to create or modify an Amazon ECS service's managed EBS volume and Service Connect TLS private key, can encrypt the managed EBS volume and Service Connect TLS private key with an AWS owned or AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle. | [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Service.6](#mitigationecsservice6) |
| [Threat.ECS.Task.3](#threatecstask3) | IAM principal, with IAM permissions to create an Amazon ECS task's managed EBS volume, can encrypt the managed EBS volume with an AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle. | [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Task.5](#mitigationecstask5) |

### [Logging disablement](../potential-threats.md#logging-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Cluster.8](#threatecscluster8) | IAM principal, with IAM permissions to create or modify an Amazon ECS cluster's execute command configuration, can disable ECS Exec session logging for the execute command configuration. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Cluster.1](#mitigationecscluster1) |
| [Threat.ECS.DaemonTaskDefinition.5](#threatecsdaemontaskdefinition5) | IAM principal, with IAM permissions to create an Amazon ECS daemon task definition revision and to modify the daemon that references it, can disable CloudWatch Logs delivery for the daemon task definition by registering a superseding revision in the same family whose container definitions omit the log configuration and pointing the daemon at that revision. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1) |
| [Threat.ECS.ExpressGatewayService.8](#threatecsexpressgatewayservice8) | IAM principal, with IAM permissions to create or modify an Amazon ECS express gateway service, can disable container logging to Amazon CloudWatch Logs for the express gateway service. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1) |
| [Threat.ECS.Service.16](#threatecsservice16) | IAM principal, with IAM permissions to modify an Amazon ECS service's Service Connect configuration, can disable Service Connect proxy and access logging to CloudWatch Logs for the Service Connect configuration. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Service.1](#mitigationecsservice1) |
| [Threat.ECS.TaskDefinition.8](#threatecstaskdefinition8) | IAM principal, with IAM permissions to modify an Amazon ECS task definition by registering a new revision in its family, and to use that revision in a task or service, can disable container log delivery to Amazon CloudWatch Logs through the `awslogs` log driver for the task definition by omitting the log configuration from the new revision, then starting replacement tasks whose container logs no longer reach Amazon CloudWatch Logs. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1) |

### [Monitoring disablement](../potential-threats.md#monitoring-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.AccountSetting.1](#threatecsaccountsetting1) | IAM principal, with IAM permissions to modify or delete an Amazon ECS account setting, can disable Container Insights with enhanced observability for the account setting. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1) |
| [Threat.ECS.CapacityProvider.1](#threatecscapacityprovider1) | IAM principal, with IAM permissions to modify an Amazon ECS capacity provider or to turn off detailed monitoring on the Amazon EC2 instances it launches, can disable detailed CloudWatch monitoring of Amazon ECS Managed Instances for the capacity provider. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7) |
| [Threat.ECS.Cluster.1](#threatecscluster1) | IAM principal, with IAM permissions to modify an Amazon ECS cluster, can disable Container Insights with enhanced observability for the cluster. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Cluster.5](#mitigationecscluster5) |
| [Threat.ECS.Service.1](#threatecsservice1) | IAM principal, with IAM permissions to modify an Amazon ECS service, can disable high-resolution service metrics for the service. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Service.10](#mitigationecsservice10) |
| [Threat.ECS.TaskDefinition.1](#threatecstaskdefinition1) | IAM principal, with IAM permissions to create, modify or use an Amazon ECS task definition, including registering a new revision in an existing task definition family and running tasks with container overrides, can disable AWS Distro for OpenTelemetry (ADOT) metric collection to Amazon Managed Service for Prometheus for the task definition, by registering a revision that drops or neuters the `aws-otel-collector` container or replaces `$.taskRoleArn`, or by overriding the collector's `$.overrides.containerOverrides[*].command[*]` at launch, so that task-level and application metrics stop being exported while the workload continues to run. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4) |

### [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.ExpressGatewayService.4](#threatecsexpressgatewayservice4) | Threat actor, with network access to an Amazon ECS express gateway service that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the express gateway service. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7) |
| [Threat.ECS.Service.10](#threatecsservice10) | Threat actor, with network access to an Amazon ECS service that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the service. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.Service.11](#mitigationecsservice11) |
| [Threat.ECS.Task.8](#threatecstask8) | Threat actor, with network access to an Amazon ECS task that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the task. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.Task.8](#mitigationecstask8) |
| [Threat.ECS.TaskSet.4](#threatecstaskset4) | Threat actor, with network access to an Amazon ECS task set that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the task set. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5) |

### [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.8](#threatecscapacityprovider8) | Threat actor, with possession of credentials for my Amazon ECS capacity provider's Managed Instances instance profile, can use my Managed Instances instance profile while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1) |
| [Threat.ECS.DaemonTaskDefinition.3](#threatecsdaemontaskdefinition3) | Threat actor, with possession of credentials for my Amazon ECS daemon task definition's task IAM role and task execution role, can use my task IAM role and task execution role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.DaemonTaskDefinition.2](#mitigationecsdaemontaskdefinition2) |
| [Threat.ECS.ExpressGatewayService.7](#threatecsexpressgatewayservice7) | Threat actor, with possession of credentials for my Amazon ECS express gateway service's task IAM role and task execution role, can use my task IAM role and task execution role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.ExpressGatewayService.2](#mitigationecsexpressgatewayservice2) |
| [Threat.ECS.Task.11](#threatecstask11) | Threat actor, with possession of credentials for my Amazon ECS task's task IAM role and task execution IAM role, can use my task IAM role and task execution IAM role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.Task.1](#mitigationecstask1) |
| [Threat.ECS.TaskDefinition.5](#threatecstaskdefinition5) | Threat actor, with possession of credentials for my Amazon ECS task definition's task IAM role and task execution role, can use my task IAM role and task execution role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.TaskDefinition.2](#mitigationecstaskdefinition2) |

### [Network traffic interception through AWS network-inspection features](../potential-threats.md#network-traffic-interception-through-aws-network-inspection-features)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Service.15](#threatecsservice15) | Threat actor, with permissions to configure AWS network-inspection resources that target the network interfaces used by the Amazon ECS service in a VPC, or to redirect traffic to or from those interfaces through an inspection appliance they control, can capture network traffic at the network interfaces used by the service or at the redirected path, and read data in transit that has not been encrypted. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.ECS.Service.8](#mitigationecsservice8) |
| [Threat.ECS.TaskDefinition.7](#threatecstaskdefinition7) | Threat actor, with permissions to configure AWS network-inspection resources that target the network interfaces used by the Amazon ECS task definition's Amazon EFS volume in a VPC, or to redirect traffic to or from those interfaces through an inspection appliance they control, can capture network traffic at the network interfaces used by the Amazon EFS volume or at the redirected path, and read data in transit that has not been encrypted. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5) |

### [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.5](#threatecscapacityprovider5) | Threat actor, with network access to an Amazon ECS capacity provider's managed instance that has been placed in a public subnet, can access the managed instance directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3) |
| [Threat.ECS.ExpressGatewayService.5](#threatecsexpressgatewayservice5) | Threat actor, with network access to an Amazon ECS express gateway service that has been placed in a public subnet, can access the express gateway service directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4) |
| [Threat.ECS.Service.11](#threatecsservice11) | Threat actor, with network access to an Amazon ECS service that has been placed in a public subnet, can access the service directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.Service.4](#mitigationecsservice4) |
| [Threat.ECS.Task.9](#threatecstask9) | Threat actor, with network access to an Amazon ECS task that has been placed in a public subnet, can access the task directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.Task.4](#mitigationecstask4) |
| [Threat.ECS.TaskSet.5](#threatecstaskset5) | Threat actor, with network access to an Amazon ECS task set that has been placed in a public subnet, can access the task set directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2) |

### [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.2](#threatecscapacityprovider2) | IAM principal, with IAM permissions to create or modify an Amazon ECS capacity provider, can configure the capacity provider without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6) |
| [Threat.ECS.Cluster.3](#threatecscluster3) | IAM principal, with IAM permissions to create or modify an Amazon ECS cluster, can configure the cluster without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Cluster.3](#mitigationecscluster3) |
| [Threat.ECS.ContainerInstance.1](#threatecscontainerinstance1) | IAM principal, with IAM permissions to create or modify an Amazon ECS container instance, can configure the container instance without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1) |
| [Threat.ECS.Daemon.1](#threatecsdaemon1) | IAM principal, with IAM permissions to create or modify an Amazon ECS daemon, can configure the daemon without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1) |
| [Threat.ECS.DaemonTaskDefinition.1](#threatecsdaemontaskdefinition1) | IAM principal, with IAM permissions to create or modify an Amazon ECS daemon task definition, can configure the daemon task definition without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4) |
| [Threat.ECS.ExpressGatewayService.1](#threatecsexpressgatewayservice1) | IAM principal, with IAM permissions to create or modify an Amazon ECS express gateway service, can configure the express gateway service without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6) |
| [Threat.ECS.Service.3](#threatecsservice3) | IAM principal, with IAM permissions to create or modify an Amazon ECS service, can configure the service without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Service.9](#mitigationecsservice9) |
| [Threat.ECS.Task.2](#threatecstask2) | IAM principal, with IAM permissions to create or modify an Amazon ECS task, can configure the task without required tagging information, or remove or overwrite the tags on an existing task. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Task.7](#mitigationecstask7) |
| [Threat.ECS.TaskDefinition.2](#threatecstaskdefinition2) | IAM principal, with IAM permissions to create or modify an Amazon ECS task definition, can configure the task definition without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6) |
| [Threat.ECS.TaskSet.1](#threatecstaskset1) | IAM principal, with IAM permissions to create or modify an Amazon ECS task set, can configure the task set without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4) |

### [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.2](#threatecs2) | IAM principal, with IAM permissions on untrusted Amazon ECS resources, can interact with those untrusted resources from a governed VPC network. | [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.1](#mitigationecs1) |

### [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.4](#threatecscapacityprovider4) | IAM principal, with IAM permissions to create or modify an Amazon ECS capacity provider, can create the capacity provider with its Amazon ECS Managed Instances in a single Availability Zone by specifying only subnets that reside in that zone, or narrow the subnet list of an existing capacity provider to one zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5) |
| [Threat.ECS.ExpressGatewayService.2](#threatecsexpressgatewayservice2) | IAM principal, with IAM permissions to create an Amazon ECS express gateway service, can create the express gateway service with its Fargate tasks and their elastic network interfaces in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5) |
| [Threat.ECS.Service.8](#threatecsservice8) | IAM principal, with IAM permissions to create or modify an Amazon ECS service, can create the service with its tasks in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.Service.5](#mitigationecsservice5), [Mitigation.ECS.Service.7](#mitigationecsservice7) |
| [Threat.ECS.Task.6](#threatecstask6) | IAM principal, with IAM permissions to create an Amazon ECS task, can create the task in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.Task.6](#mitigationecstask6) |
| [Threat.ECS.TaskSet.2](#threatecstaskset2) | IAM principal, with IAM permissions to create an Amazon ECS task set, can create the task set with its tasks in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3) |

### [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.11](#threatecscapacityprovider11) | Threat actor, with access to an Amazon ECS capacity provider's managed instance, can submit a volume of requests that exceeds the capacity of the managed instance or the network it is connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3) |
| [Threat.ECS.ExpressGatewayService.11](#threatecsexpressgatewayservice11) | Threat actor, with access to an Amazon ECS express gateway service, can submit a volume of requests that exceeds the capacity of the express gateway service or the network it is connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4) |
| [Threat.ECS.Service.20](#threatecsservice20) | Threat actor, with access to an Amazon ECS service, can submit a volume of requests that exceeds the capacity of the service or the network it is connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.Service.4](#mitigationecsservice4) |
| [Threat.ECS.Task.14](#threatecstask14) | Threat actor, with access to an Amazon ECS task, can submit a volume of requests that exceeds the capacity of the task or the network it is connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.Task.4](#mitigationecstask4) |
| [Threat.ECS.TaskSet.9](#threatecstaskset9) | Threat actor, with access to an Amazon ECS task set, can submit a volume of requests that exceeds the capacity of the task set or the network it is connected to. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2) |

### [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Service.14](#threatecsservice14) | IAM principal, with unintended IAM permissions to read the private key material from an Amazon ECS service's Service Connect TLS private key that has been encrypted with an AWS owned KMS key, can read the private key material in the Service Connect TLS private key. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.Service.6](#mitigationecsservice6) |

### [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.Service.7](#threatecsservice7) | IAM principal, with unintended IAM permissions to read the managed Amazon EBS volume data from an Amazon ECS service where Amazon ECS uses grant-based KMS authorization, can read the managed Amazon EBS volume data in the service because the KMS key policy cannot restrict which IAM principals can decrypt the data. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.Service.4](#mitigationecsservice4), [Mitigation.ECS.Service.11](#mitigationecsservice11) |
| [Threat.ECS.Task.5](#threatecstask5) | IAM principal, with unintended IAM permissions to read task data in ephemeral storage from an Amazon ECS task where the service uses grant-based KMS authorization, can read task data in ephemeral storage in the task because the KMS key policy cannot restrict which IAM principals can decrypt the data. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.ECS.Task.4](#mitigationecstask4), [Mitigation.ECS.Task.8](#mitigationecstask8) |

### [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.10](#threatecscapacityprovider10) | Threat actor, who can execute code in an Amazon ECS capacity provider's managed instance that is attached to a subnet with a route to an internet gateway, can send the managed instance's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3) |
| [Threat.ECS.ExpressGatewayService.10](#threatecsexpressgatewayservice10) | Threat actor, who can execute code in an Amazon ECS express gateway service that is attached to a subnet with a route to an internet gateway, can send the express gateway service's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4) |
| [Threat.ECS.Service.19](#threatecsservice19) | Threat actor, who can execute code in an Amazon ECS service that is attached to a subnet with a route to an internet gateway, can send the service's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Service.4](#mitigationecsservice4) |
| [Threat.ECS.Task.13](#threatecstask13) | Threat actor, who can execute code in an Amazon ECS task that is attached to a subnet with a route to an internet gateway, can send the task's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.Task.4](#mitigationecstask4) |
| [Threat.ECS.TaskSet.8](#threatecstaskset8) | Threat actor, who can execute code in an Amazon ECS task set that is attached to a subnet with a route to an internet gateway, can send the task set's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2) |

### [Unencrypted instance-to-instance traffic through instance type selection](../potential-threats.md#unencrypted-instance-to-instance-traffic-through-instance-type-selection)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.ECS.CapacityProvider.6](#threatecscapacityprovider6) | IAM principal, with IAM permissions to create or update an Amazon ECS capacity provider, can configure the capacity provider to use an instance type that does not provide automatic encryption of instance-to-instance traffic. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4) |

---

## Threats

Each scoping statement below describes the exposure the threat's mapped mitigations address. It assumes those mitigations are not applied and makes no claim about your resources.

### Threat.ECS.1

Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon ECS service API calls.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.2](#mitigationecs2)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)): Amazon ECS is reachable from your VPC through an interface VPC endpoint (powered by AWS PrivateLink), for API and container agent traffic. An IAM principal from outside your organization can use that network path to make Amazon ECS API calls with its own credentials. That access enables transfer of data from your VPC to Amazon ECS task definitions and tasks in an AWS account outside your organization.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:ExecuteCommand` | `$.cluster`, `$.command`, `$.container`, `$.interactive`, `$.task` |
| `ecs:RegisterTaskDefinition` | `$.containerDefinitions[*].environment[*].value`, `$.containerDefinitions[*].image`, `$.containerDefinitions[*].secrets[*].valueFrom`, `$.family`, `$.taskRoleArn` |
| `ecs:RunTask` | `$.cluster`, `$.overrides.containerOverrides[*].command[*]`, `$.overrides.taskRoleArn`, `$.taskDefinition` |

### Threat.ECS.2

IAM principal, with IAM permissions on untrusted Amazon ECS resources, can interact with those untrusted resources from a governed VPC network.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.1](#mitigationecs1)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)): Amazon ECS API calls from your VPCs can target clusters, services, task definitions, and tasks in another account. With credentials that hold permissions on them, an IAM principal on your network can run tasks on those clusters with data from your network in container environment variables.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskDefinition` | `$.taskDefinition` |
| `ecs:ExecuteCommand` | `$.cluster`, `$.command`, `$.interactive`, `$.task` |
| `ecs:PutAttributes` | `$.attributes[*].value`, `$.cluster` |
| `ecs:RegisterTaskDefinition` | `$.containerDefinitions[*].environment[*].value`, `$.containerDefinitions[*].image`, `$.family`, `$.taskRoleArn` |
| `ecs:RunTask` | `$.cluster`, `$.overrides.containerOverrides[*].command[*]`, `$.overrides.containerOverrides[*].environment[*].value`, `$.taskDefinition` |

### Threat.ECS.AccountSetting.1

IAM principal, with IAM permissions to modify or delete an Amazon ECS account setting, can disable Container Insights with enhanced observability for the account setting.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): Through your `containerInsights` account setting, Container Insights with enhanced observability can become the default for new clusters, adding task and container metrics beyond the free cluster and service metrics. An IAM principal with permission to modify or delete the account setting can disable it, by setting `$.value` to `enabled` or `disabled`, or by deleting an explicit `enhanced` value.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:DeleteAccountSetting` | `$.name`, `$.principalArn` |
| `ecs:PutAccountSetting` | `$.name`, `$.principalArn`, `$.value` |
| `ecs:PutAccountSettingDefault` | `$.name`, `$.value` |

### Threat.ECS.AccountSetting.2

IAM principal, with IAM permissions to create or modify an Amazon ECS account setting that supports Container Insights with enhanced observability, can create or configure the account setting without Container Insights with enhanced observability enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): Your `containerInsights` account setting defaults to `disabled`, and an IAM principal can create or modify it without enabling Container Insights with enhanced observability. Without task- and container-level metrics, anomalous resource use in a single container, such as a cryptomining burst or a memory spike, blends into cluster- and service-level aggregates.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.settings[*].name`, `$.settings[*].value` |
| `ecs:PutAccountSetting` | `$.name`, `$.principalArn`, `$.value` |
| `ecs:PutAccountSettingDefault` | `$.name`, `$.value` |

### Threat.ECS.CapacityProvider.1

IAM principal, with IAM permissions to modify an Amazon ECS capacity provider or to turn off detailed monitoring on the Amazon EC2 instances it launches, can disable detailed CloudWatch monitoring of Amazon ECS Managed Instances for the capacity provider.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): Detailed monitoring for Amazon ECS Managed Instances is an opt-in setting that provides all CloudWatch metrics for your capacity provider's managed instances in 1-minute periods. An IAM principal with permission to modify the capacity provider or its managed instances can disable it, returning all metrics other than status checks to 5-minute periods.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ec2:UnmonitorInstances` | `$.InstanceId[*]` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.monitoring` |

### Threat.ECS.CapacityProvider.2

IAM principal, with IAM permissions to create or modify an Amazon ECS capacity provider, can configure the capacity provider without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/), [T1578](https://attack.mitre.org/techniques/T1578/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS capacity provider can omit the required tags, or remove them later through `UntagResource`. A capacity provider left without those tags still supplies compute capacity for tasks and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.propagateTags`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.tagKeys[*]` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.propagateTags` |
| `resource-groups-tagging-api:UntagResources` | `$.TagKeys[*]` |

### Threat.ECS.CapacityProvider.3

IAM principal, with IAM permissions to create or modify an Amazon ECS capacity provider that supports detailed monitoring for Amazon ECS Managed Instances, can create or configure the capacity provider without detailed monitoring enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): An IAM principal can create or update your Amazon ECS Managed Instances capacity provider without detailed monitoring by setting `$.managedInstancesProvider.instanceLaunchTemplate.monitoring` to `BASIC`, the default. Without 1-minute CPU, network, and disk metrics, a short burst of anomalous resource use, such as cryptomining or data staging, blends into 5-minute averages.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.cluster`, `$.managedInstancesProvider.instanceLaunchTemplate.monitoring` |
| `ecs:UpdateCapacityProvider` | `$.cluster`, `$.managedInstancesProvider.instanceLaunchTemplate.monitoring` |

### Threat.ECS.CapacityProvider.4

IAM principal, with IAM permissions to create or modify an Amazon ECS capacity provider, can create the capacity provider with its Amazon ECS Managed Instances in a single Availability Zone by specifying only subnets that reside in that zone, or narrow the subnet list of an existing capacity provider to one zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5)

**Frameworks:** MITRE ATT&CK [T1578.005](https://attack.mitre.org/techniques/T1578/005/)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create or modify an Amazon ECS capacity provider can set the subnet list in `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]` so that the provider's Amazon ECS Managed Instances all launch in a single Availability Zone. An impairment of that zone can then make those instances unreachable and interrupt the services and standalone tasks whose capacity provider strategy names the provider.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.cluster`, `$.managedInstancesProvider.instanceLaunchTemplate.capacityOptionType`, `$.managedInstancesProvider.instanceLaunchTemplate.capacityReservations.reservationGroupArn`, `$.managedInstancesProvider.instanceLaunchTemplate.capacityReservations.reservationPreference`, `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]`, `$.name` |
| `ecs:UpdateCapacityProvider` | `$.cluster`, `$.managedInstancesProvider.instanceLaunchTemplate.capacityReservations.reservationGroupArn`, `$.managedInstancesProvider.instanceLaunchTemplate.capacityReservations.reservationPreference`, `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]`, `$.name` |

### Threat.ECS.CapacityProvider.5

Threat actor, with network access to an Amazon ECS capacity provider's managed instance that has been placed in a public subnet, can access the managed instance directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): An Amazon ECS capacity provider's managed instance is an Amazon EC2 instance that Amazon ECS launches into a subnet you list on the provider. An Amazon ECS capacity provider's managed instance placed in a subnet that routes to an internet gateway can be reached directly from an ungoverned network when it has a public IP address.

### Threat.ECS.CapacityProvider.6

IAM principal, with IAM permissions to create or update an Amazon ECS capacity provider, can configure the capacity provider to use an instance type that does not provide automatic encryption of instance-to-instance traffic.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4)

**Scoping** (for [Unencrypted instance-to-instance traffic through instance type selection](../potential-threats.md#unencrypted-instance-to-instance-traffic-through-instance-type-selection)): An Amazon ECS capacity provider's allowed instance types setting, `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.AllowedInstanceType[*]`, accepts Amazon EC2 instance types that do not provide automatic encryption of instance-to-instance traffic. If such a type is selected, traffic between the capacity provider's instances themselves and other instances you operate is carried without that encryption.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.AllowedInstanceType[*]`, `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.ExcludedInstanceType[*]` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.AllowedInstanceType[*]`, `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.ExcludedInstanceType[*]` |

### Threat.ECS.CapacityProvider.7

Untrusted IAM principal, that has been granted access to my Amazon ECS capacity provider's infrastructure IAM role and instance profile role through their trust policies, can assume my infrastructure role and instance profile role and act with their permissions.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1580](https://attack.mitre.org/techniques/T1580/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon ECS capacity provider designates an infrastructure role and a Managed Instances instance profile role through `$.managedInstancesProvider.infrastructureRoleArn`, `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn`, and you author their trust policies. Those trust policies can allow identities outside your organization to assume the roles and act with their permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.infrastructureRoleArn`, `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.infrastructureRoleArn`, `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.ECS.CapacityProvider.8

Threat actor, with possession of credentials for my Amazon ECS capacity provider's Managed Instances instance profile, can use my Managed Instances instance profile while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): On each managed instance your Amazon ECS capacity provider provisions, the instance metadata service delivers temporary credentials for the capacity provider's Managed Instances instance profile. A threat actor who obtains those credentials can use that instance profile while on an unexpected network.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:DiscoverPollEndpoint` | `$.cluster`, `$.containerInstance` |
| `ecs:RegisterContainerInstance` | `$.cluster`, `$.instanceIdentityDocument`, `$.instanceIdentityDocumentSignature` |
| `ecs:SubmitAttachmentStateChanges` | `$.attachments[*].attachmentArn`, `$.cluster` |
| `ecs:SubmitTaskStateChange` | `$.cluster`, `$.status`, `$.task` |

### Threat.ECS.CapacityProvider.9

Threat actor, with control of an Amazon ECS capacity provider configured with overly permissive outbound security group rules, can use the capacity provider to interact with ungoverned network destinations.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): An Amazon ECS capacity provider for Managed Instances supplies the security groups whose outbound rules govern connections from the instances it launches. Where those rules permit an unrestricted destination range, a threat actor controlling the capacity provider can use it to interact with ungoverned network destinations.

### Threat.ECS.CapacityProvider.10

Threat actor, who can execute code in an Amazon ECS capacity provider's managed instance that is attached to a subnet with a route to an internet gateway, can send the managed instance's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): An Amazon ECS capacity provider of type `MANAGED_INSTANCES` launches its managed instances, with the network interfaces that carry their outbound traffic, in the subnets you specify. Where those subnets route to an internet gateway, code running on those managed instances can reach the public internet directly when those interfaces have a public IP address.

### Threat.ECS.CapacityProvider.11

Threat actor, with access to an Amazon ECS capacity provider's managed instance, can submit a volume of requests that exceeds the capacity of the managed instance or the network it is connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3)

**Frameworks:** MITRE ATT&CK [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1499.002](https://attack.mitre.org/techniques/T1499/002/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): Each managed instance that an Amazon ECS capacity provider launches into your subnets receives HTTP, HTTPS, TCP, or UDP requests and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

### Threat.ECS.Cluster.1

IAM principal, with IAM permissions to modify an Amazon ECS cluster, can disable Container Insights with enhanced observability for the cluster.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Cluster.5](#mitigationecscluster5)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): Your cluster can carry an opt-in monitoring feature: Container Insights with enhanced observability for detailed Amazon CloudWatch metrics. An IAM principal with permission to modify the cluster can disable it, by setting `$.settings[*].value` to `disabled`.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:PutAccountSetting` | `$.name`, `$.principalArn`, `$.value` |
| `ecs:PutAccountSettingDefault` | `$.name`, `$.value` |
| `ecs:UpdateCluster` | `$.settings[*].name`, `$.settings[*].value` |
| `ecs:UpdateClusterSettings` | `$.settings[*].name`, `$.settings[*].value` |

### Threat.ECS.Cluster.2

IAM principal, with IAM permissions to create or modify an Amazon ECS cluster's managed storage and Fargate ephemeral storage, can encrypt the Amazon ECS managed storage and Fargate ephemeral storage for tasks in the cluster, including the Amazon EBS volumes attached to tasks, with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact)

**Mitigations:** [Mitigation.ECS.Cluster.2](#mitigationecscluster2)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal able to create or update your Amazon ECS cluster chooses the KMS key that encrypts the storage for tasks in your cluster by setting `$.configuration.managedStorageConfiguration.kmsKeyId` and `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`. If that key material sits outside your control, its holder can later revoke or delete it and leave that storage unreadable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, `$.configuration.managedStorageConfiguration.kmsKeyId` |
| `ecs:UpdateCluster` | `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, `$.configuration.managedStorageConfiguration.kmsKeyId` |

### Threat.ECS.Cluster.3

IAM principal, with IAM permissions to create or modify an Amazon ECS cluster, can configure the cluster without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Cluster.3](#mitigationecscluster3)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS cluster can omit the required tags, or remove them later through `UntagResource`. A cluster left without those tags still hosts your containerized applications and may fall outside ownership attribution and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.clusterName`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |
| `resource-groups-tagging-api:TagResources` | `$.ResourceARNList[*]`, `$.Tags{key}` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.ECS.Cluster.4

IAM principal, with IAM permissions to create or modify an Amazon ECS cluster, can encrypt its tasks' Amazon EBS volumes and Fargate ephemeral storage with an AWS owned or AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Cluster.2](#mitigationecscluster2), [Mitigation.ECS.Cluster.4](#mitigationecscluster4)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal able to create or modify your Amazon ECS cluster chooses whether an AWS owned or AWS managed key protects its tasks' Amazon EBS volumes and Fargate ephemeral storage at rest. You cannot govern either key's policy or lifecycle, and the service's use of an AWS owned key produces no CloudTrail events in your account.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, `$.configuration.managedStorageConfiguration.kmsKeyId` |
| `ecs:UpdateCluster` | `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, `$.configuration.managedStorageConfiguration.kmsKeyId` |

### Threat.ECS.Cluster.5

IAM principal, with IAM permissions to create or modify an Amazon ECS cluster that supports Container Insights with enhanced observability, can create or configure the cluster without Container Insights with enhanced observability enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Cluster.5](#mitigationecscluster5)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): An IAM principal can create or modify your Amazon ECS cluster to run without Container Insights with enhanced observability. Without per-task and per-container metrics, anomalous resource use in one task, such as a cryptomining burst or unexpected outbound traffic, blends into cluster- and service-level averages.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.settings[*].name`, `$.settings[*].value` |
| `ecs:UpdateCluster` | `$.settings[*].name`, `$.settings[*].value` |
| `ecs:UpdateClusterSettings` | `$.settings[*].name`, `$.settings[*].value` |

### Threat.ECS.Cluster.6

IAM principal, with IAM permissions to create or modify an Amazon ECS cluster, can create or modify the cluster to not use encryption at rest.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.Cluster.2](#mitigationecscluster2), [Mitigation.ECS.Cluster.4](#mitigationecscluster4)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Data exposure from missing encryption at rest](../potential-threats.md#data-exposure-from-missing-encryption-at-rest)): Encryption at rest is an optional setting on your Amazon ECS cluster. An IAM principal with permissions to create or modify the cluster can leave the data on Amazon EBS volumes that Amazon ECS creates and attaches to tasks in the cluster without encryption at rest.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.configuration.managedStorageConfiguration.kmsKeyId` |
| `ecs:UpdateCluster` | `$.configuration.managedStorageConfiguration.kmsKeyId` |

### Threat.ECS.Cluster.7

IAM principal, with IAM permissions to use an Amazon ECS container within a task running in the cluster, can run commands in an ECS Exec session without a record of those commands and their output being persisted by ECS Exec session logging to Amazon CloudWatch Logs or Amazon S3.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Cluster.1](#mitigationecscluster1)

**Frameworks:** MITRE ATT&CK [T1609](https://attack.mitre.org/techniques/T1609/)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When ECS Exec session logging is not configured on your Amazon ECS cluster and no `awslogs` log driver is configured in the task definition, no session logs are written by ECS Exec session logging to Amazon CloudWatch Logs or Amazon S3. An IAM principal with permissions to use the containers within tasks running in your cluster can run commands in an ECS Exec session, leaving no record of those commands persisted by ECS Exec session logging.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:ExecuteCommand` | `$.cluster`, `$.command`, `$.container`, `$.interactive`, `$.task` |

### Threat.ECS.Cluster.8

IAM principal, with IAM permissions to create or modify an Amazon ECS cluster's execute command configuration, can disable ECS Exec session logging for the execute command configuration.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Cluster.1](#mitigationecscluster1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to create or modify your Amazon ECS cluster's execute command configuration can set `$.configuration.executeCommandConfiguration.logging` to `NONE`. That change stops ECS Exec session logs from reaching CloudWatch Logs while operator session commands can still run on your cluster's tasks; reverting `$.configuration.executeCommandConfiguration.logging` from `OVERRIDE` to `DEFAULT` stops session logs only for tasks whose task definitions have no `awslogs` log driver.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.configuration.executeCommandConfiguration.logConfiguration.cloudWatchLogGroupName`, `$.configuration.executeCommandConfiguration.logging` |
| `ecs:UpdateCluster` | `$.configuration.executeCommandConfiguration.logConfiguration.cloudWatchLogGroupName`, `$.configuration.executeCommandConfiguration.logConfiguration.s3BucketName`, `$.configuration.executeCommandConfiguration.logging` |

### Threat.ECS.ContainerInstance.1

IAM principal, with IAM permissions to create or modify an Amazon ECS container instance, can configure the container instance without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/), [T1578.002](https://attack.mitre.org/techniques/T1578/002/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS container instance can omit the required tags, or remove them later through `UntagResource`. A container instance left without those tags still runs tasks in your cluster and may fall outside ownership attribution and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ec2:RunInstances` | `$.TagSpecification[*].Tag[*].key`, `$.TagSpecification[*].Tag[*].value`, `$.UserData` |
| `ecs:RegisterContainerInstance` | `$.cluster`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.ECS.Daemon.1

IAM principal, with IAM permissions to create or modify an Amazon ECS daemon, can configure the daemon without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS daemon can omit the required tags, which `CreateDaemon` treats as optional. A daemon left without those tags still runs on its container instances and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateDaemon` | `$.propagateTags`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UpdateDaemon` | `$.propagateTags` |

### Threat.ECS.DaemonTaskDefinition.1

IAM principal, with IAM permissions to create or modify an Amazon ECS daemon task definition, can configure the daemon task definition without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS daemon task definition can omit the required tags, or remove them later by calling `UntagResource`. A daemon task definition left without those tags still serves as a daemon blueprint and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RegisterDaemonTaskDefinition` | `$.family`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.ECS.DaemonTaskDefinition.2

Untrusted IAM principal, that has been granted access to my Amazon ECS daemon task definition's IAM task role and task execution role through their trust policies, can assume my task role and task execution role and act with their permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon ECS daemon task definition designates a task role and a task execution role through `$.taskRoleArn`, `$.executionRoleArn`, and you author their trust policies. Those trust policies can allow identities outside your organization to assume the roles and act with their permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RegisterDaemonTaskDefinition` | `$.executionRoleArn`, `$.taskRoleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.ECS.DaemonTaskDefinition.3

Threat actor, with possession of credentials for my Amazon ECS daemon task definition's task IAM role and task execution role, can use my task IAM role and task execution role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.DaemonTaskDefinition.2](#mitigationecsdaemontaskdefinition2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1550.001](https://attack.mitre.org/techniques/T1550/001/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): In your daemon task definition, you set the task IAM role and the task execution role, and Amazon ECS supplies temporary credentials for both roles on each container instance that runs the daemon task. A threat actor who obtains those credentials can use those roles while on an unexpected network.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:CreateLogStream` | `$.logGroupName` |
| `ecr:GetAuthorizationToken` | `$.registryIds[*]` |
| `s3:GetObject` | `$.Bucket`, `$.Key` |
| `secrets-manager:GetSecretValue` | `$.SecretId` |

### Threat.ECS.DaemonTaskDefinition.4

IAM principal, with IAM permissions to use an Amazon ECS daemon task definition, can run daemon containers whose `ENTRYPOINT` commands write output to the `STDOUT` and `STDERR` I/O streams without a record of that command output being persisted by container log delivery to Amazon CloudWatch Logs through the `awslogs` log driver.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1)

**Frameworks:** MITRE ATT&CK [T1610](https://attack.mitre.org/techniques/T1610/)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When container log delivery through the `awslogs` log driver is not configured on your Amazon ECS daemon task definition, no container logs are written by the `awslogs` log driver to Amazon CloudWatch Logs. An IAM principal with permissions to use the daemon task definition can run daemon containers whose `ENTRYPOINT` commands write output to the `STDOUT` and `STDERR` I/O streams, leaving no record of that command output persisted by the `awslogs` log driver.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateDaemon` | `$.daemonTaskDefinitionArn` |
| `ecs:RegisterDaemonTaskDefinition` | `$.containerDefinitions[*].command[*]`, `$.containerDefinitions[*].entryPoint[*]`, `$.containerDefinitions[*].image`, `$.containerDefinitions[*].logConfiguration.logDriver`, `$.containerDefinitions[*].logConfiguration.options{key}`, `$.containerDefinitions[*].privileged`, `$.executionRoleArn`, `$.family` |
| `ecs:UpdateDaemon` | `$.daemonTaskDefinitionArn` |

### Threat.ECS.DaemonTaskDefinition.5

IAM principal, with IAM permissions to create an Amazon ECS daemon task definition revision and to modify the daemon that references it, can disable CloudWatch Logs delivery for the daemon task definition by registering a superseding revision in the same family whose container definitions omit the log configuration and pointing the daemon at that revision.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to create or modify your Amazon ECS daemon task definition can register a superseding revision in the same `$.family` that omits `$.containerDefinitions[*].logConfiguration.logDriver` and point your daemon at it through `$.daemonTaskDefinitionArn` if also permitted to modify that daemon. That change stops your opt-in daemon container logs from reaching CloudWatch Logs while each daemon container keeps writing its application output.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RegisterDaemonTaskDefinition` | `$.containerDefinitions[*].logConfiguration.logDriver`, `$.containerDefinitions[*].logConfiguration.options{key}`, `$.family` |
| `ecs:UpdateDaemon` | `$.daemonTaskDefinitionArn` |

### Threat.ECS.ExpressGatewayService.1

IAM principal, with IAM permissions to create or modify an Amazon ECS express gateway service, can configure the express gateway service without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS express gateway service can omit the required tags, or remove them later through `UntagResource`. An express gateway service left without those tags still serves application traffic and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.serviceName`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.ECS.ExpressGatewayService.2

IAM principal, with IAM permissions to create an Amazon ECS express gateway service, can create the express gateway service with its Fargate tasks and their elastic network interfaces in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create an Amazon ECS express gateway service can specify one subnet in `$.networkConfiguration.awsvpcConfiguration.subnets[*]` through `UpdateService` to place the service's Fargate tasks and their elastic network interfaces in a single Availability Zone. An impairment of that zone can then make those tasks unavailable and interrupt the client requests that the service's Application Load Balancer routes to them.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.networkConfiguration.subnets[*]`, `$.scalingTarget.minTaskCount` |
| `ecs:UpdateExpressGatewayService` | `$.networkConfiguration.subnets[*]`, `$.scalingTarget.minTaskCount` |
| `ecs:UpdateService` | `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |

### Threat.ECS.ExpressGatewayService.3

Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon ECS express gateway service, can interact with the express gateway service's Fargate task elastic network interface on the listening container port over HTTP through overly permissive network rules and move laterally within the VPC.

**Impacts:** [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)

**Protocols:** `HTTP`, `HTTPS`

**Scoping** (for [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)): Your Amazon ECS express gateway service's Fargate tasks have elastic network interfaces in your VPC subnets. A threat actor controlling another resource in that VPC can reach those interfaces through overly permissive network rules and move laterally.

### Threat.ECS.ExpressGatewayService.4

Threat actor, with network access to an Amazon ECS express gateway service that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the express gateway service.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1133](https://attack.mitre.org/techniques/T1133/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`

**Scoping** (for [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)): Your Amazon ECS express gateway service associates VPC security groups with the network interface of each Fargate task, which receives a public IP address in a public subnet. An inbound rule permitting an unrestricted source range lets any host that can route to that interface reach its listening services and administrative ports.

### Threat.ECS.ExpressGatewayService.5

Threat actor, with network access to an Amazon ECS express gateway service that has been placed in a public subnet, can access the express gateway service directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): An Amazon ECS express gateway service launches Fargate tasks in the subnets you choose, or in the default VPC's public subnets when you choose none. An express gateway service placed in a subnet that routes to an internet gateway can be reached directly from an ungoverned network when those tasks have a public IP address.

### Threat.ECS.ExpressGatewayService.6

Untrusted IAM principal, that has been granted access to my Amazon ECS express gateway service's IAM infrastructure role, task execution role, and task role through their trust policies, can assume my infrastructure role, task execution role, and task role and act with their permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon ECS express gateway service designates an infrastructure role, a task execution role, and an optional task role through `$.infrastructureRoleArn`, `$.executionRoleArn`, `$.taskRoleArn`, and you author their trust policies. Those trust policies can allow identities outside your organization to assume the roles and act with their permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.executionRoleArn`, `$.infrastructureRoleArn`, `$.taskRoleArn` |
| `ecs:UpdateExpressGatewayService` | `$.executionRoleArn`, `$.taskRoleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.ECS.ExpressGatewayService.7

Threat actor, with possession of credentials for my Amazon ECS express gateway service's task IAM role and task execution role, can use my task IAM role and task execution role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.2](#mitigationecsexpressgatewayservice2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): When you create or update your Amazon ECS express gateway service with a task IAM role and a task execution role, Amazon ECS provides temporary credentials for each role. A threat actor who obtains those credentials can use those roles while on an unexpected network.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:CreateLogStream` | `$.logGroupName` |
| `ecr:BatchGetImage` | `$.repositoryName` |
| `ecr:GetAuthorizationToken` | *(any parameter)* |
| `ecr:GetDownloadUrlForLayer` | `$.layerDigest`, `$.repositoryName` |
| `s3:GetObject` | `$.Bucket`, `$.Key` |
| `secrets-manager:GetSecretValue` | `$.SecretId` |

### Threat.ECS.ExpressGatewayService.8

IAM principal, with IAM permissions to create or modify an Amazon ECS express gateway service, can disable container logging to Amazon CloudWatch Logs for the express gateway service.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to create or modify your Amazon ECS express gateway service can set `$.taskDefinitionArn` on `CreateExpressGatewayService` or `UpdateExpressGatewayService` to a task definition whose `Main` container omits a log configuration. That change stops the container logs that the `awslogs` driver delivers by default from reaching Amazon CloudWatch Logs while the `Main` container keeps running your application code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.taskDefinitionArn` |
| `ecs:UpdateExpressGatewayService` | `$.taskDefinitionArn` |

### Threat.ECS.ExpressGatewayService.9

Threat actor, with control of an Amazon ECS express gateway service configured with overly permissive outbound security group rules, can use the express gateway service to interact with ungoverned network destinations over HTTPS, HTTP, or DNS.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): The outbound rules of the security groups associated with your Amazon ECS express gateway service govern HTTPS, HTTP, and DNS connections from the network interfaces of its Fargate tasks. Where those rules permit an unrestricted destination range, a threat actor controlling the express gateway service can use it to interact with ungoverned network destinations.

### Threat.ECS.ExpressGatewayService.10

Threat actor, who can execute code in an Amazon ECS express gateway service that is attached to a subnet with a route to an internet gateway, can send the express gateway service's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): When you call `CreateExpressGatewayService` or `UpdateExpressGatewayService`, your Amazon ECS express gateway service places its Fargate tasks, with the network interfaces that carry their outbound traffic, in the subnets you specify. Where those subnets route to an internet gateway, code running in those tasks can reach the public internet directly when those interfaces have a public IP address.

### Threat.ECS.ExpressGatewayService.11

Threat actor, with access to an Amazon ECS express gateway service, can submit a volume of requests that exceeds the capacity of the express gateway service or the network it is connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4)

**Frameworks:** MITRE ATT&CK [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1499.002](https://attack.mitre.org/techniques/T1499/002/)

**Protocols:** `HTTPS`, `HTTP`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): Each task that your Amazon ECS express gateway service runs in your subnets receives HTTP requests forwarded from the service's HTTPS endpoint and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

### Threat.ECS.Service.1

IAM principal, with IAM permissions to modify an Amazon ECS service, can disable high-resolution service metrics for the service.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Service.10](#mitigationecsservice10)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): With opt-in high-resolution service metrics, Amazon ECS publishes your service's `CPUUtilization` and `MemoryUtilization` CloudWatch metrics at 20-second resolution instead of the default 60 seconds. An IAM principal with permission to modify the service can disable it, setting `$.monitoring.metricConfigurations[*].resolutionSeconds` back to `60`.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:UpdateService` | `$.monitoring.metricConfigurations[*].metricNames[*]`, `$.monitoring.metricConfigurations[*].resolutionSeconds` |

### Threat.ECS.Service.2

IAM principal, with IAM permissions to create or modify an Amazon ECS service's managed EBS volume and Service Connect TLS private key, can encrypt the data stored on the managed EBS volume that Amazon ECS creates for each task in the service, any snapshots created from it, and the Service Connect TLS private keys that Amazon ECS stores in AWS Secrets Manager, with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.ECS.Service.6](#mitigationecsservice6)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/), [T1490](https://attack.mitre.org/techniques/T1490/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal able to create or update your Amazon ECS service chooses the KMS key that encrypts the service's managed EBS volume data and Service Connect TLS private keys by setting `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` and `$.serviceConnectConfiguration.services[*].tls.kmsKey`. If that key material sits outside your control, its holder can later revoke or delete it and leave that volume data and those private keys unreadable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.serviceConnectConfiguration.services[*].tls.kmsKey`, `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |
| `ecs:UpdateService` | `$.serviceConnectConfiguration.services[*].tls.kmsKey`, `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |

### Threat.ECS.Service.3

IAM principal, with IAM permissions to create or modify an Amazon ECS service, can configure the service without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Service.9](#mitigationecsservice9)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS service can omit the required tags, or later remove them through `UntagResource`. An Amazon ECS service left without those tags still maintains its running tasks and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.cluster`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.tagKeys[*]` |
| `resource-groups-tagging-api:UntagResources` | `$.TagKeys[*]` |

### Threat.ECS.Service.4

IAM principal, with IAM permissions to create or modify an Amazon ECS service's managed EBS volume and Service Connect TLS private key, can encrypt the managed EBS volume and Service Connect TLS private key with an AWS owned or AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Service.6](#mitigationecsservice6)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal able to create or modify your Amazon ECS service's managed EBS volume and Service Connect TLS private key chooses whether an AWS owned or AWS managed key protects them at rest. You cannot govern either key's policy or lifecycle, and the service's use of an AWS owned key produces no CloudTrail events in your account.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.serviceConnectConfiguration.services[*].tls.kmsKey`, `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |
| `ecs:UpdateService` | `$.serviceConnectConfiguration.services[*].tls.kmsKey`, `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |

### Threat.ECS.Service.5

IAM principal, with IAM permissions to create or modify an Amazon ECS service that supports high-resolution service metrics, can create or configure the service without high-resolution service metrics enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Service.10](#mitigationecsservice10)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): An IAM principal can create or update your Amazon ECS service by calling `CreateService` or `UpdateService` without enabling high-resolution service metrics. Without 20-second CPU and memory utilization data points, a short burst of resource abuse in your tasks blends into the default 60-second aggregates.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.cluster`, `$.deploymentController.type`, `$.desiredCount`, `$.monitoring.metricConfigurations[*].metricNames[*]`, `$.monitoring.metricConfigurations[*].resolutionSeconds`, `$.serviceName`, `$.taskDefinition` |
| `ecs:UpdateService` | `$.cluster`, `$.desiredCount`, `$.monitoring.metricConfigurations[*].metricNames[*]`, `$.monitoring.metricConfigurations[*].resolutionSeconds`, `$.service` |

### Threat.ECS.Service.6

IAM principal, with IAM permissions to create or modify an Amazon ECS service, can create or modify the service to not use encryption at rest.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.Service.3](#mitigationecsservice3), [Mitigation.ECS.Service.6](#mitigationecsservice6)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Data exposure from missing encryption at rest](../potential-threats.md#data-exposure-from-missing-encryption-at-rest)): Encryption at rest is an optional setting on your Amazon ECS service. An IAM principal with permissions to create or modify the service can leave the application data on each task's managed EBS volume without encryption at rest.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.volumeConfigurations[*].managedEBSVolume.encrypted` |
| `ecs:UpdateService` | `$.volumeConfigurations[*].managedEBSVolume.encrypted` |

### Threat.ECS.Service.7

IAM principal, with unintended IAM permissions to read the managed Amazon EBS volume data from an Amazon ECS service where Amazon ECS uses grant-based KMS authorization, can read the managed Amazon EBS volume data in the service because the KMS key policy cannot restrict which IAM principals can decrypt the data.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.Service.4](#mitigationecsservice4), [Mitigation.ECS.Service.11](#mitigationecsservice11)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Scoping** (for [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)): Amazon EC2 decrypts your service's managed Amazon EBS volume data under a KMS grant it holds, so the key policy cannot restrict which principals read the data. A customer managed key gives you an audit trail and control over the key's lifecycle and availability, not a per-principal gate on reading the data.

### Threat.ECS.Service.8

IAM principal, with IAM permissions to create or modify an Amazon ECS service, can create the service with its tasks in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.Service.5](#mitigationecsservice5), [Mitigation.ECS.Service.7](#mitigationecsservice7)

**Frameworks:** MITRE ATT&CK [T1578.005](https://attack.mitre.org/techniques/T1578/005/)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create or modify an Amazon ECS service can set its subnet list, `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, to place all of its tasks in a single Availability Zone. An impairment of that zone can then disrupt all of those tasks and leave the service unavailable to the clients that depend on it.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.cluster`, `$.desiredCount`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.placementConstraints[*].expression`, `$.placementConstraints[*].type`, `$.placementStrategy[*].field`, `$.placementStrategy[*].type`, `$.serviceName` |
| `ecs:UpdateService` | `$.cluster`, `$.desiredCount`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.placementConstraints[*].expression`, `$.placementConstraints[*].type`, `$.placementStrategy[*].field`, `$.placementStrategy[*].type`, `$.service` |

### Threat.ECS.Service.9

Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon ECS service, can interact with the task network interfaces the service maintains, over HTTP, HTTPS, gRPC or TCP, through overly permissive network rules and move laterally within the VPC.

**Impacts:** [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.ECS.Service.11](#mitigationecsservice11)

**Frameworks:** MITRE ATT&CK [T1021](https://attack.mitre.org/techniques/T1021/)

**Protocols:** `HTTP`, `HTTPS`, `gRPC`, `TCP`

**Scoping** (for [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)): In `awsvpc` network mode, each task your Amazon ECS service maintains receives a dedicated network interface with a private address in your VPC subnets and, when you enable Service Connect, a proxy listener on the container port or its override. A threat actor controlling another resource in that VPC can reach those interfaces through overly permissive network rules and move laterally.

### Threat.ECS.Service.10

Threat actor, with network access to an Amazon ECS service that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the service.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.Service.11](#mitigationecsservice11)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1133](https://attack.mitre.org/techniques/T1133/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`, `HTTP/2`, `gRPC`, `TCP`, `UDP`

**Scoping** (for [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)): In `awsvpc` network mode, your Amazon ECS service attaches the security groups you supply to the task ENI it provisions for each task, which can carry a public IP address on Fargate. An inbound rule permitting an unrestricted source range lets any host that can route to that ENI reach its listening services and administrative ports.

### Threat.ECS.Service.11

Threat actor, with network access to an Amazon ECS service that has been placed in a public subnet, can access the service directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.Service.4](#mitigationecsservice4)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): An Amazon ECS service places the network interfaces for its tasks in the subnets you choose for it. A service placed in a subnet that routes to an internet gateway can be reached directly from an ungoverned network when those interfaces have a public IP address.

### Threat.ECS.Service.12

Untrusted IAM principal, that has been granted access to my Amazon ECS service's IAM infrastructure role, infrastructure role for load balancers, deployment lifecycle hook role, and service role through their trust policies, can assume my infrastructure role, infrastructure role for load balancers, deployment lifecycle hook role, and service role and act with their permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.Service.2](#mitigationecsservice2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon ECS service designates the infrastructure role, the infrastructure role for load balancers, the deployment lifecycle hook role, and the legacy load balancer service role through `$.volumeConfigurations[*].managedEBSVolume.roleArn`, `$.vpcLatticeConfigurations[*].roleArn`, `$.serviceConnectConfiguration.services[*].tls.roleArn`, `$.loadBalancers[*].advancedConfiguration.roleArn`, `$.deploymentConfiguration.lifecycleHooks[*].roleArn`, `$.role`, and you author their trust policies. Those trust policies can allow identities outside your organization to assume the roles and act with their permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.deploymentConfiguration.lifecycleHooks[*].roleArn`, `$.loadBalancers[*].advancedConfiguration.roleArn`, `$.role`, `$.serviceConnectConfiguration.services[*].tls.roleArn`, `$.volumeConfigurations[*].managedEBSVolume.roleArn`, `$.vpcLatticeConfigurations[*].roleArn` |
| `ecs:UpdateService` | `$.deploymentConfiguration.lifecycleHooks[*].roleArn`, `$.loadBalancers[*].advancedConfiguration.roleArn`, `$.serviceConnectConfiguration.services[*].tls.roleArn`, `$.volumeConfigurations[*].managedEBSVolume.roleArn`, `$.vpcLatticeConfigurations[*].roleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.ECS.Service.13

IAM principal, with IAM permissions to use an Amazon ECS service, can interact with the service without a record of their activities being persisted by Service Connect proxy logging and access logging to Amazon CloudWatch Logs.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Service.1](#mitigationecsservice1)

**Protocols:** `HTTP`, `HTTP2`, `gRPC`, `TCP`

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When Service Connect proxy logging and access logging are not configured on your Amazon ECS service, no proxy or access logs are written by Service Connect proxy logging and access logging to Amazon CloudWatch Logs. An IAM principal with permissions to use the service can send requests to it through Service Connect, leaving no record of those requests persisted by Service Connect proxy logging and access logging.

### Threat.ECS.Service.14

IAM principal, with unintended IAM permissions to read the private key material from an Amazon ECS service's Service Connect TLS private key that has been encrypted with an AWS owned KMS key, can read the private key material in the Service Connect TLS private key.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.Service.6](#mitigationecsservice6)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Scoping** (for [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)): Your Amazon ECS service's Service Connect TLS private key can be encrypted at rest with an AWS owned key. That key type gives you no key policy to govern, no audit record of key use, and no control over the key lifecycle.

**Request parameters**

| Operation | Parameters |
|---|---|
| `secrets-manager:BatchGetSecretValue` | `$.SecretIdList[*]` |
| `secrets-manager:GetSecretValue` | `$.SecretId` |

### Threat.ECS.Service.15

Threat actor, with permissions to configure AWS network-inspection resources that target the network interfaces used by the Amazon ECS service in a VPC, or to redirect traffic to or from those interfaces through an inspection appliance they control, can capture network traffic at the network interfaces used by the service or at the redirected path, and read data in transit that has not been encrypted.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.ECS.Service.8](#mitigationecsservice8)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Scoping** (for [Network traffic interception through AWS network-inspection features](../potential-threats.md#network-traffic-interception-through-aws-network-inspection-features)): Your Amazon ECS service's tasks send and receive traffic through task elastic network interfaces in your VPC, or through their Amazon EC2 container instance's network interface. Each task or container instance network interface is a customer-visible elastic network interface that can be named as a flow log or route target.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ec2:CreateFlowLogs` | `$.LogDestination`, `$.ResourceId[*]`, `$.ResourceType`, `$.TrafficType` |
| `ec2:CreateRoute` | `$.VpcEndpointId`, `$.destinationCidrBlock`, `$.networkInterfaceId`, `$.routeTableId` |
| `ec2:CreateTrafficMirrorSession` | `$.NetworkInterfaceId`, `$.PacketLength`, `$.TrafficMirrorFilterId`, `$.TrafficMirrorTargetId` |
| `ec2:CreateTrafficMirrorTarget` | `$.GatewayLoadBalancerEndpointId`, `$.NetworkInterfaceId`, `$.NetworkLoadBalancerArn` |
| `ec2:ModifyTransitGatewayVpcAttachment` | `$.Options.ApplianceModeSupport`, `$.TransitGatewayAttachmentId` |
| `ec2:ReplaceRoute` | `$.LocalTarget`, `$.VpcEndpointId`, `$.destinationCidrBlock`, `$.networkInterfaceId`, `$.routeTableId` |
| `ecs:UpdateService` | `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |
| `network-firewall:StartFlowCapture` | `$.FirewallArn`, `$.FlowFilters[*].DestinationAddress.AddressDefinition`, `$.FlowFilters[*].SourceAddress.AddressDefinition` |
| `network-firewall:UpdateLoggingConfiguration` | `$.FirewallArn`, `$.LoggingConfiguration.LogDestinationConfigs[*].LogDestination{key}`, `$.LoggingConfiguration.LogDestinationConfigs[*].LogType` |

### Threat.ECS.Service.16

IAM principal, with IAM permissions to modify an Amazon ECS service's Service Connect configuration, can disable Service Connect proxy and access logging to CloudWatch Logs for the Service Connect configuration.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Service.1](#mitigationecsservice1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to modify your Amazon ECS service's Service Connect configuration can remove the Service Connect log configuration, including `$.serviceConnectConfiguration.logConfiguration.logDriver`, or set `$.serviceConnectConfiguration.enabled` to `false`. Either change stops the opt-in Service Connect proxy logs and Service Connect access logs from reaching CloudWatch Logs while requests made to your service continue unrecorded.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:StopServiceDeployment` | `$.serviceDeploymentArn`, `$.stopType` |
| `ecs:UpdateService` | `$.cluster`, `$.forceNewDeployment`, `$.service`, `$.serviceConnectConfiguration.accessLogConfiguration.format`, `$.serviceConnectConfiguration.enabled`, `$.serviceConnectConfiguration.logConfiguration.logDriver`, `$.serviceConnectConfiguration.logConfiguration.options{key}` |

### Threat.ECS.Service.17

Threat actor, with control of an Amazon ECS service configured with overly permissive outbound security group rules, can use the service to interact with ungoverned network destinations over HTTPS or HTTP.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.Service.11](#mitigationecsservice11)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): In `awsvpc` network mode, the outbound rules of the security groups you specify on your Amazon ECS service govern HTTPS and HTTP connections from each task's ENI. Where those rules permit an unrestricted destination range, a threat actor controlling the service can use it to interact with ungoverned network destinations.

### Threat.ECS.Service.18

Threat actor, with network adjacency to the connection path between clients and an Amazon ECS service that has not been configured to encrypt data in transit, can capture the service's network traffic and read data in transit that has not been encrypted.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.ECS.Service.8](#mitigationecsservice8)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Protocols:** `HTTP`, `HTTP/2`, `gRPC`, `TCP`

**Scoping** (for [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)): Clients connect to your Amazon ECS service's tasks through Service Connect endpoints, through load balancer target groups, or directly at task IP addresses. A network-adjacent threat actor can capture unencrypted HTTP, HTTP/2, gRPC, and TCP traffic on those connections and read the request and response payloads.

### Threat.ECS.Service.19

Threat actor, who can execute code in an Amazon ECS service that is attached to a subnet with a route to an internet gateway, can send the service's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Service.4](#mitigationecsservice4)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): Through `CreateService` or `UpdateService`, an Amazon ECS service places the tasks it runs, with the network interfaces that carry their outbound traffic, in the subnets you specify. Where those subnets route to an internet gateway, code running in those tasks can reach the public internet directly when those interfaces have a public IP address.

### Threat.ECS.Service.20

Threat actor, with access to an Amazon ECS service, can submit a volume of requests that exceeds the capacity of the service or the network it is connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.Service.4](#mitigationecsservice4)

**Frameworks:** MITRE ATT&CK [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1499.002](https://attack.mitre.org/techniques/T1499/002/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`, `gRPC`, `HTTP/2`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): Each task your Amazon ECS service runs in the `awsvpc` network mode receives HTTP, HTTPS, gRPC, TCP, or UDP requests on its own elastic network interface in your subnets and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

### Threat.ECS.Task.1

IAM principal, with IAM permissions to create an Amazon ECS task's managed EBS volume, can encrypt the managed EBS volume attached to the task with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact)

**Mitigations:** [Mitigation.ECS.Task.5](#mitigationecstask5)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal able to run or start your Amazon ECS task chooses the KMS key that encrypts the task's managed EBS volume by setting `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId`. If that key material sits outside your control, its holder can later revoke or delete it and leave that volume data unreadable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |
| `ecs:StartTask` | `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |

### Threat.ECS.Task.2

IAM principal, with IAM permissions to create or modify an Amazon ECS task, can configure the task without required tagging information, or remove or overwrite the tags on an existing task.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Task.7](#mitigationecstask7)

**Frameworks:** MITRE ATT&CK [T1036](https://attack.mitre.org/techniques/T1036/), [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS task can omit the required tags, or remove them later with `UntagResource`. A task left without those tags still runs its containers and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.enableECSManagedTags`, `$.propagateTags`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:StartTask` | `$.enableECSManagedTags`, `$.propagateTags`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |
| `resource-groups-tagging-api:TagResources` | `$.ResourceARNList[*]`, `$.Tags{key}` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.ECS.Task.3

IAM principal, with IAM permissions to create an Amazon ECS task's managed EBS volume, can encrypt the managed EBS volume with an AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Task.5](#mitigationecstask5)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal able to create your Amazon ECS task's managed EBS volume chooses whether the AWS managed key protects it at rest. You cannot govern that key's policy or lifecycle.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |
| `ecs:StartTask` | `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |

### Threat.ECS.Task.4

IAM principal, with IAM permissions to create an Amazon ECS task's managed EBS volume, can configure the managed EBS volume on creation to not use encryption at rest.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.Task.3](#mitigationecstask3), [Mitigation.ECS.Task.5](#mitigationecstask5)

**Frameworks:** MITRE ATT&CK [T1578.001](https://attack.mitre.org/techniques/T1578/001/), [T1610](https://attack.mitre.org/techniques/T1610/)

**Scoping** (for [Data exposure from missing encryption at rest](../potential-threats.md#data-exposure-from-missing-encryption-at-rest)): Encryption at rest is an optional setting on your Amazon ECS task's managed EBS volume, chosen when you create it. An IAM principal with permissions to create the task's managed EBS volume can leave the workload data stored on it without encryption at rest.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.volumeConfigurations[*].managedEBSVolume.encrypted`, `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |
| `ecs:StartTask` | `$.volumeConfigurations[*].managedEBSVolume.encrypted`, `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |

### Threat.ECS.Task.5

IAM principal, with unintended IAM permissions to read task data in ephemeral storage from an Amazon ECS task where the service uses grant-based KMS authorization, can read task data in ephemeral storage in the task because the KMS key policy cannot restrict which IAM principals can decrypt the data.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.Task.4](#mitigationecstask4), [Mitigation.ECS.Task.8](#mitigationecstask8)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1119](https://attack.mitre.org/techniques/T1119/)

**Scoping** (for [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)): AWS Fargate decrypts your task data in ephemeral storage under a KMS grant it holds, so the key policy cannot restrict which principals read the data. A customer managed key provides audit trails, lifecycle control, and availability here, not a per-principal authorization gate.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:ExecuteCommand` | `$.cluster`, `$.command`, `$.container`, `$.task` |

### Threat.ECS.Task.6

IAM principal, with IAM permissions to create an Amazon ECS task, can create the task in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.Task.6](#mitigationecstask6)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create an Amazon ECS task can pass a subnet list in `$.networkConfiguration.awsvpcConfiguration.subnets[*]` that places every launched task in a single Availability Zone. An impairment of that zone can then make all of those tasks unreachable together and interrupt the clients that depend on the endpoints they serve.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.capacityProviderStrategy[*].capacityProvider`, `$.cluster`, `$.count`, `$.group`, `$.launchType`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.placementConstraints[*].expression`, `$.placementConstraints[*].type`, `$.placementStrategy[*].field`, `$.placementStrategy[*].type`, `$.taskDefinition` |
| `ecs:StartTask` | `$.cluster`, `$.containerInstances[*]`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.taskDefinition` |

### Threat.ECS.Task.7

Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon ECS task, can interact with the task's elastic network interface and its container ports through overly permissive network rules and move laterally within the VPC.

**Impacts:** [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.ECS.Task.8](#mitigationecstask8)

**Frameworks:** MITRE ATT&CK [T1021](https://attack.mitre.org/techniques/T1021/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`, `HTTP2`, `gRPC`

**Scoping** (for [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)): In `awsvpc` network mode, your Amazon ECS task receives a dedicated elastic network interface with a private IP address in your VPC subnets. A threat actor controlling another resource in that VPC can reach that interface through overly permissive network rules and move laterally.

### Threat.ECS.Task.8

Threat actor, with network access to an Amazon ECS task that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the task.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.Task.8](#mitigationecstask8)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1133](https://attack.mitre.org/techniques/T1133/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`, `gRPC`, `TCP`, `UDP`

**Scoping** (for [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)): In `awsvpc` network mode, the security groups associated with your task control traffic to its dedicated network interface, which can carry a public IP address. An inbound rule permitting an unrestricted source range lets any host that can route to the interface reach its listening services and administrative ports.

### Threat.ECS.Task.9

Threat actor, with network access to an Amazon ECS task that has been placed in a public subnet, can access the task directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.Task.4](#mitigationecstask4)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): An Amazon ECS task that uses the `awsvpc` network mode receives its own elastic network interface in the subnets you choose for it. A task placed in a subnet that routes to an internet gateway can be reached directly from an ungoverned network when that interface has a public IP address.

### Threat.ECS.Task.10

Untrusted IAM principal, that has been granted access to my Amazon ECS task's IAM task role, IAM execution role, and IAM infrastructure role through their trust policies, can assume my task role, execution role, and infrastructure role, and act with their permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.Task.2](#mitigationecstask2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon ECS task designates the task role, the execution role, and the infrastructure role for task-attached Amazon EBS volumes through `$.overrides.taskRoleArn`, `$.overrides.executionRoleArn`, and `$.volumeConfigurations[*].managedEBSVolume.roleArn`, and you author their trust policies. Those trust policies can allow identities outside your organization to assume the roles and act with their permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.overrides.executionRoleArn`, `$.overrides.taskRoleArn`, `$.volumeConfigurations[*].managedEBSVolume.roleArn` |
| `ecs:StartTask` | `$.overrides.executionRoleArn`, `$.overrides.taskRoleArn`, `$.volumeConfigurations[*].managedEBSVolume.roleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.ECS.Task.11

Threat actor, with possession of credentials for my Amazon ECS task's task IAM role and task execution IAM role, can use my task IAM role and task execution IAM role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.Task.1](#mitigationecstask1)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1550.001](https://attack.mitre.org/techniques/T1550/001/), [T1555.006](https://attack.mitre.org/techniques/T1555/006/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): Your task definition or an override at launch sets the task IAM role, whose temporary credentials Amazon ECS delivers to your task's containers, and the task execution IAM role, whose credentials go to the Amazon ECS container agent or Fargate agent. A threat actor who obtains those credentials can use those roles while on an unexpected network.

**Data perimeter:** `RunTask` allows you to specify a service role, referred to as the task role and task execution role for your container operations. The service uses the role to make AWS API calls from your code.

**Data perimeter:** `StartTask` allows you to specify a service role, referred to as the task role and task execution role for your container operations. The service uses the role to make AWS API calls from your code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecr:BatchGetImage` | `$.imageIds[*].imageTag`, `$.repositoryName` |
| `ecr:GetAuthorizationToken` | `$.registryIds[*]` |
| `ecr:GetDownloadUrlForLayer` | `$.layerDigest`, `$.repositoryName` |
| `ecs:UpdateTaskProtection` | `$.expiresInMinutes`, `$.protectionEnabled` |
| `s3:GetObject` | `$.Bucket`, `$.Key` |
| `secrets-manager:GetSecretValue` | `$.SecretId` |
| `ssm:GetParameters` | `$.Names[*]`, `$.WithDecryption` |

### Threat.ECS.Task.12

Threat actor, with control of an Amazon ECS task configured with overly permissive outbound security group rules, can use the task to interact with ungoverned network destinations.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Task.8](#mitigationecstask8)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): In `awsvpc` network mode, an Amazon ECS task is associated at launch with security groups whose outbound rules govern outbound connections from its elastic network interface in your subnets. Where those rules permit an unrestricted destination range, a threat actor controlling the task can use it to interact with ungoverned network destinations.

### Threat.ECS.Task.13

Threat actor, who can execute code in an Amazon ECS task that is attached to a subnet with a route to an internet gateway, can send the task's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.Task.4](#mitigationecstask4)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): For Amazon ECS tasks in the `awsvpc` network mode, `RunTask` and `StartTask` place the elastic network interfaces that carry the tasks' outbound traffic in the subnets you specify. Where those subnets route to an internet gateway, code running in the tasks can reach the public internet directly when those interfaces have a public IP address.

### Threat.ECS.Task.14

Threat actor, with access to an Amazon ECS task, can submit a volume of requests that exceeds the capacity of the task or the network it is connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.Task.4](#mitigationecstask4)

**Frameworks:** MITRE ATT&CK [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1499.002](https://attack.mitre.org/techniques/T1499/002/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`, `gRPC`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): Under the `awsvpc` network mode, your task receives HTTP, HTTPS, TCP, or UDP requests on a dedicated elastic network interface in your subnets and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

### Threat.ECS.TaskDefinition.1

IAM principal, with IAM permissions to create, modify or use an Amazon ECS task definition, including registering a new revision in an existing task definition family and running tasks with container overrides, can disable AWS Distro for OpenTelemetry (ADOT) metric collection to Amazon Managed Service for Prometheus for the task definition, by registering a revision that drops or neuters the `aws-otel-collector` container or replaces `$.taskRoleArn`, or by overriding the collector's `$.overrides.containerOverrides[*].command[*]` at launch, so that task-level and application metrics stop being exported while the workload continues to run.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): Your Amazon ECS task definition can declare an opt-in AWS Distro for OpenTelemetry (ADOT) metric collection sidecar that exports task-level and application metrics to Amazon Managed Service for Prometheus. An IAM principal with permission to call `RegisterTaskDefinition` or `RunTask` can disable it, by registering a revision that drops or neuters the sidecar or by overriding its command at launch.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:DeregisterTaskDefinition` | `$.taskDefinition` |
| `ecs:RegisterTaskDefinition` | `$.containerDefinitions[*].command[*]`, `$.containerDefinitions[*].environment[*].name`, `$.containerDefinitions[*].environment[*].value`, `$.containerDefinitions[*].essential`, `$.containerDefinitions[*].image`, `$.containerDefinitions[*].name`, `$.family`, `$.taskRoleArn` |
| `ecs:RunTask` | `$.overrides.containerOverrides[*].command[*]`, `$.overrides.containerOverrides[*].name`, `$.overrides.taskRoleArn`, `$.taskDefinition` |
| `ecs:UpdateService` | `$.taskDefinition` |

### Threat.ECS.TaskDefinition.2

IAM principal, with IAM permissions to create or modify an Amazon ECS task definition, can configure the task definition without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS task definition can omit the required tags, or remove them later through `UntagResource`. A task definition left without those tags still defines how your tasks run and may fall outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.family`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.ECS.TaskDefinition.3

IAM principal, with IAM permissions to create or modify an Amazon ECS task definition that supports an AWS Distro for OpenTelemetry (ADOT) sidecar container for metric collection, can create or configure the task definition without an ADOT sidecar container for metric collection enabled, leaving tasks launched from that revision without task-level CPU, memory, network, and storage metrics or application metrics in Amazon Managed Service for Prometheus.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): An IAM principal can register a revision of your Amazon ECS task definition that omits the AWS Distro for OpenTelemetry sidecar container for metric collection. Without the task-level CPU, memory, network, and storage metrics that sidecar exports to Amazon Managed Service for Prometheus, anomalous resource use in a single task, such as sustained CPU saturation from cryptomining or a memory spike, blends into the cluster- and service-level aggregates Amazon ECS publishes to CloudWatch by default.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.containerDefinitions[*].command[*]`, `$.containerDefinitions[*].environment[*].name`, `$.containerDefinitions[*].environment[*].value`, `$.containerDefinitions[*].image`, `$.containerDefinitions[*].name`, `$.family`, `$.taskRoleArn` |
| `ecs:RunTask` | `$.taskDefinition` |

### Threat.ECS.TaskDefinition.4

Untrusted IAM principal, that has been granted access to my Amazon ECS task definition's IAM task role and IAM task execution role through their trust policies, can assume my task role and task execution role and act with their permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon ECS task definition designates an IAM task role and an IAM task execution role through `$.taskRoleArn`, `$.executionRoleArn`, and you author their trust policies. Those trust policies can allow identities outside your organization to assume the roles and act with their permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.executionRoleArn`, `$.taskRoleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.ECS.TaskDefinition.5

Threat actor, with possession of credentials for my Amazon ECS task definition's task IAM role and task execution role, can use my task IAM role and task execution role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.TaskDefinition.2](#mitigationecstaskdefinition2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1550.001](https://attack.mitre.org/techniques/T1550/001/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): The task definition sets the task IAM role, whose temporary credentials Amazon ECS issues to code in your task's containers, and the task execution role, whose temporary credentials Amazon ECS issues to the agent running your task. A threat actor who obtains those credentials can use those roles while on an unexpected network.

**Data perimeter:** `RegisterTaskDefinition` allows you to specify a service role, referred to as the task role and task execution role for your container operations. The service uses the role to make AWS API calls from your code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch-logs:CreateLogStream` | `$.logGroupName`, `$.logStreamName` |
| `cloudwatch-logs:PutLogEvents` | `$.logEvents[*].message`, `$.logGroupName`, `$.logStreamName` |
| `ecr:GetAuthorizationToken` | `$.registryIds[*]` |
| `ecr:GetDownloadUrlForLayer` | `$.layerDigest`, `$.repositoryName` |
| `s3:GetObject` | `$.Bucket`, `$.Key` |
| `secrets-manager:GetSecretValue` | `$.SecretId` |

### Threat.ECS.TaskDefinition.6

IAM principal, with IAM permissions to use an Amazon ECS task definition, can run commands inside containers launched from the task definition without a record of the output of those commands being persisted by container log delivery to Amazon CloudWatch Logs through the awslogs log driver.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)

**Frameworks:** MITRE ATT&CK [T1610](https://attack.mitre.org/techniques/T1610/)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When container log delivery through the `awslogs` log driver is not configured on your Amazon ECS task definition, no container logs are written by the `awslogs` log driver to Amazon CloudWatch Logs. An IAM principal with permissions to use the task definition can run commands inside containers launched from it, leaving no record of the output of those commands persisted by the `awslogs` log driver.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.taskDefinition` |
| `ecs:ExecuteCommand` | `$.command`, `$.task` |
| `ecs:RegisterTaskDefinition` | `$.containerDefinitions[*].logConfiguration.logDriver`, `$.family` |
| `ecs:RunTask` | `$.overrides.containerOverrides[*].command[*]`, `$.taskDefinition` |
| `ecs:StartTask` | `$.taskDefinition` |

### Threat.ECS.TaskDefinition.7

Threat actor, with permissions to configure AWS network-inspection resources that target the network interfaces used by the Amazon ECS task definition's Amazon EFS volume in a VPC, or to redirect traffic to or from those interfaces through an inspection appliance they control, can capture network traffic at the network interfaces used by the Amazon EFS volume or at the redirected path, and read data in transit that has not been encrypted.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Scoping** (for [Network traffic interception through AWS network-inspection features](../potential-threats.md#network-traffic-interception-through-aws-network-inspection-features)): Tasks launched from your task definition reach each Amazon EFS volume only through task network interfaces or container instance network interfaces in your VPC. Each task or container instance network interface is a customer-visible elastic network interface that can be named as a flow log or route target.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ec2:CreateRoute` | `$.VpcEndpointId`, `$.destinationCidrBlock`, `$.networkInterfaceId`, `$.routeTableId` |
| `ec2:CreateTrafficMirrorFilterRule` | `$.DestinationCidrBlock`, `$.DestinationPortRange.FromPort`, `$.DestinationPortRange.ToPort` |
| `ec2:CreateTrafficMirrorSession` | `$.NetworkInterfaceId`, `$.TrafficMirrorFilterId`, `$.TrafficMirrorTargetId` |
| `ec2:CreateTrafficMirrorTarget` | `$.GatewayLoadBalancerEndpointId`, `$.NetworkInterfaceId`, `$.NetworkLoadBalancerArn` |
| `ec2:ReplaceRoute` | `$.VpcEndpointId`, `$.destinationCidrBlock`, `$.networkInterfaceId`, `$.routeTableId` |

### Threat.ECS.TaskDefinition.8

IAM principal, with IAM permissions to modify an Amazon ECS task definition by registering a new revision in its family, and to use that revision in a task or service, can disable container log delivery to Amazon CloudWatch Logs through the `awslogs` log driver for the task definition by omitting the log configuration from the new revision, then starting replacement tasks whose container logs no longer reach Amazon CloudWatch Logs.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to create or modify your Amazon ECS task definition can call `RegisterTaskDefinition` to register a new revision that omits the log configuration. That change stops your opt-in container logs from reaching Amazon CloudWatch Logs while tasks launched from that revision keep running.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.containerDefinitions[*].logConfiguration.logDriver`, `$.containerDefinitions[*].logConfiguration.options{key}`, `$.family` |
| `ecs:RunTask` | `$.taskDefinition` |
| `ecs:UpdateService` | `$.taskDefinition` |

### Threat.ECS.TaskDefinition.9

Threat actor, with network adjacency to the connection path between clients and an Amazon ECS task definition's Amazon EFS volume that has not been configured to encrypt data in transit, can capture Amazon EFS volume network traffic and read data in transit that has not been encrypted.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Protocols:** `NFS`

**Scoping** (for [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)): Tasks mount your task definition's Amazon EFS volume over NFS on port 2049 between the Amazon ECS host and the Amazon EFS mount target. A network-adjacent threat actor can capture unencrypted NFS traffic on that path and read the file contents, names, and attributes it carries.

### Threat.ECS.TaskSet.1

IAM principal, with IAM permissions to create or modify an Amazon ECS task set, can configure the task set without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify an Amazon ECS task set can omit the required tags, or later remove them through `UntagResource`. A task set left without those tags still runs your workload tasks and may fall outside ownership attribution and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateTaskSet` | `$.cluster`, `$.service`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:UntagResource` | `$.resourceArn`, `$.tagKeys[*]` |

### Threat.ECS.TaskSet.2

IAM principal, with IAM permissions to create an Amazon ECS task set, can create the task set with its tasks in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create an Amazon ECS task set can use the `$.networkConfiguration.awsvpcConfiguration.subnets[*]` request parameter to place the task set's tasks in a single Availability Zone. An impairment of that zone can then render those tasks unreachable to the clients that use the task set's load balancer or AWS Cloud Map service registry.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateTaskSet` | `$.cluster`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.service` |

### Threat.ECS.TaskSet.3

Threat actor, who controls a VPC-connected resource in the same VPC as an Amazon ECS task set, such as a neighbouring task, an Amazon EC2 container instance, or a bastion host, can interact with the task set's tasks over their dedicated task elastic network interfaces, on the container ports the application listens on, through overly permissive network rules and move laterally within the VPC.

**Impacts:** [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)

**Frameworks:** MITRE ATT&CK [T1021](https://attack.mitre.org/techniques/T1021/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`, `gRPC`

**Scoping** (for [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)): In `awsvpc` network mode, each task in your task set receives a dedicated task elastic network interface with a private IP address in your VPC subnets. A threat actor controlling another resource in that VPC can reach those interfaces through overly permissive network rules and move laterally.

### Threat.ECS.TaskSet.4

Threat actor, with network access to an Amazon ECS task set that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the task set.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`, `gRPC`, `TCP`, `UDP`

**Scoping** (for [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)): The security groups you specify on an Amazon ECS task set govern each task's dedicated elastic network interface, which can carry a public IP address on AWS Fargate. An inbound rule permitting an unrestricted source range lets any host that can route to that interface reach its listening services and administrative ports.

### Threat.ECS.TaskSet.5

Threat actor, with network access to an Amazon ECS task set that has been placed in a public subnet, can access the task set directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): An Amazon ECS task set launches each of its tasks with a dedicated elastic network interface in the subnets you select. A task set placed in a subnet that routes to an internet gateway can be reached directly from an ungoverned network when those interfaces have a public IP address.

### Threat.ECS.TaskSet.6

Untrusted IAM principal, that has been granted access to my Amazon ECS task set's IAM infrastructure role for load balancers through its trust policy, can assume my infrastructure role for load balancers and act with its permissions.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon ECS task set designates an optional infrastructure role for load balancers through `$.loadBalancers[*].advancedConfiguration.roleArn`, and you author its trust policy. That trust policy can allow identities outside your organization to assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `ecs:CreateTaskSet` | `$.loadBalancers[*].advancedConfiguration.roleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.ECS.TaskSet.7

Threat actor, with control of an Amazon ECS task set configured with overly permissive outbound security group rules, can use the task set to interact with ungoverned network destinations over HTTPS, HTTP, or DNS.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): The outbound rules of the security groups your task set specifies at creation govern HTTPS and HTTP traffic from every task it launches. Where those rules permit an unrestricted destination range, a threat actor controlling the task set can use it to interact with ungoverned network destinations.

### Threat.ECS.TaskSet.8

Threat actor, who can execute code in an Amazon ECS task set that is attached to a subnet with a route to an internet gateway, can send the task set's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1567](https://attack.mitre.org/techniques/T1567/)

**Protocols:** `HTTP`, `HTTPS`, `DNS`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): When you create a task set, it launches your containers, with the task network interfaces that carry their outbound traffic, in the subnets you choose. Where those subnets route to an internet gateway, code running in those containers can reach the public internet directly when those interfaces have a public IP address.

### Threat.ECS.TaskSet.9

Threat actor, with access to an Amazon ECS task set, can submit a volume of requests that exceeds the capacity of the task set or the network it is connected to.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2)

**Frameworks:** MITRE ATT&CK [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1499.002](https://attack.mitre.org/techniques/T1499/002/)

**Protocols:** `HTTP`, `HTTPS`, `TCP`, `UDP`

**Scoping** (for [System capacity exhaustion](../potential-threats.md#system-capacity-exhaustion)): Each `awsvpc`-mode task that your Amazon ECS task set launches receives HTTP, HTTPS, TCP, or UDP requests through its elastic network interface in your subnets and has finite capacity. A threat actor who can reach it can send request volume that exhausts that capacity or the network it connects to.

---

## Mitigations

Each scoping statement below names the capability the mitigation goal governs on the entity and the objective of governing it. Each mitigation's implementations describe how to govern or monitor that capability.

### Mitigation.ECS.1

Amazon ECS service API calls on a governed VPC network should only be made against trusted resources or AWS service-owned resources

**Goal:** [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)

**Addresses:** [Threat.ECS.2](#threatecs2)

**Implemented by:** [Implementation.ECS.1](#implementationecs1), [Implementation.ECS.2](#implementationecs2)

**Scoping** (for [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)): Amazon ECS API calls from your VPCs can target resources in another account. Restricting those calls to resources in your organization enforces the resource perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.ECS.2

Amazon ECS service API calls on a governed VPC network should only be made by trusted identities or AWS services

**Goal:** [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)

**Addresses:** [Threat.ECS.1](#threatecs1)

**Implemented by:** [Implementation.ECS.3](#implementationecs3), [Implementation.ECS.4](#implementationecs4)

**Scoping** (for [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)): Amazon ECS API calls from your VPCs can be made by identities outside your organization. Restricting those calls to principals in your organization enforces the identity perimeter on your network.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.ECS.AccountSetting.1

An Amazon ECS account setting should have Container Insights with enhanced observability enabled to collect detailed CloudWatch metrics

**Goal:** [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)

**Addresses:** [Threat.ECS.AccountSetting.1](#threatecsaccountsetting1), [Threat.ECS.AccountSetting.2](#threatecsaccountsetting2)

**Implemented by:** [Implementation.ECS.AccountSetting.1](#implementationecsaccountsetting1), [Implementation.ECS.AccountSetting.2](#implementationecsaccountsetting2), [ManagedControl.ECS.AccountSetting.1](#managedcontrolecsaccountsetting1)

**Scoping** (for [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)): Setting `$.name` to `containerInsights` and `$.value` to `enhanced` turns on Container Insights with enhanced observability for new clusters, adding task and container CloudWatch metrics beyond the default cluster and service metrics. Collecting those metrics improves security visibility by providing finer-grained data for anomaly detection and incident response.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:PutAccountSetting` | `$.name`, `$.principalArn`, `$.value` |
| `ecs:PutAccountSettingDefault` | `$.name`, `$.value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:ListAccountSettings` | `$.effectiveSettings`, `$.name`, `$.principalArn`, `$.value` |

### Mitigation.ECS.CapacityProvider.1

An Amazon ECS capacity provider's Managed Instances instance profile should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.ECS.CapacityProvider.8](#threatecscapacityprovider8)

**Implemented by:** [Implementation.ECS.CapacityProvider.1](#implementationecscapacityprovider1)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): A capacity provider for Amazon ECS Managed Instances uses its `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn` request parameter to designate the Managed Instances instance profile for the instances it launches, where the container agent and your containers can call AWS services with that profile's credentials. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeCapacityProviders` | `$.capacityProviders[*]`, `$.cluster` |

### Mitigation.ECS.CapacityProvider.2

An Amazon ECS capacity provider's infrastructure role should be assumed only by Amazon ECS, and its instance profile role only by Amazon EC2, acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.ECS.CapacityProvider.7](#threatecscapacityprovider7)

**Implemented by:** [Implementation.ECS.CapacityProvider.2](#implementationecscapacityprovider2), [Implementation.ECS.CapacityProvider.3](#implementationecscapacityprovider3), [Implementation.ECS.CapacityProvider.4](#implementationecscapacityprovider4), [Implementation.ECS.CapacityProvider.5](#implementationecscapacityprovider5), [Implementation.ECS.CapacityProvider.6](#implementationecscapacityprovider6)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): An Amazon ECS Managed Instances capacity provider references an infrastructure role through the `$.managedInstancesProvider.infrastructureRoleArn` request parameter and an instance profile role through the `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn` request parameter, and each role's trust policy specifies which principals can assume it. Allowing only Amazon ECS and Amazon EC2, acting on behalf of trusted resources, to assume those roles helps prevent an unintended principal from assuming them.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.infrastructureRoleArn`, `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.infrastructureRoleArn`, `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeCapacityProviders` | `$.capacityProviders[*]`, `$.cluster` |

### Mitigation.ECS.CapacityProvider.3

An Amazon ECS capacity provider's managed instance should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.ECS.CapacityProvider.5](#threatecscapacityprovider5), [Threat.ECS.CapacityProvider.10](#threatecscapacityprovider10), [Threat.ECS.CapacityProvider.11](#threatecscapacityprovider11)

**Implemented by:** [Implementation.ECS.CapacityProvider.7](#implementationecscapacityprovider7), [Implementation.ECS.CapacityProvider.8](#implementationecscapacityprovider8)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): For an Amazon ECS Managed Instances capacity provider, the `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]` request parameter specifies the subnets where Amazon ECS launches its managed instances. Private subnet placement helps prevent direct internet routing to those managed instances, reducing their network exposure.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeCapacityProviders` | `$.capacityProviders[*]`, `$.cluster` |

### Mitigation.ECS.CapacityProvider.4

An Amazon ECS capacity provider should use EC2 instance types that support automatic encryption of instance-to-instance traffic

**Goal:** [Use Nitro instances](../mitigation-goals.md#use-nitro-instances)

**Addresses:** [Threat.ECS.CapacityProvider.6](#threatecscapacityprovider6)

**Implemented by:** [Implementation.ECS.CapacityProvider.9](#implementationecscapacityprovider9), [Implementation.ECS.CapacityProvider.10](#implementationecscapacityprovider10)

**Scoping** (for [Use Nitro instances](../mitigation-goals.md#use-nitro-instances)): An Amazon ECS Managed Instances capacity provider runs your tasks on Amazon EC2 instances that Amazon ECS launches in your account and VPC. The instance type you allow on your capacity provider determines whether traffic that your tasks exchange with other instances can be automatically encrypted in transit, which applies between supported instance types in the same Region on a direct network path.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.cluster`, `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.AllowedInstanceType[*]`, `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.ExcludedInstanceType[*]`, `$.name` |
| `ecs:UpdateCapacityProvider` | `$.cluster`, `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.AllowedInstanceType[*]`, `$.managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.ExcludedInstanceType[*]`, `$.name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeCapacityProviders` | `$.capacityProviders[*]`, `$.cluster` |

### Mitigation.ECS.CapacityProvider.5

An Amazon ECS capacity provider should launch its Amazon ECS Managed Instances in subnets in more than one Availability Zone

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.ECS.CapacityProvider.4](#threatecscapacityprovider4)

**Implemented by:** [Implementation.ECS.CapacityProvider.11](#implementationecscapacityprovider11), [Implementation.ECS.CapacityProvider.12](#implementationecscapacityprovider12), [Implementation.ECS.CapacityProvider.13](#implementationecscapacityprovider13)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): The `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]` subnet list determines where Amazon ECS launches your Amazon ECS capacity provider's Amazon ECS Managed Instances, each with its own primary elastic network interface. Placing those network interfaces in more than one Availability Zone provides network-level redundancy and helps keep connectivity available when one zone is disrupted.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeCapacityProviders` | `$.capacityProviders[*]`, `$.cluster` |

### Mitigation.ECS.CapacityProvider.6

An Amazon ECS capacity provider should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.CapacityProvider.2](#threatecscapacityprovider2)

**Implemented by:** [Implementation.ECS.CapacityProvider.14](#implementationecscapacityprovider14), [Implementation.ECS.CapacityProvider.15](#implementationecscapacityprovider15), [Implementation.ECS.CapacityProvider.16](#implementationecscapacityprovider16), [ManagedControl.ECS.CapacityProvider.1](#managedcontrolecscapacityprovider1)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag a capacity provider through the `CreateCapacityProvider` request parameters `$.tags[*].key` and `$.tags[*].value`, which carry a list of key/value pairs, or later through `TagResource`. Owner and purpose tags help track ownership and the business function each capacity provider supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.name`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeCapacityProviders` | `$.capacityProviders[*]`, `$.include[*]` |
| `ecs:ListTagsForResource` | `$.resourceArn` |

### Mitigation.ECS.CapacityProvider.7

An Amazon ECS capacity provider should have detailed monitoring for Managed Instances enabled to collect detailed CloudWatch metrics

**Goal:** [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)

**Addresses:** [Threat.ECS.CapacityProvider.1](#threatecscapacityprovider1), [Threat.ECS.CapacityProvider.3](#threatecscapacityprovider3)

**Implemented by:** [Implementation.ECS.CapacityProvider.17](#implementationecscapacityprovider17), [Implementation.ECS.CapacityProvider.18](#implementationecscapacityprovider18), [Implementation.ECS.CapacityProvider.19](#implementationecscapacityprovider19)

**Scoping** (for [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)): Detailed monitoring for Amazon ECS Managed Instances, turned on when the `$.managedInstancesProvider.instanceLaunchTemplate.monitoring` instance monitoring setting is `DETAILED`, makes all CloudWatch metrics for the managed instances your capacity provider launches available in 1-minute periods, not only status check metrics. Collecting those metrics improves security visibility by providing finer-grained data for anomaly detection and incident response.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.monitoring` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.monitoring` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeCapacityProviders` | `$.capacityProviders[*]`, `$.cluster` |

### Mitigation.ECS.CapacityProvider.8

An Amazon ECS capacity provider should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.ECS.CapacityProvider.9](#threatecscapacityprovider9)

**Implemented by:** [Implementation.ECS.CapacityProvider.20](#implementationecscapacityprovider20), [Implementation.ECS.CapacityProvider.21](#implementationecscapacityprovider21), [Implementation.ECS.CapacityProvider.22](#implementationecscapacityprovider22)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): For a Managed Instances capacity provider, the security group IDs you supply at `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.securityGroups[*]` control the network traffic allowed to and from your Amazon ECS Managed Instances. Restricting the inbound and outbound rules to the required sources, destinations, protocols, and ports helps reduce the network exposure of your Amazon ECS Managed Instances.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.securityGroups[*]` |
| `ecs:UpdateCapacityProvider` | `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.securityGroups[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeCapacityProviders` | `$.capacityProviders[*]`, `$.cluster` |

### Mitigation.ECS.Cluster.1

An Amazon ECS cluster should be configured to send ECS Exec session logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.ECS.Cluster.7](#threatecscluster7), [Threat.ECS.Cluster.8](#threatecscluster8)

**Implemented by:** [Implementation.ECS.Cluster.1](#implementationecscluster1), [Implementation.ECS.Cluster.2](#implementationecscluster2), [Implementation.ECS.Cluster.3](#implementationecscluster3)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): Containers in your cluster's tasks can record ECS Exec session logs of commands and their output, with delivery controlled by the cluster's ECS Exec log setting. Sending those logs to CloudWatch Logs provides security-relevant visibility for audit trails, access monitoring, or error detection.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.configuration.executeCommandConfiguration.logConfiguration.cloudWatchLogGroupName`, `$.configuration.executeCommandConfiguration.logging` |
| `ecs:UpdateCluster` | `$.configuration.executeCommandConfiguration.logConfiguration.cloudWatchLogGroupName`, `$.configuration.executeCommandConfiguration.logging` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeClusters` | `$.clusters[*]`, `$.include[*]` |

### Mitigation.ECS.Cluster.2

An Amazon ECS cluster's managed storage and Fargate ephemeral storage should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.ECS.Cluster.2](#threatecscluster2), [Threat.ECS.Cluster.4](#threatecscluster4), [Threat.ECS.Cluster.6](#threatecscluster6)

**Implemented by:** [Implementation.ECS.Cluster.4](#implementationecscluster4), [Implementation.ECS.Cluster.5](#implementationecscluster5), [Implementation.ECS.Cluster.6](#implementationecscluster6)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): Your cluster accepts a customer managed key for the Amazon ECS managed storage attached to your tasks and for Fargate ephemeral storage. A customer managed key provides an audit trail of key use and lifecycle control, including disabling the key to make the stored data unavailable.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, `$.configuration.managedStorageConfiguration.kmsKeyId` |
| `ecs:UpdateCluster` | `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, `$.configuration.managedStorageConfiguration.kmsKeyId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeClusters` | `$.clusters[*]`, `$.include[*]` |

### Mitigation.ECS.Cluster.3

An Amazon ECS cluster should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.Cluster.3](#threatecscluster3)

**Implemented by:** [Implementation.ECS.Cluster.7](#implementationecscluster7), [Implementation.ECS.Cluster.8](#implementationecscluster8), [Implementation.ECS.Cluster.9](#implementationecscluster9)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag an Amazon ECS cluster through the `$.tags[*].key` and `$.tags[*].value` request parameters, set as a list of key-value pairs on `CreateCluster` or `TagResource`. Owner and purpose tags help track ownership and the business function each cluster supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.clusterName`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeClusters` | `$.clusters[*]`, `$.include[*]` |
| `ecs:ListTagsForResource` | `$.resourceArn` |

### Mitigation.ECS.Cluster.4

An Amazon ECS cluster's managed storage configuration should encrypt new Amazon ECS managed storage by default with a customer managed KMS key

**Goal:** [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage)

**Addresses:** [Threat.ECS.Cluster.4](#threatecscluster4), [Threat.ECS.Cluster.6](#threatecscluster6)

**Implemented by:** [Implementation.ECS.Cluster.10](#implementationecscluster10), [Implementation.ECS.Cluster.11](#implementationecscluster11), [Implementation.ECS.Cluster.12](#implementationecscluster12)

**Scoping** (for [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage)): An Amazon ECS cluster's managed storage configuration carries the default KMS keys for Amazon EBS volumes attached to tasks and for AWS Fargate ephemeral storage. Once set, that default applies a customer managed key to new Amazon ECS managed storage created without an explicit key.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.clusterName`, `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, `$.configuration.managedStorageConfiguration.kmsKeyId` |
| `ecs:UpdateCluster` | `$.cluster`, `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, `$.configuration.managedStorageConfiguration.kmsKeyId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeClusters` | `$.clusters[*]`, `$.include[*]` |

### Mitigation.ECS.Cluster.5

An Amazon ECS cluster should have Container Insights with enhanced observability enabled to collect detailed CloudWatch metrics

**Goal:** [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)

**Addresses:** [Threat.ECS.Cluster.1](#threatecscluster1), [Threat.ECS.Cluster.5](#threatecscluster5)

**Implemented by:** [Implementation.ECS.Cluster.13](#implementationecscluster13), [Implementation.ECS.Cluster.14](#implementationecscluster14), [Implementation.ECS.Cluster.15](#implementationecscluster15), [ManagedControl.ECS.Cluster.1](#managedcontrolecscluster1)

**Scoping** (for [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)): Your Amazon ECS cluster can enable Container Insights with enhanced observability through `$.settings[*].name` and `$.settings[*].value` (the `containerInsights` cluster setting) to collect cluster-, service-, task-, and container-level CloudWatch metrics beyond the default. Collecting those metrics improves security visibility by providing finer-grained data for anomaly detection and incident response.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateCluster` | `$.settings[*].name`, `$.settings[*].value` |
| `ecs:UpdateCluster` | `$.settings[*].name`, `$.settings[*].value` |
| `ecs:UpdateClusterSettings` | `$.settings[*].name`, `$.settings[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeClusters` | `$.clusters[*]`, `$.include[*]` |

### Mitigation.ECS.ContainerInstance.1

An Amazon ECS container instance should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.ContainerInstance.1](#threatecscontainerinstance1)

**Implemented by:** [Implementation.ECS.ContainerInstance.1](#implementationecscontainerinstance1), [Implementation.ECS.ContainerInstance.2](#implementationecscontainerinstance2)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag a container instance at registration through the `$.tags[*].key` and `$.tags[*].value` key-value pairs on the Amazon ECS container agent's `RegisterContainerInstance` call, or afterward by calling `TagResource`. Owner and purpose tags help track ownership and the business function each container instance supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterContainerInstance` | `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeContainerInstances` | `$.cluster`, `$.containerInstances[*]`, `$.include[*]` |
| `ecs:ListTagsForResource` | `$.resourceArn` |

### Mitigation.ECS.Daemon.1

An Amazon ECS daemon should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.Daemon.1](#threatecsdaemon1)

**Implemented by:** [Implementation.ECS.Daemon.1](#implementationecsdaemon1), [Implementation.ECS.Daemon.2](#implementationecsdaemon2), [Implementation.ECS.Daemon.3](#implementationecsdaemon3)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You can tag a daemon through `CreateDaemon`, whose `$.tags[*].key` and `$.tags[*].value` request parameters carry a list of key-value pairs, or later through `TagResource`. Owner and purpose tags help track ownership and the business function each daemon supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateDaemon` | `$.daemonName`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeDaemon` | `$.daemonArn` |
| `ecs:ListDaemons` | `$.clusterArn` |

### Mitigation.ECS.DaemonTaskDefinition.1

An Amazon ECS daemon task definition should be configured to send container logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.ECS.DaemonTaskDefinition.4](#threatecsdaemontaskdefinition4), [Threat.ECS.DaemonTaskDefinition.5](#threatecsdaemontaskdefinition5)

**Implemented by:** [Implementation.ECS.DaemonTaskDefinition.1](#implementationecsdaemontaskdefinition1), [Implementation.ECS.DaemonTaskDefinition.2](#implementationecsdaemontaskdefinition2), [Implementation.ECS.DaemonTaskDefinition.3](#implementationecsdaemontaskdefinition3)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): Each container in your daemon task definition produces container logs from its `STDOUT` and `STDERR` streams, and its log configuration controls their delivery through the `awslogs` log driver. Sending those logs to CloudWatch Logs provides security-relevant visibility for audit trails, access monitoring, or error detection.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterDaemonTaskDefinition` | `$.containerDefinitions[*].logConfiguration.logDriver`, `$.containerDefinitions[*].logConfiguration.options{key}`, `$.executionRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeDaemonTaskDefinition` | `$.daemonTaskDefinition` |
| `ecs:ListDaemonTaskDefinitions` | `$.family`, `$.status` |

### Mitigation.ECS.DaemonTaskDefinition.2

An Amazon ECS daemon task definition's task IAM role and task execution role should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.ECS.DaemonTaskDefinition.3](#threatecsdaemontaskdefinition3)

**Implemented by:** [Implementation.ECS.DaemonTaskDefinition.4](#implementationecsdaemontaskdefinition4)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): A daemon task definition's `$.taskRoleArn` request parameter designates the task IAM role, whose temporary credentials your daemon's containers use to call AWS services, and its `$.executionRoleArn` request parameter designates the task execution role, whose temporary credentials the Amazon ECS container agent uses to call AWS services for the daemon task. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterDaemonTaskDefinition` | `$.executionRoleArn`, `$.taskRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeDaemonTaskDefinition` | `$.daemonTaskDefinition` |

### Mitigation.ECS.DaemonTaskDefinition.3

An Amazon ECS daemon task definition's task role and task execution role should be assumed only by Amazon ECS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.ECS.DaemonTaskDefinition.2](#threatecsdaemontaskdefinition2)

**Implemented by:** [Implementation.ECS.DaemonTaskDefinition.5](#implementationecsdaemontaskdefinition5), [Implementation.ECS.DaemonTaskDefinition.6](#implementationecsdaemontaskdefinition6), [Implementation.ECS.DaemonTaskDefinition.7](#implementationecsdaemontaskdefinition7), [Implementation.ECS.DaemonTaskDefinition.8](#implementationecsdaemontaskdefinition8)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): A daemon task definition references an Amazon ECS task role and task execution role through the `$.taskRoleArn` and `$.executionRoleArn` request parameters, and each role's trust policy specifies which principals can assume it. Allowing only Amazon ECS, acting on behalf of trusted resources, to assume those roles helps prevent an unintended principal from assuming them.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterDaemonTaskDefinition` | `$.executionRoleArn`, `$.taskRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeDaemonTaskDefinition` | `$.daemonTaskDefinition` |

### Mitigation.ECS.DaemonTaskDefinition.4

An Amazon ECS daemon task definition should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.DaemonTaskDefinition.1](#threatecsdaemontaskdefinition1)

**Implemented by:** [Implementation.ECS.DaemonTaskDefinition.9](#implementationecsdaemontaskdefinition9), [Implementation.ECS.DaemonTaskDefinition.10](#implementationecsdaemontaskdefinition10), [Implementation.ECS.DaemonTaskDefinition.11](#implementationecsdaemontaskdefinition11)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag a daemon task definition through a list of key/value pairs in `$.tags[*].key` and `$.tags[*].value` when you call `RegisterDaemonTaskDefinition`, or later through `TagResource`. Owner and purpose tags help track ownership and the business function each daemon task definition supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterDaemonTaskDefinition` | `$.family`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `resource-groups-tagging-api:GetResources` | `$.ResourceARNList[*]` |

### Mitigation.ECS.ExpressGatewayService.1

An Amazon ECS express gateway service should be configured to send container logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.ECS.ExpressGatewayService.8](#threatecsexpressgatewayservice8)

**Implemented by:** [Implementation.ECS.ExpressGatewayService.1](#implementationecsexpressgatewayservice1), [Implementation.ECS.ExpressGatewayService.2](#implementationecsexpressgatewayservice2), [Implementation.ECS.ExpressGatewayService.3](#implementationecsexpressgatewayservice3)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): The primary container of your Amazon ECS express gateway service records your application's output in container logs, which its `awslogs` log driver configuration directs to a log group. Sending those logs to CloudWatch Logs provides security-relevant visibility for audit trails, access monitoring, or error detection.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.primaryContainer.awsLogsConfiguration.logGroup`, `$.primaryContainer.awsLogsConfiguration.logStreamPrefix` |
| `ecs:UpdateExpressGatewayService` | `$.primaryContainer.awsLogsConfiguration.logGroup`, `$.primaryContainer.awsLogsConfiguration.logStreamPrefix` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeExpressGatewayService` | `$.serviceArn` |

### Mitigation.ECS.ExpressGatewayService.2

An Amazon ECS express gateway service's task IAM role and task execution role should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.ECS.ExpressGatewayService.7](#threatecsexpressgatewayservice7)

**Implemented by:** [Implementation.ECS.ExpressGatewayService.4](#implementationecsexpressgatewayservice4)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): The `$.taskRoleArn` request parameter designates the task IAM role for your Amazon ECS express gateway service, whose containers call AWS services with that role's credentials, and the `$.executionRoleArn` request parameter designates the task execution role, whose credentials the Amazon ECS container and Fargate agents in its tasks use to call AWS services. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.executionRoleArn`, `$.taskRoleArn` |
| `ecs:UpdateExpressGatewayService` | `$.executionRoleArn`, `$.taskRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeExpressGatewayService` | `$.serviceArn` |

### Mitigation.ECS.ExpressGatewayService.3

An Amazon ECS express gateway service's infrastructure role, task execution role, and task role should be assumed only by Amazon ECS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.ECS.ExpressGatewayService.6](#threatecsexpressgatewayservice6)

**Implemented by:** [Implementation.ECS.ExpressGatewayService.5](#implementationecsexpressgatewayservice5), [Implementation.ECS.ExpressGatewayService.6](#implementationecsexpressgatewayservice6), [Implementation.ECS.ExpressGatewayService.7](#implementationecsexpressgatewayservice7), [Implementation.ECS.ExpressGatewayService.8](#implementationecsexpressgatewayservice8), [Implementation.ECS.ExpressGatewayService.9](#implementationecsexpressgatewayservice9)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): An Amazon ECS express gateway service designates an infrastructure role through the `$.infrastructureRoleArn` request parameter, a task execution role through `$.executionRoleArn`, and an optional task role through `$.taskRoleArn`, and each role's trust policy specifies which principals can assume it. Allowing only Amazon ECS, acting on behalf of trusted resources, to assume those roles helps prevent an unintended principal from assuming them.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.executionRoleArn`, `$.infrastructureRoleArn`, `$.taskRoleArn` |
| `ecs:UpdateExpressGatewayService` | `$.executionRoleArn`, `$.taskRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeExpressGatewayService` | `$.serviceArn` |

### Mitigation.ECS.ExpressGatewayService.4

An Amazon ECS express gateway service should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.ECS.ExpressGatewayService.5](#threatecsexpressgatewayservice5), [Threat.ECS.ExpressGatewayService.10](#threatecsexpressgatewayservice10), [Threat.ECS.ExpressGatewayService.11](#threatecsexpressgatewayservice11)

**Implemented by:** [Implementation.ECS.ExpressGatewayService.10](#implementationecsexpressgatewayservice10), [Implementation.ECS.ExpressGatewayService.11](#implementationecsexpressgatewayservice11)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): When you create or update your express gateway service, `$.networkConfiguration.subnets[*]` specifies the IDs of the subnets that hold its Fargate tasks' elastic network interfaces. Private subnet placement helps prevent direct internet routing to those Fargate tasks, reducing their network exposure.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.networkConfiguration.subnets[*]` |
| `ecs:UpdateExpressGatewayService` | `$.networkConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeExpressGatewayService` | `$.serviceArn` |

### Mitigation.ECS.ExpressGatewayService.5

An Amazon ECS express gateway service should launch its tasks in subnets in more than one Availability Zone

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.ECS.ExpressGatewayService.2](#threatecsexpressgatewayservice2)

**Implemented by:** [Implementation.ECS.ExpressGatewayService.12](#implementationecsexpressgatewayservice12), [Implementation.ECS.ExpressGatewayService.13](#implementationecsexpressgatewayservice13), [Implementation.ECS.ExpressGatewayService.14](#implementationecsexpressgatewayservice14)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): An express gateway service runs its tasks in `awsvpc` network mode, so Amazon ECS creates a dedicated network interface for each task in the `$.networkConfiguration.subnets[*]` subnets you configure. Placing those network interfaces in more than one Availability Zone provides network-level redundancy and helps keep connectivity available when one zone is disrupted.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.networkConfiguration.subnets[*]`, `$.scalingTarget.minTaskCount` |
| `ecs:UpdateExpressGatewayService` | `$.networkConfiguration.subnets[*]`, `$.scalingTarget.minTaskCount` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeExpressGatewayService` | `$.serviceArn` |

### Mitigation.ECS.ExpressGatewayService.6

An Amazon ECS express gateway service should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.ExpressGatewayService.1](#threatecsexpressgatewayservice1)

**Implemented by:** [Implementation.ECS.ExpressGatewayService.15](#implementationecsexpressgatewayservice15), [Implementation.ECS.ExpressGatewayService.16](#implementationecsexpressgatewayservice16), [Implementation.ECS.ExpressGatewayService.17](#implementationecsexpressgatewayservice17)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You can tag an Amazon ECS express gateway service at creation through the `$.tags[*].key` and `$.tags[*].value` request parameters, which carry a list of key/value pairs, or afterward through `TagResource`. Owner and purpose tags help track ownership and the business function each express gateway service supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.serviceName`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeExpressGatewayService` | `$.include[*]`, `$.serviceArn` |

### Mitigation.ECS.ExpressGatewayService.7

An Amazon ECS express gateway service should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.ECS.ExpressGatewayService.3](#threatecsexpressgatewayservice3), [Threat.ECS.ExpressGatewayService.4](#threatecsexpressgatewayservice4), [Threat.ECS.ExpressGatewayService.9](#threatecsexpressgatewayservice9)

**Implemented by:** [Implementation.ECS.ExpressGatewayService.18](#implementationecsexpressgatewayservice18), [Implementation.ECS.ExpressGatewayService.19](#implementationecsexpressgatewayservice19), [Implementation.ECS.ExpressGatewayService.20](#implementationecsexpressgatewayservice20)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): You can associate security groups with your express gateway service through `$.networkConfiguration.securityGroups[*]` to govern traffic on the network interface that each Fargate task receives in `awsvpc` network mode, and the groups you supply are layered on top of the Service Security Group that Express Mode creates and attaches unconditionally. Restricting the inbound and outbound rules to the required sources, destinations, protocols, and ports helps reduce the network exposure of your express gateway service.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateExpressGatewayService` | `$.networkConfiguration.securityGroups[*]` |
| `ecs:UpdateExpressGatewayService` | `$.networkConfiguration.securityGroups[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeExpressGatewayService` | `$.serviceArn` |

### Mitigation.ECS.Service.1

An Amazon ECS service's Service Connect configuration should be configured to send Service Connect proxy and access logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.ECS.Service.13](#threatecsservice13), [Threat.ECS.Service.16](#threatecsservice16)

**Implemented by:** [Implementation.ECS.Service.1](#implementationecsservice1), [Implementation.ECS.Service.2](#implementationecsservice2), [Implementation.ECS.Service.3](#implementationecsservice3)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): The managed Service Connect proxy in your service's tasks writes proxy logs about its start-up and access logs with per-request traffic metadata such as HTTP methods, paths and response codes, and your Service Connect log and access log configurations control their delivery. Sending those logs to CloudWatch Logs provides security-relevant visibility for audit trails, access monitoring, or error detection.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.serviceConnectConfiguration.accessLogConfiguration.format`, `$.serviceConnectConfiguration.enabled`, `$.serviceConnectConfiguration.logConfiguration.logDriver`, `$.serviceConnectConfiguration.logConfiguration.options{key}` |
| `ecs:UpdateService` | `$.serviceConnectConfiguration.accessLogConfiguration.format`, `$.serviceConnectConfiguration.enabled`, `$.serviceConnectConfiguration.logConfiguration.logDriver`, `$.serviceConnectConfiguration.logConfiguration.options{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Service.2

An Amazon ECS service's infrastructure role, infrastructure role for load balancers, deployment lifecycle hook role, and service role should be assumed only by Amazon ECS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.ECS.Service.12](#threatecsservice12)

**Implemented by:** [Implementation.ECS.Service.4](#implementationecsservice4), [Implementation.ECS.Service.5](#implementationecsservice5), [Implementation.ECS.Service.6](#implementationecsservice6), [Implementation.ECS.Service.7](#implementationecsservice7)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): When you call `CreateService` or `UpdateService`, you can pass the following IAM roles, each with a trust policy that specifies which principals can assume it: the infrastructure role through `$.volumeConfigurations[*].managedEBSVolume.roleArn`, `$.vpcLatticeConfigurations[*].roleArn`, or `$.serviceConnectConfiguration.services[*].tls.roleArn`, the infrastructure role for load balancers through `$.loadBalancers[*].advancedConfiguration.roleArn`, the deployment lifecycle hook role through `$.deploymentConfiguration.lifecycleHooks[*].roleArn`, and the service role through `$.role` on `CreateService` only. Allowing only Amazon ECS, acting on behalf of trusted resources, to assume those roles helps prevent an unintended principal from assuming them.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.deploymentConfiguration.lifecycleHooks[*].roleArn`, `$.loadBalancers[*].advancedConfiguration.roleArn`, `$.role`, `$.serviceConnectConfiguration.services[*].tls.roleArn`, `$.volumeConfigurations[*].managedEBSVolume.roleArn`, `$.vpcLatticeConfigurations[*].roleArn` |
| `ecs:UpdateService` | `$.deploymentConfiguration.lifecycleHooks[*].roleArn`, `$.loadBalancers[*].advancedConfiguration.roleArn`, `$.serviceConnectConfiguration.services[*].tls.roleArn`, `$.volumeConfigurations[*].managedEBSVolume.roleArn`, `$.vpcLatticeConfigurations[*].roleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceDeployments` | `$.serviceDeploymentArns[*]` |
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Service.3

An Amazon ECS service's managed EBS volume should have encryption at rest enabled

**Goal:** [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest)

**Addresses:** [Threat.ECS.Service.6](#threatecsservice6)

**Implemented by:** [Implementation.ECS.Service.8](#implementationecsservice8), [Implementation.ECS.Service.9](#implementationecsservice9)

**Scoping** (for [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest)): When Amazon ECS creates the Amazon EBS volume for each task in your service, it applies the `$.volumeConfigurations[*].managedEBSVolume.encrypted` request parameter, the optional setting that turns on Amazon EBS encryption. Enabling it helps protect the data on each task's volume.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.cluster`, `$.serviceName`, `$.volumeConfigurations[*].managedEBSVolume.encrypted`, `$.volumeConfigurations[*].name` |
| `ecs:UpdateService` | `$.cluster`, `$.service`, `$.volumeConfigurations[*].managedEBSVolume.encrypted`, `$.volumeConfigurations[*].name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Service.4

An Amazon ECS service should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.ECS.Service.7](#threatecsservice7), [Threat.ECS.Service.11](#threatecsservice11), [Threat.ECS.Service.19](#threatecsservice19), [Threat.ECS.Service.20](#threatecsservice20)

**Implemented by:** [Implementation.ECS.Service.10](#implementationecsservice10), [Implementation.ECS.Service.11](#implementationecsservice11), [ManagedControl.ECS.Service.1](#managedcontrolecsservice1)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): Through the `$.networkConfiguration.awsvpcConfiguration.subnets[*]` request parameter, the required subnet list of the `awsvpc` network configuration, you select the subnets that hold the elastic network interfaces of your service's tasks. Private subnet placement helps prevent direct internet routing to those tasks, reducing their network exposure.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.networkConfiguration.awsvpcConfiguration.assignPublicIp`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |
| `ecs:UpdateService` | `$.networkConfiguration.awsvpcConfiguration.assignPublicIp`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Service.5

An Amazon ECS service should be deployed across more than one Availability Zone with the Availability Zone rebalancing feature

**Goal:** [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment)

**Addresses:** [Threat.ECS.Service.8](#threatecsservice8)

**Implemented by:** [Implementation.ECS.Service.12](#implementationecsservice12), [Implementation.ECS.Service.13](#implementationecsservice13)

**Scoping** (for [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment)): Availability Zone rebalancing, which you select by setting `$.availabilityZoneRebalancing` to `ENABLED`, launches tasks for your Amazon ECS service in the Availability Zones with the fewest tasks and stops tasks in overloaded ones. Deploying an Amazon ECS service across more than one Availability Zone helps keep it available when one Availability Zone fails.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.availabilityZoneRebalancing`, `$.cluster`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.placementStrategy[*].field`, `$.placementStrategy[*].type`, `$.serviceName` |
| `ecs:UpdateService` | `$.availabilityZoneRebalancing`, `$.cluster`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.placementStrategy[*].field`, `$.placementStrategy[*].type`, `$.service` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Service.6

An Amazon ECS service's managed EBS volume and Service Connect TLS private key should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.ECS.Service.2](#threatecsservice2), [Threat.ECS.Service.4](#threatecsservice4), [Threat.ECS.Service.6](#threatecsservice6), [Threat.ECS.Service.14](#threatecsservice14)

**Implemented by:** [Implementation.ECS.Service.14](#implementationecsservice14), [Implementation.ECS.Service.15](#implementationecsservice15), [Implementation.ECS.Service.16](#implementationecsservice16)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): Amazon ECS can create and attach an Amazon EBS volume to each task in your service and store your Service Connect TLS private keys. Encrypting both with a customer managed key gives you control over the key policy and key lifecycle.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.serviceConnectConfiguration.services[*].tls.kmsKey`, `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |
| `ecs:UpdateService` | `$.serviceConnectConfiguration.services[*].tls.kmsKey`, `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.services[*]` |

### Mitigation.ECS.Service.7

An Amazon ECS service should launch its tasks in subnets in more than one Availability Zone

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.ECS.Service.8](#threatecsservice8)

**Implemented by:** [Implementation.ECS.Service.17](#implementationecsservice17), [Implementation.ECS.Service.18](#implementationecsservice18), [Implementation.ECS.Service.19](#implementationecsservice19)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): Amazon ECS launches each of your service's `awsvpc`-mode tasks with its own elastic network interface in a subnet from the `$.networkConfiguration.awsvpcConfiguration.subnets[*]` subnet list. Placing those network interfaces in more than one Availability Zone provides network-level redundancy and helps keep connectivity available when one zone is disrupted.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |
| `ecs:UpdateService` | `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Service.8

An Amazon ECS service should use Service Connect TLS to require encryption of data in transit using TLS 1.3

**Goal:** [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature)

**Addresses:** [Threat.ECS.Service.15](#threatecsservice15), [Threat.ECS.Service.18](#threatecsservice18)

**Implemented by:** [Implementation.ECS.Service.20](#implementationecsservice20), [Implementation.ECS.Service.21](#implementationecsservice21)

**Scoping** (for [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature)): For each Service Connect endpoint of your Amazon ECS service, setting `$.serviceConnectConfiguration.services[*].tls.issuerCertificateAuthority.awsPcaAuthorityArn`, the signing AWS Private CA, makes the Service Connect proxies encrypt traffic with TLS 1.3 instead of forwarding it unencrypted. Choosing Service Connect TLS helps protect your service-to-service traffic in transit.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.serviceConnectConfiguration.enabled`, `$.serviceConnectConfiguration.services[*].tls.issuerCertificateAuthority.awsPcaAuthorityArn` |
| `ecs:UpdateService` | `$.serviceConnectConfiguration.enabled`, `$.serviceConnectConfiguration.services[*].tls.issuerCertificateAuthority.awsPcaAuthorityArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Service.9

An Amazon ECS service should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.Service.3](#threatecsservice3)

**Implemented by:** [Implementation.ECS.Service.22](#implementationecsservice22), [Implementation.ECS.Service.23](#implementationecsservice23), [Implementation.ECS.Service.24](#implementationecsservice24)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag an Amazon ECS service through the list of `$.tags[*].key` and `$.tags[*].value` pairs on `CreateService` or by calling `TagResource`, separately from its cluster's tags. Owner and purpose tags help track ownership and the business function each service supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.serviceName`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServices` | `$.cluster`, `$.include[*]`, `$.services[*]` |
| `ecs:ListTagsForResource` | `$.resourceArn` |

### Mitigation.ECS.Service.10

An Amazon ECS service should have high-resolution service metrics enabled to collect detailed CloudWatch metrics

**Goal:** [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)

**Addresses:** [Threat.ECS.Service.1](#threatecsservice1), [Threat.ECS.Service.5](#threatecsservice5)

**Implemented by:** [Implementation.ECS.Service.25](#implementationecsservice25), [Implementation.ECS.Service.26](#implementationecsservice26), [Implementation.ECS.Service.27](#implementationecsservice27)

**Scoping** (for [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)): High-resolution service metrics, enabled through the `$.monitoring.metricConfigurations[*].resolutionSeconds` collection resolution setting, collect your service's `CPUUtilization` and `MemoryUtilization` CloudWatch metrics at a 20-second resolution instead of the default 60 seconds. Collecting those metrics improves security visibility by providing finer-grained data for anomaly detection and incident response.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.monitoring.metricConfigurations[*].metricNames[*]`, `$.monitoring.metricConfigurations[*].resolutionSeconds` |
| `ecs:UpdateService` | `$.monitoring.metricConfigurations[*].metricNames[*]`, `$.monitoring.metricConfigurations[*].resolutionSeconds` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Service.11

An Amazon ECS service should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.ECS.Service.7](#threatecsservice7), [Threat.ECS.Service.9](#threatecsservice9), [Threat.ECS.Service.10](#threatecsservice10), [Threat.ECS.Service.17](#threatecsservice17)

**Implemented by:** [Implementation.ECS.Service.28](#implementationecsservice28), [Implementation.ECS.Service.29](#implementationecsservice29), [Implementation.ECS.Service.30](#implementationecsservice30)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): When your service uses `awsvpc` network mode, you can supply security groups through the `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]` request parameter to control inbound and outbound traffic for each task's elastic network interface, and the VPC's default security group is used when you supply none. Restricting the inbound and outbound rules to the required sources, destinations, protocols, and ports helps reduce the network exposure of your Amazon ECS service.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateService` | `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]` |
| `ecs:UpdateService` | `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeServiceRevisions` | `$.serviceRevisionArns[*]` |
| `ecs:DescribeServices` | `$.cluster`, `$.services[*]` |

### Mitigation.ECS.Task.1

An Amazon ECS task's task IAM role and task execution IAM role should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.ECS.Task.11](#threatecstask11)

**Implemented by:** [Implementation.ECS.Task.1](#implementationecstask1)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): A task takes its roles from its task definition unless request parameters on `RunTask` or `StartTask` override them: `$.overrides.taskRoleArn` designates the task IAM role, whose temporary credentials your containers use to call AWS services, and `$.overrides.executionRoleArn` designates the task execution IAM role, whose temporary credentials the Amazon ECS container agent or Fargate agent uses to call AWS services. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.overrides.executionRoleArn`, `$.overrides.taskRoleArn` |
| `ecs:StartTask` | `$.overrides.executionRoleArn`, `$.overrides.taskRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTasks` | `$.cluster`, `$.tasks[*]` |

### Mitigation.ECS.Task.2

An Amazon ECS task's task role, execution role, and infrastructure role should be assumed only by Amazon ECS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.ECS.Task.10](#threatecstask10)

**Implemented by:** [Implementation.ECS.Task.2](#implementationecstask2), [Implementation.ECS.Task.3](#implementationecstask3), [Implementation.ECS.Task.4](#implementationecstask4), [Implementation.ECS.Task.5](#implementationecstask5)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): When you call `RunTask` or `StartTask`, you can pass the task role, task execution role, and infrastructure role in the `$.overrides.taskRoleArn`, `$.overrides.executionRoleArn`, and `$.volumeConfigurations[*].managedEBSVolume.roleArn` request parameters, and each role's trust policy specifies which principals can assume it. Allowing only Amazon ECS, acting on behalf of trusted resources, to assume those roles helps prevent an unintended principal from assuming them.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.overrides.executionRoleArn`, `$.overrides.taskRoleArn`, `$.volumeConfigurations[*].managedEBSVolume.roleArn` |
| `ecs:StartTask` | `$.overrides.executionRoleArn`, `$.overrides.taskRoleArn`, `$.volumeConfigurations[*].managedEBSVolume.roleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTasks` | `$.cluster`, `$.tasks[*]` |

### Mitigation.ECS.Task.3

An Amazon ECS task's managed ebs volume should have encryption at rest enabled

**Goal:** [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest)

**Addresses:** [Threat.ECS.Task.4](#threatecstask4)

**Implemented by:** [Implementation.ECS.Task.6](#implementationecstask6)

**Scoping** (for [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest)): When you launch a task that stores data on an Amazon EBS volume created by Amazon ECS, the optional `$.volumeConfigurations[*].managedEBSVolume.encrypted` request parameter, the volume's encryption setting, turns on Amazon EBS volume encryption. Enabling it at creation helps protect the task's data on the volume.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.cluster`, `$.taskDefinition`, `$.volumeConfigurations[*].managedEBSVolume.encrypted`, `$.volumeConfigurations[*].name` |
| `ecs:StartTask` | `$.cluster`, `$.taskDefinition`, `$.volumeConfigurations[*].managedEBSVolume.encrypted`, `$.volumeConfigurations[*].name` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTasks` | `$.cluster`, `$.tasks[*]` |

### Mitigation.ECS.Task.4

An Amazon ECS task should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.ECS.Task.5](#threatecstask5), [Threat.ECS.Task.9](#threatecstask9), [Threat.ECS.Task.13](#threatecstask13), [Threat.ECS.Task.14](#threatecstask14)

**Implemented by:** [Implementation.ECS.Task.7](#implementationecstask7)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): Under the `awsvpc` network mode, the required `$.networkConfiguration.awsvpcConfiguration.subnets[*]` request parameter selects the subnets that hold your task's own elastic network interface. Private subnet placement helps prevent direct internet routing to your task's elastic network interface, reducing its network exposure.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.networkConfiguration.awsvpcConfiguration.assignPublicIp`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |
| `ecs:StartTask` | `$.networkConfiguration.awsvpcConfiguration.assignPublicIp`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTasks` | `$.cluster`, `$.tasks[*]` |

### Mitigation.ECS.Task.5

An Amazon ECS task's managed EBS volume should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.ECS.Task.1](#threatecstask1), [Threat.ECS.Task.3](#threatecstask3), [Threat.ECS.Task.4](#threatecstask4)

**Implemented by:** [Implementation.ECS.Task.8](#implementationecstask8), [Implementation.ECS.Task.9](#implementationecstask9)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): Amazon ECS provisions and attaches an Amazon EBS volume to your task to hold data at rest. The volume configuration accepts a symmetric customer managed key, giving you lifecycle and policy control over that key.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |
| `ecs:StartTask` | `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTasks` | `$.cluster`, `$.tasks[*]` |
| `ecs:ListTasks` | `$.cluster` |

### Mitigation.ECS.Task.6

Amazon ECS tasks should be launched with subnets from more than one Availability Zone (AZ), so that task elastic network interfaces (ENIs) exist in more than one Availability Zone

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.ECS.Task.6](#threatecstask6)

**Implemented by:** [Implementation.ECS.Task.10](#implementationecstask10)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): In the `awsvpc` network mode, Amazon ECS creates each task's single elastic network interface in one of the subnets you supply through the `$.networkConfiguration.awsvpcConfiguration.subnets[*]` subnet list. Placing those network interfaces in more than one Availability Zone provides network-level redundancy and helps keep connectivity available when one zone is disrupted.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |
| `ecs:StartTask` | `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTasks` | `$.cluster`, `$.tasks[*]` |
| `ecs:ListTasks` | `$.cluster` |

### Mitigation.ECS.Task.7

An Amazon ECS task should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.Task.2](#threatecstask2)

**Implemented by:** [Implementation.ECS.Task.11](#implementationecstask11), [Implementation.ECS.Task.12](#implementationecstask12)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag an Amazon ECS task by setting `$.tags[*].key` and `$.tags[*].value`, a list of key/value pairs, on `RunTask` or `StartTask`, or afterwards by calling `TagResource` against the task ARN. Owner and purpose tags help track ownership and the business function each task supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.tags[*].key`, `$.tags[*].value` |
| `ecs:StartTask` | `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTasks` | `$.cluster`, `$.include[*]`, `$.tasks[*]` |
| `ecs:ListTagsForResource` | `$.resourceArn` |

### Mitigation.ECS.Task.8

An Amazon ECS task should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.ECS.Task.5](#threatecstask5), [Threat.ECS.Task.7](#threatecstask7), [Threat.ECS.Task.8](#threatecstask8), [Threat.ECS.Task.12](#threatecstask12)

**Implemented by:** [Implementation.ECS.Task.13](#implementationecstask13), [Implementation.ECS.Task.14](#implementationecstask14)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): When a task uses the `awsvpc` network mode, the security groups you attach through `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]` govern traffic on the task's elastic network interface, and the VPC's default security group applies when you specify none. Restricting the inbound and outbound rules to the required sources, destinations, protocols, and ports helps reduce the network exposure of your task.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RunTask` | `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]` |
| `ecs:StartTask` | `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTasks` | `$.cluster`, `$.tasks[*]` |

### Mitigation.ECS.TaskDefinition.1

An Amazon ECS task definition should be configured to send container logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.ECS.TaskDefinition.6](#threatecstaskdefinition6), [Threat.ECS.TaskDefinition.8](#threatecstaskdefinition8)

**Implemented by:** [Implementation.ECS.TaskDefinition.1](#implementationecstaskdefinition1), [Implementation.ECS.TaskDefinition.2](#implementationecstaskdefinition2), [Implementation.ECS.TaskDefinition.3](#implementationecstaskdefinition3), [ManagedControl.ECS.TaskDefinition.1](#managedcontrolecstaskdefinition1), [ManagedControl.ECS.TaskDefinition.2](#managedcontrolecstaskdefinition2)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): Each container in an Amazon ECS task definition records its `STDOUT` and `STDERR` streams as container logs, and its log configuration controls their delivery through the `awslogs` log driver. Sending those logs to CloudWatch Logs provides security-relevant visibility for audit trails, access monitoring, or error detection.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.containerDefinitions[*].logConfiguration.logDriver`, `$.containerDefinitions[*].logConfiguration.options{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskDefinition` | `$.taskDefinition` |

### Mitigation.ECS.TaskDefinition.2

An Amazon ECS task definition's task IAM role and task execution role should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.ECS.TaskDefinition.5](#threatecstaskdefinition5)

**Implemented by:** [Implementation.ECS.TaskDefinition.4](#implementationecstaskdefinition4)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): In an Amazon ECS task definition, the `$.taskRoleArn` request parameter designates the task IAM role, whose credentials your containers use to call AWS services, and the `$.executionRoleArn` request parameter designates the task execution role, whose credentials the Amazon ECS container and Fargate agents running your tasks use to call AWS services. Restricting those calls to expected networks enforces the network perimeter on your identities and helps prevent the use of those credentials from unexpected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.executionRoleArn`, `$.taskRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskDefinition` | `$.taskDefinition` |

### Mitigation.ECS.TaskDefinition.3

An Amazon ECS task definition's task role and task execution role should be assumed only by Amazon ECS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.ECS.TaskDefinition.4](#threatecstaskdefinition4)

**Implemented by:** [Implementation.ECS.TaskDefinition.5](#implementationecstaskdefinition5), [Implementation.ECS.TaskDefinition.6](#implementationecstaskdefinition6), [Implementation.ECS.TaskDefinition.7](#implementationecstaskdefinition7), [Implementation.ECS.TaskDefinition.8](#implementationecstaskdefinition8)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): Your task definition designates a task execution role through the `$.executionRoleArn` request parameter and a task role through the `$.taskRoleArn` request parameter, and each role's trust policy specifies which principals can assume it. Allowing only Amazon ECS, acting on behalf of trusted resources, to assume those roles helps prevent an unintended principal from assuming them.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.executionRoleArn`, `$.taskRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskDefinition` | `$.taskDefinition` |

### Mitigation.ECS.TaskDefinition.4

An Amazon ECS task definition should have an AWS Distro for OpenTelemetry (ADOT) sidecar container for metric collection enabled to collect detailed non-CloudWatch metrics

**Goal:** [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics)

**Addresses:** [Threat.ECS.TaskDefinition.1](#threatecstaskdefinition1), [Threat.ECS.TaskDefinition.3](#threatecstaskdefinition3)

**Implemented by:** [Implementation.ECS.TaskDefinition.9](#implementationecstaskdefinition9), [Implementation.ECS.TaskDefinition.10](#implementationecstaskdefinition10), [Implementation.ECS.TaskDefinition.11](#implementationecstaskdefinition11)

**Scoping** (for [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics)): An Amazon ECS task definition can declare an AWS Distro for OpenTelemetry (ADOT) sidecar container for metric collection alongside your application containers. This feature captures detailed operational telemetry, such as task-level CPU, memory, network, and storage metrics and custom application metrics, kept outside Amazon CloudWatch Metrics.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.containerDefinitions[*].command[*]`, `$.containerDefinitions[*].environment[*].name`, `$.containerDefinitions[*].environment[*].value`, `$.containerDefinitions[*].image`, `$.containerDefinitions[*].name`, `$.family`, `$.taskRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskDefinition` | `$.taskDefinition` |

### Mitigation.ECS.TaskDefinition.5

An Amazon ECS task definition's Amazon EFS volume should use transit encryption to require encryption of data in transit

**Goal:** [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature)

**Addresses:** [Threat.ECS.TaskDefinition.7](#threatecstaskdefinition7), [Threat.ECS.TaskDefinition.9](#threatecstaskdefinition9)

**Implemented by:** [Implementation.ECS.TaskDefinition.12](#implementationecstaskdefinition12), [Implementation.ECS.TaskDefinition.13](#implementationecstaskdefinition13), [ManagedControl.ECS.TaskDefinition.3](#managedcontrolecstaskdefinition3)

**Scoping** (for [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature)): Your Amazon ECS task definition's Amazon EFS volume carries a transit encryption setting that determines whether the mount between the Amazon ECS host and your Amazon EFS file system runs without TLS (`DISABLED`) or over TLS (`ENABLED`). Choosing transit encryption helps protect your Amazon EFS data in transit, because the setting defaults to `DISABLED` when omitted.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.volumes[*].efsVolumeConfiguration.transitEncryption` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskDefinition` | `$.taskDefinition` |

### Mitigation.ECS.TaskDefinition.6

An Amazon ECS task definition should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.TaskDefinition.2](#threatecstaskdefinition2)

**Implemented by:** [Implementation.ECS.TaskDefinition.14](#implementationecstaskdefinition14), [Implementation.ECS.TaskDefinition.15](#implementationecstaskdefinition15), [Implementation.ECS.TaskDefinition.16](#implementationecstaskdefinition16)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag a task definition revision through the `RegisterTaskDefinition` request parameters `$.tags[*].key` and `$.tags[*].value`, a list of key/value pairs, or later by calling `TagResource`. Owner and purpose tags help track ownership and the business function each task definition supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:RegisterTaskDefinition` | `$.family`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskDefinition` | `$.include[*]`, `$.taskDefinition` |
| `ecs:ListTagsForResource` | `$.resourceArn` |

### Mitigation.ECS.TaskSet.1

An Amazon ECS task set's infrastructure role for load balancers should be assumed only by Amazon ECS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.ECS.TaskSet.6](#threatecstaskset6)

**Implemented by:** [Implementation.ECS.TaskSet.1](#implementationecstaskset1), [Implementation.ECS.TaskSet.2](#implementationecstaskset2), [Implementation.ECS.TaskSet.3](#implementationecstaskset3)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): You can pass the infrastructure role for load balancers through the `$.loadBalancers[*].advancedConfiguration.roleArn` request parameter on `CreateTaskSet`, and the role's trust policy specifies which principals can assume it. Allowing only Amazon ECS, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateTaskSet` | `$.loadBalancers[*].advancedConfiguration.roleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskSets` | `$.cluster`, `$.service`, `$.taskSets[*]` |

### Mitigation.ECS.TaskSet.2

An Amazon ECS task set should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.ECS.TaskSet.5](#threatecstaskset5), [Threat.ECS.TaskSet.8](#threatecstaskset8), [Threat.ECS.TaskSet.9](#threatecstaskset9)

**Implemented by:** [Implementation.ECS.TaskSet.4](#implementationecstaskset4), [Implementation.ECS.TaskSet.5](#implementationecstaskset5)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): The `$.networkConfiguration.awsvpcConfiguration.subnets[*]` request parameter, required in your task set's network configuration, sets the task networking subnets that hold each task's elastic network interface. Private subnet placement helps prevent direct internet routing to those elastic network interfaces, reducing their network exposure.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateTaskSet` | `$.networkConfiguration.awsvpcConfiguration.assignPublicIp`, `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskSets` | `$.cluster`, `$.service`, `$.taskSets[*]` |

### Mitigation.ECS.TaskSet.3

An Amazon ECS task set should launch its tasks in subnets in more than one Availability Zone

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.ECS.TaskSet.2](#threatecstaskset2)

**Implemented by:** [Implementation.ECS.TaskSet.6](#implementationecstaskset6), [Implementation.ECS.TaskSet.7](#implementationecstaskset7), [Implementation.ECS.TaskSet.8](#implementationecstaskset8)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): Amazon ECS launches each task in your task set with its own elastic network interface in a subnet from the `$.networkConfiguration.awsvpcConfiguration.subnets[*]` subnet list. Placing those network interfaces in more than one Availability Zone provides network-level redundancy and helps keep connectivity available when one zone is disrupted.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateTaskSet` | `$.networkConfiguration.awsvpcConfiguration.subnets[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskSets` | `$.cluster`, `$.service`, `$.taskSets[*]` |

### Mitigation.ECS.TaskSet.4

An Amazon ECS task set should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.ECS.TaskSet.1](#threatecstaskset1)

**Implemented by:** [Implementation.ECS.TaskSet.9](#implementationecstaskset9), [Implementation.ECS.TaskSet.10](#implementationecstaskset10), [Implementation.ECS.TaskSet.11](#implementationecstaskset11)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You tag your task set through the `$.tags[*].key` and `$.tags[*].value` request parameters on `CreateTaskSet`, which carry a list of key/value pairs, or afterwards through `TagResource`, separately from its parent service's tags. Owner and purpose tags help track ownership and the business function each task set supports.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateTaskSet` | `$.cluster`, `$.service`, `$.tags[*].key`, `$.tags[*].value` |
| `ecs:TagResource` | `$.resourceArn`, `$.tags[*].key`, `$.tags[*].value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskSets` | `$.cluster`, `$.include[*]`, `$.service`, `$.taskSets[*]` |
| `ecs:ListTagsForResource` | `$.resourceArn` |

### Mitigation.ECS.TaskSet.5

An Amazon ECS task set should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.ECS.TaskSet.3](#threatecstaskset3), [Threat.ECS.TaskSet.4](#threatecstaskset4), [Threat.ECS.TaskSet.7](#threatecstaskset7)

**Implemented by:** [Implementation.ECS.TaskSet.12](#implementationecstaskset12), [Implementation.ECS.TaskSet.13](#implementationecstaskset13), [Implementation.ECS.TaskSet.14](#implementationecstaskset14)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): Through `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]`, your task set specifies the security groups attached to the dedicated task network interface in `awsvpc` network mode, or uses the VPC's default security group when you specify none. Restricting the inbound and outbound rules to the required sources, destinations, protocols, and ports helps reduce the network exposure of your task set.

**Configure via**

| Operation | Parameters |
|---|---|
| `ecs:CreateTaskSet` | `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ecs:DescribeTaskSets` | `$.cluster`, `$.service`, `$.taskSets[*]` |

---

## Implementations

### Implementation.ECS.1

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.ECS.1](#mitigationecs1)

**Scoping** (for [Mitigation.ECS.1](#mitigationecs1)): This implementation guidance recommends using your VPC endpoint policy to govern Amazon ECS API calls made through the endpoint with the `aws:ResourceOrgID` condition key. As a result, calls to Amazon ECS resources outside your organization are restricted, while AWS services acting on your behalf (`aws:PrincipalIsAWSService`) remain exempt.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.ECS.2

*CloudTrail network activity event — Detective*

Monitors the API calls below in CloudTrail.

- `ecs:DescribeTaskDefinition`
- `ecs:ExecuteCommand`
- `ecs:PutAttributes`
- `ecs:RegisterTaskDefinition`
- `ecs:RunTask`

**Implements:** [Mitigation.ECS.1](#mitigationecs1)

**Scoping** (for [Mitigation.ECS.1](#mitigationecs1)): This implementation guidance recommends recording Amazon ECS API calls made through your VPC endpoints, which are logged with an `eventSource` of `ecs.amazonaws.com`. Each event shows the account and principal that made the call, in `userIdentity`, and which operation they called. A call that violates your VPC endpoint policy carries an `errorCode` of `VpceAccessDenied`, helping you detect attempts to reach untrusted resources.

### Implementation.ECS.3

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.ECS.2](#mitigationecs2)

**Scoping** (for [Mitigation.ECS.2](#mitigationecs2)): This implementation guidance recommends governing Amazon ECS API calls with the `aws:PrincipalOrgID` condition key in your VPC endpoint policy, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, identities outside your organization are restricted from making Amazon ECS API calls through your network.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.ECS.4

*CloudTrail network activity event — Detective*

Monitors the API calls below in CloudTrail.

- `ecs:ExecuteCommand`
- `ecs:RegisterTaskDefinition`
- `ecs:RunTask`

**Implements:** [Mitigation.ECS.2](#mitigationecs2)

**Scoping** (for [Mitigation.ECS.2](#mitigationecs2)): This implementation guidance recommends recording Amazon ECS API calls made through your VPC endpoints, which are logged with an `eventSource` of `ecs.amazonaws.com`. Each event shows the account and principal that made the call, in `userIdentity`, and which operation they called, helping you detect calls made by untrusted identities. A call that violates your VPC endpoint policy carries an `errorCode` of `VpceAccessDenied`, which you can filter on.

### Implementation.ECS.AccountSetting.1

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `ecs:DeleteAccountSetting` — `$.name` = `containerInsights`
- `ecs:PutAccountSetting` — `$.name` = `containerInsights`, `$.value`
- `ecs:PutAccountSettingDefault` — `$.name` = `containerInsights`, `$.value`

**Implements:** [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1)

**Scoping** (for [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1)): This implementation guidance recommends recording `DeleteAccountSetting`, `PutAccountSetting`, and `PutAccountSettingDefault` calls carrying `$.name`, the `containerInsights` account setting, and, on the two put calls, `$.value`, the new value written to it. Watching those calls helps address the mitigation by surfacing attempts to stop the task and container metrics that Container Insights with enhanced observability collects from your new clusters.

### Implementation.ECS.AccountSetting.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListAccountSettings` with `$.name` = `containerInsights`, `$.effectiveSettings` = `true`; check `$.settings[*].name` = `containerInsights`, `$.settings[*].value` = `enhanced`, `$.settings[*].principalArn`

**Implements:** [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1)

**Scoping** (for [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1)): This implementation guidance recommends reading `$.settings[*].name`, `$.settings[*].value`, and `$.settings[*].principalArn`, the name, effective value, and principal ARN of the `containerInsights` account setting that `ListAccountSettings` returns. Flagging any Amazon ECS account setting named `containerInsights` whose effective value is not `enhanced` helps address the mitigation by identifying where the additional task and container metrics of Container Insights with enhanced observability are not collected by default for new clusters.

### Implementation.ECS.CapacityProvider.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalArn`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService`, `ec2:SourceInstanceARN` (alternatives: `aws:PrincipalTag/dp:include:network`) |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1)

**Scoping** (for [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1)): This implementation guidance recommends applying the `ec2:SourceInstanceARN`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon ECS capacity provider's Managed Instances instance profile credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `ec2:SourceInstanceARN` to apply the restriction to the role sessions whose credentials can be used outside your expected networks. Alternatively, you can use `aws:PrincipalTag/tag-key`, tagging the roles with the tag key and value your IAM policy management strategy uses. For credentials delivered to an Amazon EC2 instance, you can use `aws:Ec2InstanceSourceVpc` and `aws:Ec2InstanceSourcePrivateIPv4` to bind them to the instance they were issued to. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy for Amazon EC2](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/network_perimeter_ec2_scp.json)
- [How to use policies to restrict where EC2 instance credentials can be used from](https://aws.amazon.com/blogs/security/how-to-use-policies-to-restrict-where-ec2-instance-credentials-can-be-used-from/)

### Implementation.ECS.CapacityProvider.2

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::CapacityProvider`: `$.ManagedInstancesProvider.InfrastructureRoleArn`, `$.ManagedInstancesProvider.InstanceLaunchTemplate.Ec2InstanceProfileArn`

**Implements:** [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)

**Scoping** (for [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)): This implementation guidance recommends validating the trust policies of `$.ManagedInstancesProvider.InfrastructureRoleArn` and `$.ManagedInstancesProvider.InstanceLaunchTemplate.Ec2InstanceProfileArn`, your capacity provider's infrastructure role and instance profile role, before provisioning. This helps prevent the capacity provider from being provisioned when any trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.CapacityProvider.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:DescribeCapacityProviders` with `$.cluster`; check `$.capacityProviders[*].type` = `MANAGED_INSTANCES`, `$.capacityProviders[*].managedInstancesProvider.infrastructureRoleArn`, `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn`
3. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
4. Call `iam:GetInstanceProfile` with `$.InstanceProfileName`; check `$.InstanceProfile.Roles[*].AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)

**Scoping** (for [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)): This implementation guidance recommends reading `$.capacityProviders[*].managedInstancesProvider.infrastructureRoleArn` and `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn`, your capacity provider's infrastructure role and instance profile role, then evaluating `$.Role.AssumeRolePolicyDocument` and `$.InstanceProfile.Roles[*].AssumeRolePolicyDocument`, each role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.CapacityProvider.4

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)

**Scoping** (for [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS capacity provider's infrastructure role, `$.managedInstancesProvider.infrastructureRoleArn`, to the `ecs.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.ECS.CapacityProvider.5

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ec2.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)

**Scoping** (for [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS Managed Instances capacity provider's instance profile role, `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn`, to the `ec2.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.ECS.CapacityProvider.6

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)

**Scoping** (for [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon ECS capacity provider's infrastructure role and instance profile role, `$.managedInstancesProvider.infrastructureRoleArn` and `$.managedInstancesProvider.instanceLaunchTemplate.ec2InstanceProfileArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.ECS.CapacityProvider.7

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::CapacityProvider`: `$.ManagedInstancesProvider.InstanceLaunchTemplate.NetworkConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3)

**Scoping** (for [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3)): This implementation guidance recommends validating `$.ManagedInstancesProvider.InstanceLaunchTemplate.NetworkConfiguration.Subnets[*]`, the subnets where Amazon ECS launches managed instances, by resolving each subnet's route table for a default route to an internet gateway before provisioning. Subnets carrying that route fail validation, which helps prevent an Amazon ECS capacity provider's managed instance from being provisioned in a public subnet.

### Implementation.ECS.CapacityProvider.8

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:DescribeCapacityProviders` with `$.cluster`; check `$.capacityProviders[*].type`, `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]`
3. Call `ec2:DescribeRouteTables` with `$.Filters` = `association.subnet-id`, `$.Filters` = `association.main`; check `$.RouteTables[*].Routes[*].GatewayId`, `$.RouteTables[*].Routes[*].DestinationCidrBlock` = `0.0.0.0/0`, `$.RouteTables[*].Routes[*].DestinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3)

**Scoping** (for [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3)): This implementation guidance recommends reading `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]`, the subnets where your managed instances launch, then resolving each subnet's route table. Flagging a capacity provider with a subnet whose default route targets an internet gateway, read at `$.RouteTables[*].Routes[*].GatewayId`, helps you detect placement in a public subnet.

### Implementation.ECS.CapacityProvider.9

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::CapacityProvider`: `$.ManagedInstancesProvider.InstanceLaunchTemplate.InstanceRequirements.AllowedInstanceTypes[*]`, `$.ManagedInstancesProvider.InstanceLaunchTemplate.InstanceRequirements.ExcludedInstanceTypes[*]`

**Implements:** [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4)

**Scoping** (for [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4)): This implementation guidance recommends validating `$.ManagedInstancesProvider.InstanceLaunchTemplate.InstanceRequirements.AllowedInstanceTypes[*]` and `$.ManagedInstancesProvider.InstanceLaunchTemplate.InstanceRequirements.ExcludedInstanceTypes[*]`, your capacity provider's allowed and excluded instance types, before provisioning, which helps prevent an Amazon ECS Managed Instances capacity provider permitting instance types without automatic instance-to-instance encryption from being provisioned. That helps provide an additional layer of encryption in transit between your managed instances without application changes.

### Implementation.ECS.CapacityProvider.10

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:DescribeCapacityProviders` with `$.cluster`; check `$.capacityProviders[*].name`, `$.capacityProviders[*].type`, `$.capacityProviders[*].status`, `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.AllowedInstanceType[*]`, `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.ExcludedInstanceType[*]`
3. Call `ec2:DescribeInstanceTypes` with `$.InstanceType[*]`, `$.Filter[*].Name`, `$.Filter[*].Value[*]`; check `$.instanceTypeSet[*].instanceType`, `$.instanceTypeSet[*].networkInfo.encryptionInTransitSupported`

**Implements:** [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4)

**Scoping** (for [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4)): This implementation guidance recommends comparing the allowed and excluded instance types of your Amazon ECS capacity provider at `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.AllowedInstanceType[*]` and `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.instanceRequirements.ExcludedInstanceType[*]` with EC2 encryption-in-transit support at `$.instanceTypeSet[*].networkInfo.encryptionInTransitSupported`. That comparison flags an Amazon ECS capacity provider that can launch instance types without automatic encryption of instance-to-instance traffic.

### Implementation.ECS.CapacityProvider.11

*Guidance — Directive*

Configure the subnet list in your Amazon ECS Managed Instances capacity provider's network configuration with two or more subnets. Choose subnets that resolve to different Availability Zones. The elastic network interfaces (ENIs) that Amazon ECS creates for the capacity provider then exist in more than one Availability Zone. Network connectivity can then survive the loss of a single zone.

Subnet-to-Availability-Zone membership is a property of each subnet, not of the capacity provider. Before you supply the subnet IDs, call `DescribeSubnets` and read the `AvailabilityZone` response field to confirm that they belong to distinct Availability Zones. Standardize on infrastructure-as-code review so the templates and modules that create your capacity providers supply at least two subnets from different Availability Zones. If your capacity provider instead points at an EC2 Auto Scaling group, configure the Availability Zone spread on that Auto Scaling group. That capacity provider shape carries no subnet configuration of its own.

To verify the deployed state, enumerate the network interfaces with `DescribeNetworkInterfaces`. The `DescribeCapacityProviders` response also echoes the subnets you configured. Confirm that the subnets of the attached ENIs span more than one Availability Zone.

AWS Resilience Hub can help you track multi-AZ posture at scale. For Amazon ECS workloads, its application assessments evaluate Amazon ECS services and Auto Scaling groups, not capacity providers. Among the capacity providers you create, a Resilience Hub assessment reflects multi-AZ posture only for those backed by an Auto Scaling group. For Managed Instances capacity providers, verify the subnet-to-Availability-Zone spread directly through `DescribeSubnets` and `DescribeNetworkInterfaces`, as the earlier steps describe.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5)

**Scoping** (for [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5)): This implementation guidance recommends supplying two or more subnets from different Availability Zones in `$.managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]`, the subnet list of your Managed Instances capacity provider. Spreading the primary network interfaces of the Managed Instances that Amazon ECS launches across zones helps their connectivity survive the loss of a single Availability Zone.

### Implementation.ECS.CapacityProvider.12

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::CapacityProvider`: `$.ManagedInstancesProvider.InstanceLaunchTemplate.NetworkConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5)

**Scoping** (for [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5)): This implementation guidance recommends validating, before provisioning, that `$.ManagedInstancesProvider.InstanceLaunchTemplate.NetworkConfiguration.Subnets[*]`, your Amazon ECS capacity provider's VPC subnets, lists two or more subnet entries. That check helps prevent a single-subnet capacity provider from being provisioned, so its Managed Instances' network interfaces can span more than one Availability Zone.

### Implementation.ECS.CapacityProvider.13

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:DescribeCapacityProviders` with `$.cluster`; check `$.capacityProviders[*].type`, `$.capacityProviders[*].status`, `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]`
3. Call `ec2:DescribeSubnets` with `$.SubnetId`; check `$.subnetSet[*].subnetId`, `$.subnetSet[*].availabilityZone`

**Implements:** [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5)

**Scoping** (for [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5)): This implementation guidance recommends reading `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.subnets[*]`, the subnets where Amazon ECS launches your Amazon ECS Managed Instances, then resolving each subnet's Availability Zone. Flagging a Managed Instances capacity provider confined to a single Availability Zone helps you detect the primary network interfaces of its instances that lack multi-zone redundancy.

### Implementation.ECS.CapacityProvider.14

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::CapacityProvider`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6)

**Scoping** (for [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon ECS capacity provider that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.ECS.CapacityProvider.15

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::CapacityProvider`: `$.tags[*].key`, `$.tags[*].value`, `$.configuration.Tags[*].Key`, `$.configuration.Tags[*].Value`

**Implements:** [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6)

**Scoping** (for [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6)): This implementation guidance recommends reading `$.tags[*].key` and `$.configuration.Tags[*].Key` on each Amazon ECS capacity provider and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS capacity provider to an accountable owner and the business function it supports.

### Implementation.ECS.CapacityProvider.16

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:CreateCapacityProvider` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:capacity-provider/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:capacity-provider/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6)

**Scoping** (for [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6)): This implementation guidance recommends restricting `ecs:CreateCapacityProvider` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS capacity provider without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.CapacityProvider.17

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::CapacityProvider`: `$.ManagedInstancesProvider.InstanceLaunchTemplate.Monitoring`

**Implements:** [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7)

**Scoping** (for [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7)): This implementation guidance recommends validating `$.ManagedInstancesProvider.InstanceLaunchTemplate.Monitoring`, the CloudWatch monitoring option for your Amazon ECS Managed Instances, set to `DETAILED` before provisioning. That validation helps address the mitigation by preventing an Amazon ECS capacity provider from being provisioned without detailed CloudWatch metrics.

### Implementation.ECS.CapacityProvider.18

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `ec2:UnmonitorInstances` — `$.InstanceId[*]`
- `ecs:CreateCapacityProvider` — `$.cluster`, `$.managedInstancesProvider.instanceLaunchTemplate.monitoring`, `$.name`
- `ecs:UpdateCapacityProvider` — `$.cluster`, `$.managedInstancesProvider.instanceLaunchTemplate.monitoring` = `BASIC`, `$.name`

**Implements:** [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7)

**Scoping** (for [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7)): This implementation guidance recommends recording `UpdateCapacityProvider`, `UnmonitorInstances`, and `CreateCapacityProvider` calls carrying `$.managedInstancesProvider.instanceLaunchTemplate.monitoring`, your capacity provider's monitoring option, or `$.InstanceId[*]`, the instances whose detailed monitoring is disabled. Watching those calls helps address the mitigation by surfacing attempts to stop the 1-minute CloudWatch metrics that detailed monitoring provides for your managed instances.

### Implementation.ECS.CapacityProvider.19

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:DescribeCapacityProviders` with `$.cluster`; check `$.capacityProviders[*].type` = `MANAGED_INSTANCES`, `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.monitoring` = `DETAILED`

**Implements:** [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7)

**Scoping** (for [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7)): This implementation guidance recommends reading `$.capacityProviders[*].type`, the capacity provider type, and `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.monitoring`, the monitoring option for the managed instances the capacity provider launches. Flagging any Amazon ECS capacity provider that uses basic monitoring for Managed Instances helps address the mitigation by identifying where 1-minute managed instance metrics other than status checks are not collected.

### Implementation.ECS.CapacityProvider.20

*Guidance — Directive*

Configure the security groups on your Amazon ECS capacity provider network configuration following least-privilege best practices. These security groups control the network traffic allowed to and from the Amazon ECS Managed Instances that the capacity provider launches. Restrict both inbound and outbound rules to only the required sources and destinations (IP address ranges, referenced security groups, or prefix lists), protocols, and ports. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change.

To govern security groups at scale, use AWS Firewall Manager security group policies across an AWS Organization. You can scope each policy by account, organizational unit, or tag. A common policy applies a baseline security group. A content audit policy audits inbound and outbound rules against an allow-list or a block-list. It flags non-compliant groups in a central dashboard and sends notifications. It can also remediate those rules automatically. A usage audit policy removes unused or redundant security groups.

To verify unintended access permitted by your security groups, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended. For example, match any path that reaches administrative ports 22 or 3389 from an unrestricted address. Then run an analysis with `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`, and review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8)

**Scoping** (for [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8)): This implementation guidance recommends restricting the inbound and outbound rules on your Amazon ECS capacity provider's security groups to the required addresses, protocols, and ports. Excluding unrestricted addresses such as 0.0.0.0/0 or ::/0 from those inbound and outbound rules helps reduce the network exposure of your Amazon ECS Managed Instances.

### Implementation.ECS.CapacityProvider.21

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::CapacityProvider`: `$.ManagedInstancesProvider.InstanceLaunchTemplate.NetworkConfiguration.SecurityGroups[*]`

**Implements:** [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8)

**Scoping** (for [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8)): This implementation guidance recommends validating the inbound and outbound rules of the security groups resolved from `$.ManagedInstancesProvider.InstanceLaunchTemplate.NetworkConfiguration.SecurityGroups[*]` before provisioning. Checking for rules that allow any address on unintended ports, directly or through a prefix list, helps prevent an Amazon ECS capacity provider from being provisioned with such groups.

### Implementation.ECS.CapacityProvider.22

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:DescribeCapacityProviders` with `$.cluster`; check `$.capacityProviders[*].name`, `$.capacityProviders[*].type`, `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.securityGroups[*]`
3. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
4. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8)

**Scoping** (for [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8)): This implementation guidance recommends resolving the security groups recorded at `$.capacityProviders[*].managedInstancesProvider.instanceLaunchTemplate.networkConfiguration.securityGroups[*]` and inspecting their inbound and outbound rules. Flagging rules that admit 0.0.0.0/0 or ::/0 on sensitive ports, broad port ranges, or all protocols helps you detect your Amazon ECS Managed Instances with broader network exposure than they need.

### Implementation.ECS.Cluster.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Cluster`: `$.Configuration.ExecuteCommandConfiguration.Logging`, `$.Configuration.ExecuteCommandConfiguration.LogConfiguration.CloudWatchLogGroupName`

**Implements:** [Mitigation.ECS.Cluster.1](#mitigationecscluster1)

**Scoping** (for [Mitigation.ECS.Cluster.1](#mitigationecscluster1)): This implementation guidance recommends validating that `$.Configuration.ExecuteCommandConfiguration.Logging`, the cluster's ECS Exec log setting, is set to `OVERRIDE` and that `$.Configuration.ExecuteCommandConfiguration.LogConfiguration.CloudWatchLogGroupName`, the name of the CloudWatch log group that receives session output, is not empty before provisioning. That check helps prevent an Amazon ECS cluster from being provisioned without configured ECS Exec session logging to CloudWatch Logs, where you can review commands run inside your containers.

### Implementation.ECS.Cluster.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`, `$.logStreamName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `ecs:CreateCluster` — `$.clusterName`, `$.configuration.executeCommandConfiguration.logConfiguration.cloudWatchLogGroupName`, `$.configuration.executeCommandConfiguration.logging`
- `ecs:ExecuteCommand` — `$.cluster`, `$.command`, `$.container`, `$.interactive`, `$.task`
- `ecs:UpdateCluster` — `$.cluster`, `$.configuration.executeCommandConfiguration.logConfiguration.cloudWatchLogGroupName`, `$.configuration.executeCommandConfiguration.logging`
- `ssm:StartSession` — `$.Target`

**Implements:** [Mitigation.ECS.Cluster.1](#mitigationecscluster1)

**Scoping** (for [Mitigation.ECS.Cluster.1](#mitigationecscluster1)): This implementation guidance recommends recording `UpdateCluster` calls that set `$.configuration.executeCommandConfiguration.logging`, the cluster's ECS Exec log setting, and `$.configuration.executeCommandConfiguration.logConfiguration.cloudWatchLogGroupName`, the log group for session logs. Recording these calls alongside `DeleteLogGroup`, `DeleteLogStream`, and `PutRetentionPolicy` helps you detect changes that disable or weaken ECS Exec session logging or erase its log records. Recording `CreateCluster` as well lets you detect a cluster created without this logging.

### Implementation.ECS.Cluster.3

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::Cluster`: `$.configuration.Configuration.ExecuteCommandConfiguration.Logging`, `$.configuration.Configuration.ExecuteCommandConfiguration.LogConfiguration.CloudWatchLogGroupName`

**Implements:** [Mitigation.ECS.Cluster.1](#mitigationecscluster1)

**Scoping** (for [Mitigation.ECS.Cluster.1](#mitigationecscluster1)): This implementation guidance recommends reading each cluster's `$.configuration.Configuration.ExecuteCommandConfiguration.Logging`, the ECS Exec log setting, and `$.configuration.Configuration.ExecuteCommandConfiguration.LogConfiguration.CloudWatchLogGroupName`, the CloudWatch log group to send session logs to. Flagging clusters that lack the `OVERRIDE` log setting or a CloudWatch log group name helps you detect gaps in ECS Exec session logging to CloudWatch Logs.

### Implementation.ECS.Cluster.4

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Cluster`: `$.Configuration.ManagedStorageConfiguration.KmsKeyId`, `$.Configuration.ManagedStorageConfiguration.FargateEphemeralStorageKmsKeyId`

**Implements:** [Mitigation.ECS.Cluster.2](#mitigationecscluster2)

**Scoping** (for [Mitigation.ECS.Cluster.2](#mitigationecscluster2)): This implementation guidance recommends validating the keys for the cluster's managed storage (`$.Configuration.ManagedStorageConfiguration.KmsKeyId` for Amazon EBS volumes and `$.Configuration.ManagedStorageConfiguration.FargateEphemeralStorageKmsKeyId` for Fargate ephemeral storage) before provisioning. Requiring a customer managed key in your account, with key material that is neither imported nor held in an external key store, helps prevent an Amazon ECS cluster from being provisioned without a key whose policy, lifecycle, and key material you control.

### Implementation.ECS.Cluster.5

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:DescribeClusters` with `$.clusters[*]`, `$.include[*]` = `CONFIGURATIONS`; check `$.clusters[*].configuration.managedStorageConfiguration.kmsKeyId`, `$.clusters[*].configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`
3. Call `kms:DescribeKey` with `$.KeyId`; check `$.KeyMetadata.KeyManager` = `CUSTOMER`, `$.KeyMetadata.AWSAccountId`, `$.KeyMetadata.Origin`

**Implements:** [Mitigation.ECS.Cluster.2](#mitigationecscluster2)

**Scoping** (for [Mitigation.ECS.Cluster.2](#mitigationecscluster2)): This implementation guidance recommends reading `$.clusters[*].configuration.managedStorageConfiguration.kmsKeyId` and `$.clusters[*].configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, the keys that encrypt Amazon ECS managed storage and Fargate ephemeral storage for tasks in your cluster, and resolving `$.KeyMetadata.KeyManager`, `$.KeyMetadata.AWSAccountId`, and `$.KeyMetadata.Origin` for each key. Flagging Amazon ECS clusters whose key is not a customer managed key in your account, or whose key material is imported or held in an external key store, helps you detect where you do not control the key policy, lifecycle, or key material.

### Implementation.ECS.Cluster.6

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:CreateCluster`, `ecs:UpdateCluster` | `*` | `ecs:fargate-ephemeral-storage-kms-key` |

**Implements:** [Mitigation.ECS.Cluster.2](#mitigationecscluster2)

**Scoping** (for [Mitigation.ECS.Cluster.2](#mitigationecscluster2)): This implementation guidance recommends applying the `ecs:fargate-ephemeral-storage-kms-key` condition key in a service control policy (SCP) to govern `ecs:CreateCluster` and `ecs:UpdateCluster`. This condition helps prevent creating or modifying an Amazon ECS cluster without an approved customer managed key at `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, the key that encrypts the Fargate ephemeral storage of the cluster's tasks. No condition key in the policy governs `$.configuration.managedStorageConfiguration.kmsKeyId`, the cluster-level key for the Amazon EBS volumes that Amazon ECS attaches to the cluster's tasks.

### Implementation.ECS.Cluster.7

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Cluster`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.ECS.Cluster.3](#mitigationecscluster3)

**Scoping** (for [Mitigation.ECS.Cluster.3](#mitigationecscluster3)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon ECS cluster that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.ECS.Cluster.8

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::Cluster`: `$.configuration.Tags[*].Key`, `$.configuration.Tags[*].Value`, `$.tags[*].key`, `$.tags[*].value`

**Implements:** [Mitigation.ECS.Cluster.3](#mitigationecscluster3)

**Scoping** (for [Mitigation.ECS.Cluster.3](#mitigationecscluster3)): This implementation guidance recommends reading `$.configuration.Tags[*].Key` and `$.tags[*].key` on each Amazon ECS cluster and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS cluster to an accountable owner and the business function it supports.

### Implementation.ECS.Cluster.9

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:CreateCluster` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:cluster/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:cluster/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.Cluster.3](#mitigationecscluster3)

**Scoping** (for [Mitigation.ECS.Cluster.3](#mitigationecscluster3)): This implementation guidance recommends restricting `ecs:CreateCluster` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS cluster without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.Cluster.10

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Cluster`: `$.Configuration.ManagedStorageConfiguration.KmsKeyId`, `$.Configuration.ManagedStorageConfiguration.FargateEphemeralStorageKmsKeyId`

**Implements:** [Mitigation.ECS.Cluster.4](#mitigationecscluster4)

**Scoping** (for [Mitigation.ECS.Cluster.4](#mitigationecscluster4)): This implementation guidance recommends validating, before provisioning, that `$.Configuration.ManagedStorageConfiguration.KmsKeyId` (the cluster's managed storage key) and `$.Configuration.ManagedStorageConfiguration.FargateEphemeralStorageKmsKeyId` (the Fargate ephemeral storage key) name a customer managed key. That check helps prevent an Amazon ECS cluster from being provisioned without a customer managed default key for new tasks' Amazon EBS volumes and Fargate ephemeral storage.

### Implementation.ECS.Cluster.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:DescribeClusters` with `$.clusters[*]`, `$.include[*]` = `CONFIGURATIONS`; check `$.clusters[*].configuration.managedStorageConfiguration.kmsKeyId`, `$.clusters[*].configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`
3. Call `kms:DescribeKey` with `$.KeyId`; check `$.KeyMetadata.KeyManager` = `CUSTOMER`, `$.KeyMetadata.Arn`

**Implements:** [Mitigation.ECS.Cluster.4](#mitigationecscluster4)

**Scoping** (for [Mitigation.ECS.Cluster.4](#mitigationecscluster4)): This implementation guidance recommends reading each Amazon ECS cluster's `$.clusters[*].configuration.managedStorageConfiguration.kmsKeyId`, the key for task-attached Amazon EBS volumes, and `$.clusters[*].configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`, the key for AWS Fargate ephemeral storage. Flagging clusters missing a customer managed default key helps you detect new Amazon ECS managed storage that would be created without one.

### Implementation.ECS.Cluster.12

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:CreateCluster`, `ecs:UpdateCluster` | `*` | `aws:PrincipalArn` (alternatives: `aws:PrincipalTag/${TagKey}`), `ecs:fargate-ephemeral-storage-kms-key` |
| `ecs:UpdateCluster` | `*` | `aws:PrincipalArn` (alternatives: `aws:PrincipalTag/${TagKey}`) |

**Implements:** [Mitigation.ECS.Cluster.4](#mitigationecscluster4)

**Scoping** (for [Mitigation.ECS.Cluster.4](#mitigationecscluster4)): This implementation guidance recommends applying the `aws:PrincipalArn` condition key in a service control policy (SCP) to govern `ecs:CreateCluster` and `ecs:UpdateCluster`. This condition helps prevent principals other than those you designate from changing your cluster's managed storage configuration through `$.configuration.managedStorageConfiguration.kmsKeyId` and `$.configuration.managedStorageConfiguration.fargateEphemeralStorageKmsKeyId`. Applying `ecs:fargate-ephemeral-storage-kms-key` to `ecs:CreateCluster` and `ecs:UpdateCluster` also helps restrict those requests to ones that specify an AWS KMS key for Fargate ephemeral storage.

**Reference:** To restrict access to select principals, use the `aws:PrincipalArn` or `aws:PrincipalTag/tag-key` condition keys. Use `aws:PrincipalArn` to specify the Amazon Resource Names (ARNs) of the principals. Those ARNs can follow a naming convention or an IAM path that scopes the restriction to a set of principals without enumerating each one. Use `aws:PrincipalTag/tag-key` to identify those principals by a tag attached to them. Choose the condition key that matches your IAM policy management strategy.

### Implementation.ECS.Cluster.13

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Cluster`: `$.ClusterSettings[*].Name`, `$.ClusterSettings[*].Value`

**Implements:** [Mitigation.ECS.Cluster.5](#mitigationecscluster5)

**Scoping** (for [Mitigation.ECS.Cluster.5](#mitigationecscluster5)): This implementation guidance recommends validating `$.ClusterSettings[*].Name` and `$.ClusterSettings[*].Value`, the `containerInsights` cluster setting, set to `enhanced` before provisioning. That validation helps address the mitigation by preventing an Amazon ECS cluster from being provisioned without detailed CloudWatch metrics.

### Implementation.ECS.Cluster.14

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `ecs:PutAccountSetting` — `$.name` = `containerInsights`, `$.principalArn`, `$.value`
- `ecs:PutAccountSettingDefault` — `$.name` = `containerInsights`, `$.value`
- `ecs:UpdateCluster` — `$.cluster`, `$.settings[*].name` = `containerInsights`, `$.settings[*].value`
- `ecs:UpdateClusterSettings` — `$.cluster`, `$.settings[*].name` = `containerInsights`, `$.settings[*].value`

**Implements:** [Mitigation.ECS.Cluster.5](#mitigationecscluster5)

**Scoping** (for [Mitigation.ECS.Cluster.5](#mitigationecscluster5)): This implementation guidance recommends recording `UpdateClusterSettings`, `UpdateCluster`, `PutAccountSetting`, and `PutAccountSettingDefault` calls carrying `$.settings[*].name` and `$.settings[*].value`, the `containerInsights` cluster setting, or `$.name` and `$.value`, the `containerInsights` account setting. Watching those calls helps address the mitigation by surfacing attempts to stop the Container Insights with enhanced observability metrics for your clusters or to disable the account default new clusters inherit.

### Implementation.ECS.Cluster.15

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::Cluster`: `$.configuration.ClusterSettings[*].Name`, `$.configuration.ClusterSettings[*].Value`

**Implements:** [Mitigation.ECS.Cluster.5](#mitigationecscluster5)

**Scoping** (for [Mitigation.ECS.Cluster.5](#mitigationecscluster5)): This implementation guidance recommends reading `$.configuration.ClusterSettings[*].Name` and `$.configuration.ClusterSettings[*].Value`, the name and value of your cluster's Container Insights setting. Flagging any Amazon ECS cluster whose Container Insights setting is not `enhanced` helps address the mitigation by identifying where the additional task and container metrics of Container Insights with enhanced observability are not collected.

### Implementation.ECS.ContainerInstance.1

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListContainerInstances` with `$.cluster`; check `$.containerInstanceArns[*]`
3. Call `ecs:DescribeContainerInstances` with `$.cluster`, `$.containerInstances[*]`, `$.include[*]` = `TAGS`; check `$.containerInstances[*].containerInstanceArn`, `$.containerInstances[*].tags[*].key`, `$.containerInstances[*].tags[*].value`

**Implements:** [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1)

**Scoping** (for [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1)): This implementation guidance recommends reading `$.containerInstances[*].tags[*].key` on each Amazon ECS container instance listed through `$.containerInstanceArns[*]` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS container instance to an accountable owner and the business function it supports.

### Implementation.ECS.ContainerInstance.2

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:RegisterContainerInstance` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:container-instance/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:container-instance/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1)

**Scoping** (for [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1)): This implementation guidance recommends restricting `ecs:RegisterContainerInstance` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS container instance without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.Daemon.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Daemon`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1)

**Scoping** (for [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon ECS daemon that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.ECS.Daemon.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListDaemons` with `$.clusterArn`; check `$.daemonSummariesList[*].daemonArn`
3. Call `ecs:ListTagsForResource` with `$.resourceArn`; check `$.tags[*].key`, `$.tags[*].value`

**Implements:** [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1)

**Scoping** (for [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1)): This implementation guidance recommends reading `$.tags[*].key` on each Amazon ECS daemon listed through `$.daemonSummariesList[*].daemonArn` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS daemon to an accountable owner and the business function it supports.

### Implementation.ECS.Daemon.3

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:CreateDaemon` | `*` | `aws:RequestTag/owner` |
| `ecs:CreateDaemon` | `*` | `aws:RequestTag/purpose` |
| `ecs:CreateDaemon` | `*` | `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:daemon/*/*` | `aws:RequestTag/owner` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:daemon/*/*` | `aws:RequestTag/purpose` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:daemon/*/*` | `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:daemon/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1)

**Scoping** (for [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1)): This implementation guidance recommends restricting `ecs:CreateDaemon` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS daemon without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.DaemonTaskDefinition.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::DaemonTaskDefinition`: `$.ContainerDefinitions[*].LogConfiguration.LogDriver`, `$.ContainerDefinitions[*].LogConfiguration.Options{key}`, `$.ExecutionRoleArn`

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1)): This implementation guidance recommends validating that `$.ContainerDefinitions[*].LogConfiguration.LogDriver`, each daemon container's log driver, is set to `awslogs` and that `$.ContainerDefinitions[*].LogConfiguration.Options{key}`, its log driver options, include `awslogs-group` and `awslogs-region` before provisioning. That check helps prevent an Amazon ECS daemon task definition from being provisioned without configured container logging to CloudWatch Logs, which lets you review each daemon container's `STDOUT` and `STDERR` streams.

### Implementation.ECS.DaemonTaskDefinition.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `ecs:CreateDaemon` — `$.daemonTaskDefinitionArn`
- `ecs:RegisterDaemonTaskDefinition` — `$.containerDefinitions[*].logConfiguration.logDriver` = `awslogs`, `$.containerDefinitions[*].logConfiguration.options{key}`, `$.family`
- `ecs:UpdateDaemon` — `$.daemonTaskDefinitionArn`

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1)): This implementation guidance recommends recording `RegisterDaemonTaskDefinition` with `$.containerDefinitions[*].logConfiguration.logDriver`, each daemon container's log driver, and `$.containerDefinitions[*].logConfiguration.options{key}`, the log driver options such as the destination log group. Recording these calls alongside `DeleteLogGroup` and `PutRetentionPolicy` helps you detect changes that disable or weaken daemon container logging or erase its log records.

### Implementation.ECS.DaemonTaskDefinition.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListDaemonTaskDefinitions` with `$.status`; check `$.daemonTaskDefinitions[*].arn`, `$.daemonTaskDefinitions[*].status`
2. Call `ecs:DescribeDaemonTaskDefinition` with `$.daemonTaskDefinition`; check `$.daemonTaskDefinition.containerDefinitions[*].logConfiguration.logDriver` = `awslogs`, `$.daemonTaskDefinition.containerDefinitions[*].logConfiguration.options{key}`, `$.daemonTaskDefinition.executionRoleArn`

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1)): This implementation guidance recommends reading `$.daemonTaskDefinition.containerDefinitions[*].logConfiguration.logDriver`, each daemon container's log driver, `$.daemonTaskDefinition.containerDefinitions[*].logConfiguration.options{key}`, its `awslogs-group` and `awslogs-region` destination options, and `$.daemonTaskDefinition.executionRoleArn`, the task execution role required for sending container logs to CloudWatch Logs, on each revision that `ListDaemonTaskDefinitions` returns. Flagging daemon task definition revisions where a container's log driver is not `awslogs` or a destination option is missing, or where no task execution role is set, helps you detect gaps in container logging to CloudWatch Logs.

### Implementation.ECS.DaemonTaskDefinition.4

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalTag/dp:exclude:network`, `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.2](#mitigationecsdaemontaskdefinition2)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.2](#mitigationecsdaemontaskdefinition2)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon ECS daemon task definition's task IAM role and task execution role credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalTag/tag-key` to apply the restriction to principals whose credentials can be used outside your expected networks, tagged with the selected key and value. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)

### Implementation.ECS.DaemonTaskDefinition.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::DaemonTaskDefinition`: `$.TaskRoleArn`, `$.ExecutionRoleArn`

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)): This implementation guidance recommends validating the trust policies of `$.TaskRoleArn` and `$.ExecutionRoleArn`, your daemon task definition's task role and task execution role, including source conditions on service principals, before provisioning. This helps prevent the daemon task definition from being provisioned when any trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.

The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one sts:AssumeRole statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.DaemonTaskDefinition.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListDaemonTaskDefinitions`; check `$.daemonTaskDefinitions[*].arn`
2. Call `ecs:DescribeDaemonTaskDefinition` with `$.daemonTaskDefinition`; check `$.daemonTaskDefinition.taskRoleArn`, `$.daemonTaskDefinition.executionRoleArn`
3. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result` = `PASS`
5. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)): This implementation guidance recommends reading `$.daemonTaskDefinition.taskRoleArn` and `$.daemonTaskDefinition.executionRoleArn`, your daemon task definition's task role and task execution role, then evaluating `$.Role.AssumeRolePolicyDocument`, each role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.

The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.DaemonTaskDefinition.7

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs-tasks.amazonaws.com` | `sts:AssumeRole` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS daemon task definition's task execution role, `$.executionRoleArn`, to the `ecs-tasks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon ECS from assuming the role on behalf of Amazon ECS resources outside the sources you specify.

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS daemon task definition's task role, `$.taskRoleArn`, to the `ecs-tasks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon ECS from assuming the role on behalf of Amazon ECS resources outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.ECS.DaemonTaskDefinition.8

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon ECS daemon task definition's task role and task execution role, `$.taskRoleArn` and `$.executionRoleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.ECS.DaemonTaskDefinition.9

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::DaemonTaskDefinition`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon ECS daemon task definition that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.ECS.DaemonTaskDefinition.10

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListDaemonTaskDefinitions`; check `$.daemonTaskDefinitions[*].arn`
2. Call `resource-groups-tagging-api:GetResources` with `$.ResourceARNList[*]`; check `$.ResourceTagMappingList[*].ResourceARN`, `$.ResourceTagMappingList[*].Tags[*].Key`, `$.ResourceTagMappingList[*].Tags[*].Value`

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4)): This implementation guidance recommends reading `$.ResourceTagMappingList[*].Tags[*].Key` on each Amazon ECS daemon task definition listed through `$.daemonTaskDefinitions[*].arn` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS daemon task definition to an accountable owner and the business function it supports.

### Implementation.ECS.DaemonTaskDefinition.11

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:RegisterDaemonTaskDefinition` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose` |
| `ecs:RegisterDaemonTaskDefinition` | `*` | `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:daemon-task-definition/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:daemon-task-definition/*` | `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:daemon-task-definition/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4)

**Scoping** (for [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4)): This implementation guidance recommends restricting `ecs:RegisterDaemonTaskDefinition` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS daemon task definition without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.ExpressGatewayService.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::ExpressGatewayService`: `$.PrimaryContainer.AwsLogsConfiguration.LogGroup`, `$.PrimaryContainer.AwsLogsConfiguration.LogStreamPrefix`

**Implements:** [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1)): This implementation guidance recommends validating that `$.PrimaryContainer.AwsLogsConfiguration.LogGroup`, the log group that receives container logs, and `$.PrimaryContainer.AwsLogsConfiguration.LogStreamPrefix`, the log stream name prefix, are specified before provisioning. That check helps prevent an Amazon ECS express gateway service from being provisioned with only the default destination for its container logging to CloudWatch Logs, where you can review application output and errors.

### Implementation.ECS.ExpressGatewayService.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`, `$.logStreamName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `ecs:CreateExpressGatewayService` — `$.primaryContainer.awsLogsConfiguration.logGroup`, `$.primaryContainer.awsLogsConfiguration.logStreamPrefix`, `$.taskDefinitionArn`
- `ecs:DeleteExpressGatewayService` — `$.serviceArn`
- `ecs:RegisterTaskDefinition` — `$.containerDefinitions[*].logConfiguration.logDriver` = `awslogs`, `$.family`
- `ecs:UpdateExpressGatewayService` — `$.primaryContainer.awsLogsConfiguration.logGroup`, `$.primaryContainer.awsLogsConfiguration.logStreamPrefix`, `$.serviceArn`, `$.taskDefinitionArn`
- `ecs:UpdateService` — `$.service`, `$.taskDefinition`

**Implements:** [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1)): This implementation guidance recommends recording `UpdateExpressGatewayService` calls that set `$.primaryContainer.awsLogsConfiguration.logGroup` (the destination log group), `$.primaryContainer.awsLogsConfiguration.logStreamPrefix` (the log stream name prefix), or `$.taskDefinitionArn` (a custom task definition that configures its own logging), as well as `RegisterTaskDefinition` calls that set `$.containerDefinitions[*].logConfiguration.logDriver` (the container log driver) and `UpdateService` calls that set `$.taskDefinition` (the task definition your service runs). Recording these calls alongside `DeleteExpressGatewayService` and the CloudWatch Logs `DeleteLogGroup`, `DeleteLogStream`, and `PutRetentionPolicy` operations helps you detect changes that disable or weaken container logging or erase its log records. Recording `CreateExpressGatewayService` as well lets you detect an Amazon ECS express gateway service created without this logging.

### Implementation.ECS.ExpressGatewayService.3

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::Service`: `$.configuration.ServiceArn`, `$.configuration.TaskDefinition`

**Implements:** [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1)): This implementation guidance recommends reading `$.service.activeConfigurations[*].primaryContainer.awsLogsConfiguration.logGroup`, the log group that receives your container logs, and `$.service.activeConfigurations[*].primaryContainer.awsLogsConfiguration.logStreamPrefix`, the prefix for its log stream names, from `DescribeExpressGatewayService` for `$.configuration.ServiceArn`, your Express service's ARN, then `$.taskDefinition.containerDefinitions[*].logConfiguration.logDriver`, each container's log driver, and `$.taskDefinition.containerDefinitions[*].logConfiguration.options{key}`, the options for that driver, from `DescribeTaskDefinition` for the task definition in use. Flagging Express services that record no log group for container logs or whose containers do not use the `awslogs` log driver helps you detect gaps in container logging to CloudWatch Logs.

### Implementation.ECS.ExpressGatewayService.4

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalTag/dp:exclude:network`, `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.ECS.ExpressGatewayService.2](#mitigationecsexpressgatewayservice2)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.2](#mitigationecsexpressgatewayservice2)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon ECS express gateway service's task IAM role and task execution role credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalTag/tag-key` to apply the restriction to principals whose credentials can be used outside your expected networks, tagged with the selected key and value. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)

### Implementation.ECS.ExpressGatewayService.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::ExpressGatewayService`: `$.ExecutionRoleArn`, `$.TaskRoleArn`, `$.InfrastructureRoleArn`

**Implements:** [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)): This implementation guidance recommends validating the trust policies of `$.InfrastructureRoleArn`, `$.ExecutionRoleArn`, and `$.TaskRoleArn`, your express gateway service's infrastructure role, task execution role, and task role, including source conditions on service principals, before provisioning. This helps prevent the express gateway service from being provisioned when any trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.

The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.ExpressGatewayService.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`, `$.resourceManagementType` = `ECS`; check `$.serviceArns[*]`
3. Call `ecs:DescribeExpressGatewayService` with `$.serviceArn`; check `$.service.infrastructureRoleArn`, `$.service.activeConfigurations[*].executionRoleArn`, `$.service.activeConfigurations[*].taskRoleArn`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result` = `PASS`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
7. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)): This implementation guidance recommends reading `$.service.infrastructureRoleArn`, `$.service.activeConfigurations[*].executionRoleArn`, and `$.service.activeConfigurations[*].taskRoleArn`, your express gateway service's infrastructure role, task execution role, and task role, then evaluating `$.Role.AssumeRolePolicyDocument`, each role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys on your task execution role and task role, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.

The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.ExpressGatewayService.7

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS Express Gateway service's infrastructure role, `$.infrastructureRoleArn`, to the `ecs.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.ECS.ExpressGatewayService.8

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs-tasks.amazonaws.com` | `sts:AssumeRole` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS Express Gateway service's task execution role, `$.executionRoleArn`, to the `ecs-tasks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon ECS from assuming the role on behalf of Amazon ECS resources outside the sources you specify.

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS express gateway service's task role, `$.taskRoleArn`, to the `ecs-tasks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon ECS from assuming the role on behalf of Amazon ECS resources in accounts outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.ECS.ExpressGatewayService.9

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon ECS express gateway service's infrastructure role, task execution role, and task role, `$.infrastructureRoleArn` and `$.executionRoleArn` and `$.taskRoleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.ECS.ExpressGatewayService.10

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::ExpressGatewayService`: `$.NetworkConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4)): This implementation guidance recommends validating `$.NetworkConfiguration.Subnets[*]`, the subnets that host your service's Fargate tasks, by resolving each subnet's route table for a default route to an internet gateway before provisioning. Subnets carrying that route fail validation, which helps prevent an Amazon ECS express gateway service from being provisioned in a public subnet. A template that omits the property fails validation too, because the service then uses the default public subnets in the default VPC.

### Implementation.ECS.ExpressGatewayService.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`, `$.resourceManagementType` = `ECS`; check `$.serviceArns[*]`
3. Call `ecs:DescribeExpressGatewayService` with `$.serviceArn`; check `$.service.activeConfigurations[*].networkConfiguration.subnets[*]`, `$.service.activeConfigurations[*].ingressPaths[*].accessType`
4. Call `ec2:DescribeRouteTables` with `$.Filters`; check `$.RouteTables[*].Routes[*].GatewayId`, `$.RouteTables[*].Routes[*].DestinationCidrBlock` = `0.0.0.0/0`, `$.RouteTables[*].Routes[*].DestinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4)): This implementation guidance recommends reading `$.service.activeConfigurations[*].networkConfiguration.subnets[*]`, the subnets hosting the Express service's tasks, then resolving each subnet's route table. Flagging an Express service with a subnet whose default route targets an internet gateway, read at `$.RouteTables[*].Routes[*].GatewayId`, helps you detect placement in a public subnet.

### Implementation.ECS.ExpressGatewayService.12

*Guidance — Directive*

Configure the Express service's network configuration with two or more subnets that resolve to different Availability Zones. Each subnet resides entirely within one Availability Zone. The subnet IDs you supply therefore determine where the service places its elastic network interfaces (ENIs). Express Mode services run on AWS Fargate with the `awsvpc` network mode. Each task receives its own ENI in one of the subnets you provide. Supplying subnets from at least two Availability Zones helps network connectivity survive the loss of a single zone.

Subnet-to-Availability-Zone membership is a property of each subnet, not of the service. Before you supply the subnet IDs, confirm that they belong to distinct Availability Zones. Call `DescribeSubnets` and check the `AvailabilityZone` response field for each subnet. Standardize on infrastructure-as-code review so the templates and modules that create your Express services supply at least two subnets from different Availability Zones.

For Express Mode, the first service created in a VPC defines the subnets associated with that VPC's load balancer. Later Express services in the same VPC must use subnets matching the Availability Zones that load balancer supports. Because of this, prefer subnets from all Availability Zones in the Region.

Express Mode sets your service's minimum task count to one by default. At that minimum, your service runs a single task whose ENI sits in one Availability Zone. Set `$.scalingTarget.minTaskCount`, your service's minimum task count, to two or more when you create or update the service. Fargate makes a best attempt to balance your service's tasks across the zones of the subnets you supply. For production workloads, run tasks in three Availability Zones, for example with a minimum task count of three.

To verify the deployed state, enumerate the service's network interfaces with `DescribeNetworkInterfaces`. Confirm that the attached ENIs' subnets span more than one Availability Zone. The Express service's own describe operation also echoes the subnet placement you configured.

To assess and track multi-AZ posture at scale, use AWS Resilience Hub. Include the Express service's application resources in a Resilience Hub application and run a resiliency assessment. The assessment flags single-Availability-Zone concentration against your resiliency policy.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5)): This implementation guidance recommends supplying two or more subnets from different Availability Zones in your Express service's network configuration. Spreading the elastic network interfaces that AWS Fargate creates for your service's tasks across zones helps task connectivity survive the loss of a single Availability Zone.

### Implementation.ECS.ExpressGatewayService.13

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::ExpressGatewayService`: `$.NetworkConfiguration.Subnets[*]`, `$.ScalingTarget.MinTaskCount`

**Implements:** [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5)): This implementation guidance recommends validating, before provisioning, that `$.NetworkConfiguration.Subnets[*]`, your Amazon ECS express gateway service's VPC subnets, lists two or more subnet entries. That check helps prevent a single-subnet express gateway service from being provisioned, so its tasks' network interfaces can span more than one Availability Zone.

### Implementation.ECS.ExpressGatewayService.14

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::Service`: `$.configuration.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5)): This implementation guidance recommends reading `$.configuration.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`, the subnets where your Express service launches its tasks, then resolving each subnet's Availability Zone. Flagging an Express service confined to a single Availability Zone helps you detect the network interfaces of its Fargate tasks that lack multi-zone redundancy.

### Implementation.ECS.ExpressGatewayService.15

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::ExpressGatewayService`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon ECS express gateway service that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.ECS.ExpressGatewayService.16

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::Service`: `$.tags[*].key`, `$.tags[*].value`, `$.configuration.Tags[*].Key`, `$.configuration.Tags[*].Value`

**Implements:** [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6)): This implementation guidance recommends reading `$.tags[*].key` and `$.configuration.Tags[*].Key` on each Amazon ECS Express service and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS Express service to an accountable owner and the business function it supports.

### Implementation.ECS.ExpressGatewayService.17

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:CreateExpressGatewayService` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:service/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:service/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6)): This implementation guidance recommends restricting `ecs:CreateExpressGatewayService` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS express gateway service without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.ExpressGatewayService.18

*Guidance — Directive*

Configure the security groups associated with your Amazon ECS express gateway service following least-privilege best practices. Restrict both inbound and outbound rules to only the sources and destinations, protocols, and ports that your application requires. Each source or destination can be an IP address range, a referenced security group, or a prefix list. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change. Express Mode creates a service security group and a load balancer security group when you do not supply your own. Each security group you add to the service's network configuration provides an additional ingress path to your service. Review the rules on every group associated with the service.

To govern security groups at scale, use AWS Firewall Manager security group policies across your organization in AWS Organizations. You can scope a policy by account, organizational unit, or tag. A common security group policy can't apply a baseline security group to your service's tasks. Firewall Manager doesn't support updating security groups on network interfaces created for Fargate tasks. Express Mode runs your containers only on Fargate. A content audit policy audits inbound and outbound rules against an allow-list or a block-list. It flags non-compliant groups in a central dashboard and notifies you. You can also enable automatic remediation. A usage audit policy flags unused and redundant security groups. With automatic remediation enabled, it can remove unused groups and consolidate redundant groups, depending on the options you choose. If Firewall Manager can't delete a group, such as one that another security group references, it marks the group as non-compliant.

To verify unintended access that your security groups permit, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended. For example, define a scope that matches any path reaching administrative ports 22 or 3389 from an unrestricted address. Then run an analysis by calling `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`. Review the resulting findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)): This implementation guidance recommends restricting the inbound and outbound rules on the security groups in `$.networkConfiguration.securityGroups[*]` to the required addresses, protocols, and ports. Excluding unrestricted addresses such as 0.0.0.0/0 or ::/0 from those inbound and outbound rules helps reduce the network exposure of your Amazon ECS express gateway service.

### Implementation.ECS.ExpressGatewayService.19

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::ExpressGatewayService`: `$.NetworkConfiguration.SecurityGroups[*]`

**Implements:** [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)): This implementation guidance recommends validating the inbound and outbound rules of the security groups resolved from `$.NetworkConfiguration.SecurityGroups[*]` before provisioning. Checking for rules that allow any address on unintended ports, directly or through a prefix list, helps prevent an Amazon ECS express gateway service from being provisioned with such groups.

### Implementation.ECS.ExpressGatewayService.20

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`, `$.resourceManagementType` = `ECS`; check `$.serviceArns[*]`
3. Call `ecs:DescribeExpressGatewayService` with `$.serviceArn`; check `$.service.activeConfigurations[*].networkConfiguration.securityGroups[*]`
4. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
5. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)

**Scoping** (for [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7)): This implementation guidance recommends resolving the security groups recorded at `$.service.activeConfigurations[*].networkConfiguration.securityGroups[*]` and inspecting their inbound and outbound rules. Flagging rules that admit 0.0.0.0/0 or ::/0 on all protocols helps you detect your Amazon ECS express gateway services with broader network exposure than they need.

### Implementation.ECS.Service.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.ServiceConnectConfiguration.Enabled`, `$.ServiceConnectConfiguration.LogConfiguration.LogDriver`, `$.ServiceConnectConfiguration.LogConfiguration.Options{key}`, `$.ServiceConnectConfiguration.AccessLogConfiguration.Format`

**Implements:** [Mitigation.ECS.Service.1](#mitigationecsservice1)

**Scoping** (for [Mitigation.ECS.Service.1](#mitigationecsservice1)): This implementation guidance recommends validating `$.ServiceConnectConfiguration.LogConfiguration.LogDriver`, the Service Connect proxy log driver, for `awslogs`; `$.ServiceConnectConfiguration.LogConfiguration.Options{key}`, the driver options, for a log group named in `awslogs-group`; and `$.ServiceConnectConfiguration.AccessLogConfiguration.Format`, the access log format, for a `TEXT` or `JSON` value before provisioning. That check helps prevent an Amazon ECS service's Service Connect configuration from being provisioned without configured proxy and access logging to CloudWatch Logs, so you can review service-to-service traffic.

### Implementation.ECS.Service.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`, `$.logStreamName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `ecs:ContinueServiceDeployment` — `$.action` = `ROLLBACK`, `$.serviceDeploymentArn`
- `ecs:CreateService` — `$.cluster`, `$.serviceConnectConfiguration.accessLogConfiguration.format`, `$.serviceConnectConfiguration.enabled`, `$.serviceConnectConfiguration.logConfiguration.logDriver`, `$.serviceConnectConfiguration.logConfiguration.options{key}`, `$.serviceName`
- `ecs:StopServiceDeployment` — `$.serviceDeploymentArn`, `$.stopType` = `ROLLBACK`
- `ecs:UpdateService` — `$.cluster`, `$.service`, `$.serviceConnectConfiguration.accessLogConfiguration.format`, `$.serviceConnectConfiguration.enabled`, `$.serviceConnectConfiguration.logConfiguration.logDriver`, `$.serviceConnectConfiguration.logConfiguration.options{key}`

**Implements:** [Mitigation.ECS.Service.1](#mitigationecsservice1)

**Scoping** (for [Mitigation.ECS.Service.1](#mitigationecsservice1)): This implementation guidance recommends recording `UpdateService` calls that set `$.serviceConnectConfiguration.enabled`, whether the service uses Service Connect, and `$.serviceConnectConfiguration.logConfiguration.logDriver`, the log driver that delivers Service Connect logs. Recording these calls alongside CloudWatch Logs `DeleteLogGroup`, `DeleteLogStream`, and `PutRetentionPolicy` calls against the destination log group helps you detect changes that disable or weaken Service Connect proxy and access logging or erase its log records. Recording `CreateService` as well lets you detect an Amazon ECS service's Service Connect configuration created without this logging.

### Implementation.ECS.Service.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].deployments[*].status`, `$.services[*].deployments[*].serviceConnectConfiguration.enabled`, `$.services[*].deployments[*].serviceConnectConfiguration.logConfiguration.logDriver`, `$.services[*].deployments[*].serviceConnectConfiguration.logConfiguration.options{key}`, `$.services[*].deployments[*].serviceConnectConfiguration.accessLogConfiguration.format`, `$.services[*].currentServiceRevisions[*].arn`
4. Call `ecs:DescribeServiceRevisions` with `$.serviceRevisionArns[*]`; check `$.serviceRevisions[*].serviceConnectConfiguration.enabled`, `$.serviceRevisions[*].serviceConnectConfiguration.logConfiguration.logDriver`, `$.serviceRevisions[*].serviceConnectConfiguration.logConfiguration.options{key}`, `$.serviceRevisions[*].serviceConnectConfiguration.accessLogConfiguration.format`

**Implements:** [Mitigation.ECS.Service.1](#mitigationecsservice1)

**Scoping** (for [Mitigation.ECS.Service.1](#mitigationecsservice1)): This implementation guidance recommends reading the Service Connect logging settings on each service's deployments and current service revisions, such as `$.services[*].deployments[*].serviceConnectConfiguration.logConfiguration.logDriver`, the proxy's log driver, and `$.serviceRevisions[*].serviceConnectConfiguration.accessLogConfiguration.format`, the access log format, after listing each cluster's services through `ListClusters` and `ListServices`. Flagging services that use Service Connect but lack either the `awslogs` log driver or an access log configuration helps you detect gaps in Service Connect proxy and access logging to CloudWatch Logs.

### Implementation.ECS.Service.4

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.Role`, `$.VolumeConfigurations[*].ManagedEBSVolume.RoleArn`, `$.VpcLatticeConfigurations[*].RoleArn`, `$.LoadBalancers[*].AdvancedConfiguration.RoleArn`, `$.DeploymentConfiguration.LifecycleHooks[*].RoleArn`, `$.ServiceConnectConfiguration.Services[*].Tls.RoleArn`

**Implements:** [Mitigation.ECS.Service.2](#mitigationecsservice2)

**Scoping** (for [Mitigation.ECS.Service.2](#mitigationecsservice2)): This implementation guidance recommends validating the trust policies of `$.VolumeConfigurations[*].ManagedEBSVolume.RoleArn` and `$.VpcLatticeConfigurations[*].RoleArn` and `$.ServiceConnectConfiguration.Services[*].Tls.RoleArn` and `$.LoadBalancers[*].AdvancedConfiguration.RoleArn` and `$.DeploymentConfiguration.LifecycleHooks[*].RoleArn` and `$.Role`, your Amazon ECS service's infrastructure role, infrastructure role for load balancers, deployment lifecycle hook role, and service role, before provisioning. This helps prevent the Amazon ECS service from being provisioned when any trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.Service.5

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].roleArn`, `$.services[*].loadBalancers[*].advancedConfiguration.roleArn`, `$.services[*].deploymentConfiguration.lifecycleHooks[*].roleArn`, `$.services[*].deployments[*].volumeConfigurations[*].managedEBSVolume.roleArn`, `$.services[*].deployments[*].vpcLatticeConfigurations[*].roleArn`, `$.services[*].deployments[*].serviceConnectConfiguration.services[*].tls.roleArn`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.ECS.Service.2](#mitigationecsservice2)

**Scoping** (for [Mitigation.ECS.Service.2](#mitigationecsservice2)): This implementation guidance recommends reading `$.services[*].deployments[*].volumeConfigurations[*].managedEBSVolume.roleArn`, `$.services[*].deployments[*].vpcLatticeConfigurations[*].roleArn`, `$.services[*].deployments[*].serviceConnectConfiguration.services[*].tls.roleArn`, `$.services[*].loadBalancers[*].advancedConfiguration.roleArn`, `$.services[*].deploymentConfiguration.lifecycleHooks[*].roleArn`, and `$.services[*].roleArn`, your service's infrastructure role, infrastructure role for load balancers, deployment lifecycle hook role, and service role, then evaluating `$.Role.AssumeRolePolicyDocument`, each role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.Service.6

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.ECS.Service.2](#mitigationecsservice2)

**Scoping** (for [Mitigation.ECS.Service.2](#mitigationecsservice2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS service's deployment lifecycle hook role, `$.deploymentConfiguration.lifecycleHooks[*].roleArn`, to the `ecs.amazonaws.com` service principal.

**Scoping** (for [Mitigation.ECS.Service.2](#mitigationecsservice2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS service's infrastructure role for load balancers, `$.loadBalancers[*].advancedConfiguration.roleArn`, to the `ecs.amazonaws.com` service principal.

**Scoping** (for [Mitigation.ECS.Service.2](#mitigationecsservice2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS service's infrastructure role, `$.volumeConfigurations[*].managedEBSVolume.roleArn` and `$.vpcLatticeConfigurations[*].roleArn` and `$.serviceConnectConfiguration.services[*].tls.roleArn`, to the `ecs.amazonaws.com` service principal.

**Scoping** (for [Mitigation.ECS.Service.2](#mitigationecsservice2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS service's service role, `$.role`, to the `ecs.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.ECS.Service.7

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.ECS.Service.2](#mitigationecsservice2)

**Scoping** (for [Mitigation.ECS.Service.2](#mitigationecsservice2)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon ECS service's infrastructure role, infrastructure role for load balancers, deployment lifecycle hook role, and service role, `$.volumeConfigurations[*].managedEBSVolume.roleArn`, `$.vpcLatticeConfigurations[*].roleArn`, `$.serviceConnectConfiguration.services[*].tls.roleArn`, `$.loadBalancers[*].advancedConfiguration.roleArn`, `$.deploymentConfiguration.lifecycleHooks[*].roleArn`, and `$.role`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.ECS.Service.8

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.VolumeConfigurations[*].ManagedEBSVolume.Encrypted`, `$.VolumeConfigurations[*].Name`

**Implements:** [Mitigation.ECS.Service.3](#mitigationecsservice3)

**Scoping** (for [Mitigation.ECS.Service.3](#mitigationecsservice3)): This implementation guidance recommends validating that `$.VolumeConfigurations[*].ManagedEBSVolume.Encrypted`, the encryption setting for managed Amazon EBS volumes, is set to true before provisioning an Amazon ECS service. This helps prevent the Amazon ECS service from being provisioned without encryption on its managed volumes.

### Implementation.ECS.Service.9

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].serviceArn`, `$.services[*].deployments[*].status`, `$.services[*].deployments[*].volumeConfigurations[*].name`, `$.services[*].deployments[*].volumeConfigurations[*].managedEBSVolume.encrypted`

**Implements:** [Mitigation.ECS.Service.3](#mitigationecsservice3)

**Scoping** (for [Mitigation.ECS.Service.3](#mitigationecsservice3)): This implementation guidance recommends flagging an Amazon ECS service whose `$.services[*].deployments[*].volumeConfigurations[*].managedEBSVolume.encrypted`, the encryption setting for its managed Amazon EBS volumes, is not true. Surfacing those services helps you detect where data on the Amazon EBS volumes attached to their tasks can be stored unencrypted.

### Implementation.ECS.Service.10

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`, `$.NetworkConfiguration.AwsvpcConfiguration.AssignPublicIp`

**Implements:** [Mitigation.ECS.Service.4](#mitigationecsservice4)

**Scoping** (for [Mitigation.ECS.Service.4](#mitigationecsservice4)): This implementation guidance recommends validating `$.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`, the subnets that hold your tasks' network interfaces, by resolving each subnet's route table for a default route to an internet gateway before provisioning. Subnets carrying that route fail validation, which helps prevent an Amazon ECS service from being provisioned in a public subnet.

### Implementation.ECS.Service.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.services[*].networkConfiguration.awsvpcConfiguration.assignPublicIp`
4. Call `ec2:DescribeRouteTables` with `$.Filters`; check `$.RouteTables[*].Routes[*].GatewayId`, `$.RouteTables[*].Routes[*].DestinationCidrBlock` = `0.0.0.0/0`, `$.RouteTables[*].Routes[*].DestinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.ECS.Service.4](#mitigationecsservice4)

**Scoping** (for [Mitigation.ECS.Service.4](#mitigationecsservice4)): This implementation guidance recommends reading `$.services[*].networkConfiguration.awsvpcConfiguration.subnets[*]`, the subnets for your tasks' network interfaces, then resolving each subnet's route table. Flagging an Amazon ECS service with a subnet whose default route targets an internet gateway, read at `$.RouteTables[*].Routes[*].GatewayId`, helps you detect placement in a public subnet.

### Implementation.ECS.Service.12

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.AvailabilityZoneRebalancing`, `$.PlacementStrategies[*].Type`, `$.PlacementStrategies[*].Field`, `$.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.Service.5](#mitigationecsservice5)

**Scoping** (for [Mitigation.ECS.Service.5](#mitigationecsservice5)): This implementation guidance recommends validating before provisioning that `$.AvailabilityZoneRebalancing`, the service's Availability Zone rebalancing setting, is `ENABLED`. This helps prevent an Amazon ECS service unable to automatically redistribute tasks across Availability Zones from being provisioned.

### Implementation.ECS.Service.13

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].availabilityZoneRebalancing`, `$.services[*].networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.services[*].placementStrategy[*].type`, `$.services[*].placementStrategy[*].field`, `$.services[*].schedulingStrategy`, `$.services[*].launchType`, `$.services[*].deploymentConfiguration.maximumPercent`, `$.services[*].status`
4. Call `ec2:DescribeSubnets` with `$.SubnetId[*]`; check `$.subnetSet[*].subnetId`, `$.subnetSet[*].availabilityZone`

**Implements:** [Mitigation.ECS.Service.5](#mitigationecsservice5)

**Scoping** (for [Mitigation.ECS.Service.5](#mitigationecsservice5)): This implementation guidance recommends reading `$.services[*].availabilityZoneRebalancing`, the rebalancing setting, and resolving `$.subnetSet[*].availabilityZone`, each subnet's Availability Zone. Flagging services confined to one Availability Zone or with rebalancing turned off helps you detect workloads that cannot automatically restore a balanced distribution after a zone disruption.

### Implementation.ECS.Service.14

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.VolumeConfigurations[*].ManagedEBSVolume.KmsKeyId`, `$.ServiceConnectConfiguration.Services[*].Tls.KmsKey`

**Implements:** [Mitigation.ECS.Service.6](#mitigationecsservice6)

**Scoping** (for [Mitigation.ECS.Service.6](#mitigationecsservice6)): This implementation guidance recommends validating `$.VolumeConfigurations[*].ManagedEBSVolume.KmsKeyId`, the key that encrypts your service's managed Amazon EBS volumes, and `$.ServiceConnectConfiguration.Services[*].Tls.KmsKey`, the key that encrypts your service's Service Connect TLS resources, before provisioning. Requiring a customer managed key in your account, with key material that is neither imported nor held in an external key store, helps prevent an Amazon ECS service from being provisioned without a key whose policy, lifecycle, and key material you control.

### Implementation.ECS.Service.15

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].deployments[*].volumeConfigurations[*].managedEBSVolume.kmsKeyId`, `$.services[*].deployments[*].serviceConnectConfiguration.services[*].tls.kmsKey`
4. Call `kms:DescribeKey` with `$.KeyId`; check `$.KeyMetadata.KeyManager` = `CUSTOMER`, `$.KeyMetadata.AWSAccountId`, `$.KeyMetadata.Origin`

**Implements:** [Mitigation.ECS.Service.6](#mitigationecsservice6)

**Scoping** (for [Mitigation.ECS.Service.6](#mitigationecsservice6)): This implementation guidance recommends reading `$.services[*].deployments[*].volumeConfigurations[*].managedEBSVolume.kmsKeyId`, the key that encrypts the Amazon EBS volumes attached to your service's tasks, and `$.services[*].deployments[*].serviceConnectConfiguration.services[*].tls.kmsKey`, the key that encrypts the private keys of your service's Service Connect TLS certificates, and resolving `$.KeyMetadata.KeyManager`, `$.KeyMetadata.AWSAccountId`, and `$.KeyMetadata.Origin` for each key. Flagging Amazon ECS services whose key is not a customer managed key in your account, or whose key material is imported or held in an external key store, helps you detect where you do not control the key policy, lifecycle, or key material.

### Implementation.ECS.Service.16

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ec2:CreateVolume` | `arn:aws:ec2:*:*:volume/*` | `aws:RequestTag/AmazonECSManaged`, `ec2:KmsKeyId` |
| `secretsmanager:CreateSecret` | `arn:aws:secretsmanager:*:*:secret:*` | `aws:RequestTag/AmazonECSManaged`, `secretsmanager:KmsKeyArn` |

**Implements:** [Mitigation.ECS.Service.6](#mitigationecsservice6)

**Scoping** (for [Mitigation.ECS.Service.6](#mitigationecsservice6)): This implementation guidance recommends applying the `ec2:KmsKeyId` and `secretsmanager:KmsKeyArn` condition keys in a service control policy (SCP) to govern `ec2:CreateVolume` and `secretsmanager:CreateSecret`, scoped by `aws:RequestTag/AmazonECSManaged`. These conditions help prevent creating an Amazon ECS service's managed EBS volume or Service Connect TLS secret without an approved customer managed key.

### Implementation.ECS.Service.17

*Guidance — Directive*

Configure your Amazon ECS service's `networkConfiguration.awsvpcConfiguration.subnets` list with two or more subnets that resolve to different Availability Zones. When a task definition uses the `awsvpc` network mode, Amazon ECS creates a dedicated elastic network interface (ENI) for each task it places in that subnet list. Supplying subnets from more than one zone puts those ENIs in more than one Availability Zone. Network connectivity for your service then survives the loss of a single zone.

Availability Zone membership is a property of each subnet, not of the service. Before you supply subnet IDs to `CreateService` or `UpdateService`, confirm that they belong to distinct Availability Zones. Call `DescribeSubnets` in Amazon EC2 and compare the `AvailabilityZone` response field for each subnet. Standardize on infrastructure-as-code review so the templates and modules that create your services supply at least two subnets from different zones.

To verify the deployed state, enumerate the network interfaces attached to your service's tasks with `DescribeNetworkInterfaces` in Amazon EC2. You can also call `DescribeServices`, which echoes the subnet placement recorded for the service. Confirm that the subnets of the attached ENIs span more than one Availability Zone.

To assess and track multi-AZ posture at scale, use AWS Resilience Hub. Include your service's application resources in a Resilience Hub application and run a resiliency assessment. The assessment flags single-AZ concentration against your resiliency policy.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.ECS.Service.7](#mitigationecsservice7)

**Scoping** (for [Mitigation.ECS.Service.7](#mitigationecsservice7)): This implementation guidance recommends supplying two or more subnets from different Availability Zones in `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, the subnet IDs in your Amazon ECS service's network configuration. Spreading the elastic network interfaces Amazon ECS creates for your service's tasks across zones helps task connectivity survive the loss of a single Availability Zone.

### Implementation.ECS.Service.18

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.Service.7](#mitigationecsservice7)

**Scoping** (for [Mitigation.ECS.Service.7](#mitigationecsservice7)): This implementation guidance recommends validating, before provisioning, that `$.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`, your Amazon ECS service's VPC subnets, lists two or more subnet entries. That check helps prevent a single-subnet Amazon ECS service from being provisioned, so your task elastic network interfaces can span more than one Availability Zone.

### Implementation.ECS.Service.19

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::Service`: `$.configuration.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.Service.7](#mitigationecsservice7)

**Scoping** (for [Mitigation.ECS.Service.7](#mitigationecsservice7)): This implementation guidance recommends reading `$.configuration.NetworkConfiguration.AwsvpcConfiguration.Subnets[*]`, the subnets where your service launches its tasks, then resolving each subnet's Availability Zone. Flagging an `awsvpc`-mode service confined to a single Availability Zone helps you detect task network interfaces that lack multi-zone redundancy.

### Implementation.ECS.Service.20

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.ServiceConnectConfiguration.Enabled`, `$.ServiceConnectConfiguration.Services[*].Tls.IssuerCertificateAuthority.AwsPcaAuthorityArn`

**Implements:** [Mitigation.ECS.Service.8](#mitigationecsservice8)

**Scoping** (for [Mitigation.ECS.Service.8](#mitigationecsservice8)): This implementation guidance recommends validating that `$.ServiceConnectConfiguration.Enabled`, the setting that turns on Service Connect, is `true` and that `$.ServiceConnectConfiguration.Services[*].Tls.IssuerCertificateAuthority.AwsPcaAuthorityArn`, the signer certificate authority for each Service Connect endpoint, holds an AWS Private Certificate Authority ARN before provisioning an Amazon ECS service. This check helps prevent an Amazon ECS service from being provisioned without a TLS issuer for its Service Connect endpoints, which can leave inter-service traffic in cleartext.

### Implementation.ECS.Service.21

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].serviceArn`, `$.services[*].deployments[*].status`, `$.services[*].deployments[*].serviceConnectConfiguration.enabled`, `$.services[*].deployments[*].serviceConnectConfiguration.services[*].tls.issuerCertificateAuthority.awsPcaAuthorityArn`, `$.services[*].deployments[*].serviceConnectConfiguration.services[*].tls.roleArn`, `$.services[*].deployments[*].serviceConnectConfiguration.services[*].tls.kmsKey`

**Implements:** [Mitigation.ECS.Service.8](#mitigationecsservice8)

**Scoping** (for [Mitigation.ECS.Service.8](#mitigationecsservice8)): This implementation guidance recommends reading `$.services[*].deployments[*].serviceConnectConfiguration.enabled`, whether Service Connect is turned on, and `$.services[*].deployments[*].serviceConnectConfiguration.services[*].tls.issuerCertificateAuthority.awsPcaAuthorityArn`, the AWS Private CA that signs your Service Connect TLS certificates, in the primary deployment of each of your Amazon ECS services. Flagging services that name no Service Connect TLS issuer helps you detect service-to-service traffic that travels between your services without TLS protection.

### Implementation.ECS.Service.22

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.ECS.Service.9](#mitigationecsservice9)

**Scoping** (for [Mitigation.ECS.Service.9](#mitigationecsservice9)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon ECS service that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.ECS.Service.23

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::Service`: `$.tags[*].key`, `$.tags[*].value`, `$.configuration.Tags[*].Key`, `$.configuration.Tags[*].Value`

**Implements:** [Mitigation.ECS.Service.9](#mitigationecsservice9)

**Scoping** (for [Mitigation.ECS.Service.9](#mitigationecsservice9)): This implementation guidance recommends reading `$.tags[*].key` and `$.configuration.Tags[*].Key` on each Amazon ECS service and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS service to an accountable owner and the business function it supports.

### Implementation.ECS.Service.24

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:CreateService` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:service/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:service/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.Service.9](#mitigationecsservice9)

**Scoping** (for [Mitigation.ECS.Service.9](#mitigationecsservice9)): This implementation guidance recommends restricting `ecs:CreateService` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS service without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.Service.25

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.Monitoring.MetricConfigurations[*].ResolutionSeconds`, `$.Monitoring.MetricConfigurations[*].MetricNames[*]`

**Implements:** [Mitigation.ECS.Service.10](#mitigationecsservice10)

**Scoping** (for [Mitigation.ECS.Service.10](#mitigationecsservice10)): This implementation guidance recommends validating `$.Monitoring.MetricConfigurations[*].ResolutionSeconds` and `$.Monitoring.MetricConfigurations[*].MetricNames[*]`, the service's monitoring configuration, set to a resolution of `20` seconds for `CPUUtilization` and `MemoryUtilization` before provisioning. That validation helps address the mitigation by preventing an Amazon ECS service from being provisioned without detailed CloudWatch metrics.

### Implementation.ECS.Service.26

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `ecs:CreateService` — `$.cluster`, `$.monitoring.metricConfigurations[*].metricNames[*]`, `$.monitoring.metricConfigurations[*].resolutionSeconds`, `$.serviceName`
- `ecs:StopServiceDeployment` — `$.serviceDeploymentArn`, `$.stopType` = `ROLLBACK`
- `ecs:UpdateService` — `$.cluster`, `$.monitoring.metricConfigurations[*].metricNames[*]`, `$.monitoring.metricConfigurations[*].resolutionSeconds`, `$.service`

**Implements:** [Mitigation.ECS.Service.10](#mitigationecsservice10)

**Scoping** (for [Mitigation.ECS.Service.10](#mitigationecsservice10)): This implementation guidance recommends recording `UpdateService`, `StopServiceDeployment`, and `CreateService` calls carrying `$.monitoring.metricConfigurations[*].resolutionSeconds`, the resolution of your service metrics, on the update and create calls, or `$.stopType`, a rollback that can revert that resolution, on `StopServiceDeployment`. Watching those calls helps address the mitigation by surfacing attempts to stop high-resolution service metrics for your service.

### Implementation.ECS.Service.27

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].serviceArn`, `$.services[*].status`, `$.services[*].currentServiceRevisions[*].arn`
4. Call `ecs:DescribeServiceRevisions` with `$.serviceRevisionArns[*]`; check `$.serviceRevisions[*].serviceArn`, `$.serviceRevisions[*].monitoring.metricConfigurations[*].resolutionSeconds`, `$.serviceRevisions[*].monitoring.metricConfigurations[*].metricNames[*]`

**Implements:** [Mitigation.ECS.Service.10](#mitigationecsservice10)

**Scoping** (for [Mitigation.ECS.Service.10](#mitigationecsservice10)): This implementation guidance recommends reading `$.serviceRevisions[*].monitoring.metricConfigurations[*].resolutionSeconds`, the resolution at which your service metrics are collected, and `$.serviceRevisions[*].monitoring.metricConfigurations[*].metricNames[*]`, the metrics that the resolution applies to. Flagging any Amazon ECS service that has no monitoring configuration or that specifies a 60-second resolution helps address the mitigation by identifying where high-resolution `CPUUtilization` and `MemoryUtilization` service metrics are not collected.

### Implementation.ECS.Service.28

*Guidance — Directive*

Configure the security groups in your Amazon ECS service's awsvpc network configuration following least-privilege best practices. Restrict both inbound and outbound rules to only the sources and destinations, protocols, and ports your tasks require. Sources and destinations can be IP address ranges, referenced security groups, or prefix lists. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change. Avoid rules that permit unrestricted access from or to any address, such as 0.0.0.0/0 or ::/0, particularly on administrative or sensitive ports.

To govern security groups at scale, use AWS Firewall Manager security group policies across your organization in AWS Organizations. You can scope each policy by account, organizational unit, or tag. A common security group policy applies a baseline group to in-scope resources. Firewall Manager can update your service's task network interfaces only when the service uses the rolling update (Amazon ECS) deployment controller. It doesn't support updating security groups on network interfaces created for Fargate tasks. A content audit policy audits inbound and outbound rules against an allow-list or a block-list. It flags noncompliant groups in a central dashboard, sends notifications, and can automatically remediate them. A usage audit policy identifies unused and redundant groups and, with automatic remediation enabled, can consolidate redundant groups and delete unused ones.

To verify unintended access permitted by your security groups, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended. For example, match any path that reaches administrative ports 22 or 3389 from an unrestricted address. Then run an analysis with `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`, and review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.ECS.Service.11](#mitigationecsservice11)

**Scoping** (for [Mitigation.ECS.Service.11](#mitigationecsservice11)): This implementation guidance recommends restricting the inbound and outbound rules on your Amazon ECS service's security groups to the required addresses, protocols, and ports. Excluding unrestricted addresses such as 0.0.0.0/0 or ::/0 from those inbound and outbound rules helps reduce the network exposure of the tasks your service maintains.

### Implementation.ECS.Service.29

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::Service`: `$.NetworkConfiguration.AwsvpcConfiguration.SecurityGroups[*]`

**Implements:** [Mitigation.ECS.Service.11](#mitigationecsservice11)

**Scoping** (for [Mitigation.ECS.Service.11](#mitigationecsservice11)): This implementation guidance recommends validating the inbound and outbound rules of the security groups resolved from `$.NetworkConfiguration.AwsvpcConfiguration.SecurityGroups[*]` before provisioning. Checking for rules that allow any address on unintended ports, directly or through a prefix list, helps prevent an Amazon ECS service from being provisioned with such groups.

### Implementation.ECS.Service.30

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeServices` with `$.cluster`, `$.services[*]`; check `$.services[*].networkConfiguration.awsvpcConfiguration.securityGroups[*]`
4. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
5. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.ECS.Service.11](#mitigationecsservice11)

**Scoping** (for [Mitigation.ECS.Service.11](#mitigationecsservice11)): This implementation guidance recommends resolving the security groups recorded at `$.services[*].networkConfiguration.awsvpcConfiguration.securityGroups[*]` and inspecting their inbound and outbound rules. Flagging rules that admit 0.0.0.0/0 or ::/0 on all protocols helps you detect your Amazon ECS services with broader network exposure than they need.

### Implementation.ECS.Task.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalTag/dp:exclude:network`, `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.ECS.Task.1](#mitigationecstask1)

**Scoping** (for [Mitigation.ECS.Task.1](#mitigationecstask1)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon ECS task's task IAM role and task execution IAM role credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Data perimeter** (for [Mitigation.ECS.Task.1](#mitigationecstask1)): The default network perimeter SCP samples apply the restriction only to principals carrying the `aws:PrincipalTag/dp:include:network` tag. Tag both your Amazon ECS task role and task execution role so the restriction reaches calls made with their credentials. Tagging the task execution role still matters because the Amazon ECS container agent or Fargate agent uses its credentials from your expected networks, and the restriction helps prevent their use elsewhere if they are obtained. If either role assumes another role during its work, apply the same network perimeter restriction to that assumed role.
- [Network perimeter SCP sample with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)
- [Network perimeter SCP sample with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [Network perimeter guidance for the SCP examples](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/README.md#sidenforcenetworkperimetervpceorgid)

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalTag/tag-key` to apply the restriction to principals whose credentials can be used outside your expected networks, tagged with the selected key and value. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)

### Implementation.ECS.Task.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListTasks` with `$.cluster`; check `$.taskArns[*]`
3. Call `ecs:DescribeTasks` with `$.cluster`, `$.tasks[*]`; check `$.tasks[*].overrides.taskRoleArn`, `$.tasks[*].overrides.executionRoleArn`, `$.tasks[*].attachments[*].type`, `$.tasks[*].attachments[*].details[*].name`, `$.tasks[*].attachments[*].details[*].value`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
7. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.ECS.Task.2](#mitigationecstask2)

**Scoping** (for [Mitigation.ECS.Task.2](#mitigationecstask2)): This implementation guidance recommends reading `$.tasks[*].overrides.taskRoleArn`, `$.tasks[*].overrides.executionRoleArn`, and `$.tasks[*].attachments[*].details[*].value`, your task's task role override, execution role override, and infrastructure role, then evaluating `$.Role.AssumeRolePolicyDocument`, each role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys on your task role and execution role, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.Task.3

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs-tasks.amazonaws.com` | `sts:AssumeRole` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.ECS.Task.2](#mitigationecstask2)

**Scoping** (for [Mitigation.ECS.Task.2](#mitigationecstask2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS task's task execution IAM role, `$.overrides.executionRoleArn`, to the `ecs-tasks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon ECS from assuming the role on behalf of Amazon ECS resources in an account outside the sources you specify.

**Scoping** (for [Mitigation.ECS.Task.2](#mitigationecstask2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS task's task role, `$.overrides.taskRoleArn`, to the `ecs-tasks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon ECS from assuming the role on behalf of Amazon ECS resources in accounts outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.ECS.Task.4

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.ECS.Task.2](#mitigationecstask2)

**Scoping** (for [Mitigation.ECS.Task.2](#mitigationecstask2)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS task's infrastructure role, `$.volumeConfigurations[*].managedEBSVolume.roleArn`, to the `ecs.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.ECS.Task.5

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.ECS.Task.2](#mitigationecstask2)

**Scoping** (for [Mitigation.ECS.Task.2](#mitigationecstask2)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon ECS task's task role, execution role, and infrastructure role, `$.overrides.taskRoleArn`, `$.overrides.executionRoleArn`, and `$.volumeConfigurations[*].managedEBSVolume.roleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.ECS.Task.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListTasks` with `$.cluster`; check `$.taskArns[*]`
3. Call `ecs:DescribeTasks` with `$.cluster`, `$.tasks[*]`; check `$.tasks[*].taskArn`, `$.tasks[*].attachments[*].type`, `$.tasks[*].attachments[*].status`, `$.tasks[*].attachments[*].details[*].name`, `$.tasks[*].attachments[*].details[*].value`
4. Call `ec2:DescribeVolumes` with `$.VolumeId[*]`; check `$.volumeSet[*].volumeId`, `$.volumeSet[*].encrypted`, `$.volumeSet[*].kmsKeyId`

**Implements:** [Mitigation.ECS.Task.3](#mitigationecstask3)

**Scoping** (for [Mitigation.ECS.Task.3](#mitigationecstask3)): This implementation guidance recommends flagging an Amazon ECS task whose `$.volumeSet[*].encrypted`, the encryption setting of its attached Amazon EBS volume, is false. Surfacing those tasks helps you detect where data on their attached Amazon EBS volumes is stored unencrypted.

### Implementation.ECS.Task.7

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListTasks` with `$.cluster`, `$.desiredStatus` = `RUNNING`; check `$.taskArns[*]`
3. Call `ecs:DescribeTasks` with `$.cluster`, `$.tasks[*]`; check `$.tasks[*].attachments[*].type` = `ElasticNetworkInterface`, `$.tasks[*].attachments[*].details[*].name` = `subnetId`, `$.tasks[*].attachments[*].details[*].value`
4. Call `ec2:DescribeRouteTables` with `$.Filters`; check `$.RouteTables[*].Routes[*].GatewayId`, `$.RouteTables[*].Routes[*].DestinationCidrBlock` = `0.0.0.0/0`

**Implements:** [Mitigation.ECS.Task.4](#mitigationecstask4)

**Scoping** (for [Mitigation.ECS.Task.4](#mitigationecstask4)): This implementation guidance recommends reading `$.tasks[*].attachments[*].details[*].value`, the subnet of each task's network interface, then resolving each subnet's route table. Flagging an Amazon ECS task with a subnet whose default route targets an internet gateway, read at `$.RouteTables[*].Routes[*].GatewayId`, helps you detect placement in a public subnet.

### Implementation.ECS.Task.8

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListTasks` with `$.cluster`; check `$.taskArns[*]`
3. Call `ecs:DescribeTasks` with `$.cluster`, `$.tasks[*]`; check `$.tasks[*].taskArn`, `$.tasks[*].attachments[*].type` = `AmazonElasticBlockStorage`, `$.tasks[*].attachments[*].details[*].name`, `$.tasks[*].attachments[*].details[*].value`
4. Call `ec2:DescribeVolumes` with `$.VolumeId[*]`; check `$.volumeSet[*].attachmentSet[*].associatedResource`, `$.volumeSet[*].kmsKeyId`
5. Call `kms:DescribeKey` with `$.KeyId`; check `$.KeyMetadata.KeyManager` = `CUSTOMER`, `$.KeyMetadata.AWSAccountId`, `$.KeyMetadata.Origin`

**Implements:** [Mitigation.ECS.Task.5](#mitigationecstask5)

**Scoping** (for [Mitigation.ECS.Task.5](#mitigationecstask5)): This implementation guidance recommends reading `$.volumeSet[*].kmsKeyId`, the key that encrypts the Amazon EBS volume attached to your task, and resolving `$.KeyMetadata.KeyManager`, `$.KeyMetadata.AWSAccountId`, and `$.KeyMetadata.Origin` for that key. Flagging Amazon ECS tasks whose key is not a customer managed key in your account, or whose key material is imported or held in an external key store, helps you detect where you do not control the key policy, lifecycle, or key material.

### Implementation.ECS.Task.9

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ec2:CreateVolume` | `arn:aws:ec2:*:*:volume/*` | `aws:RequestTag/AmazonECSCreated`, `aws:RequestTag/AmazonECSManaged`, `ec2:KmsKeyId` |

**Implements:** [Mitigation.ECS.Task.5](#mitigationecstask5)

**Scoping** (for [Mitigation.ECS.Task.5](#mitigationecstask5)): This implementation guidance recommends applying the `ec2:KmsKeyId` condition key in a service control policy (SCP) to govern `ec2:CreateVolume`, scoped by `aws:RequestTag/AmazonECSCreated` and `aws:RequestTag/AmazonECSManaged`. This condition helps prevent creating an Amazon ECS task's managed EBS volume without an approved customer managed key at `$.volumeConfigurations[*].managedEBSVolume.kmsKeyId`, the key that encrypts the volume.

### Implementation.ECS.Task.10

*Guidance — Directive*

Launch your Amazon ECS tasks with two or more subnets that resolve to different Availability Zones. Specify those subnet IDs in the `awsvpcConfiguration` subnet array when you call `RunTask` or `StartTask`, or on the service that launches the task. A task ENI exists only when the task uses the `awsvpc` network mode. Amazon ECS creates the task elastic network interface (ENI) in one of the subnets you supply. Supplying subnets from several zones therefore lets Amazon ECS spread your task ENIs across those zones. Network connectivity for your tasks can then survive the loss of a single zone.

Subnet-to-AZ membership is a property of each subnet, not of the task. Each subnet resides entirely within one Availability Zone and cannot span zones. Before you supply subnet IDs, confirm that they belong to distinct Availability Zones by calling `DescribeSubnets` and reading the `AvailabilityZone` response field. Standardize on infrastructure-as-code review for the templates and modules that launch your tasks. Confirm that each one supplies at least two subnets from different Availability Zones.

To verify the deployed state, enumerate the task ENIs by calling `DescribeNetworkInterfaces`. You can also call `DescribeTasks`, which reports the subnet ID in the elastic network interface attachment details. That response also returns the task's Availability Zone. Confirm that the subnets of the attached ENIs span more than one Availability Zone.

To assess and track multi-AZ posture at scale, use AWS Resilience Hub. Include your Amazon ECS application resources in a Resilience Hub application and run a resiliency assessment. For tasks that use the Amazon EC2 or AWS Fargate launch type, the assessment flags single-AZ concentration against your resiliency policy.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.ECS.Task.6](#mitigationecstask6)

**Scoping** (for [Mitigation.ECS.Task.6](#mitigationecstask6)): This implementation guidance recommends supplying two or more subnets from different Availability Zones in `$.networkConfiguration.awsvpcConfiguration.subnets[*]`, the subnet list of your task's `awsvpc` network configuration. Spreading the task elastic network interfaces that Amazon ECS creates across zones helps task network connectivity survive the loss of a single Availability Zone.

### Implementation.ECS.Task.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListTasks` with `$.cluster`; check `$.taskArns[*]`
3. Call `ecs:DescribeTasks` with `$.cluster`, `$.tasks[*]`, `$.include[*]` = `TAGS`; check `$.tasks[*].taskArn`, `$.tasks[*].tags[*].key`, `$.tasks[*].tags[*].value`

**Implements:** [Mitigation.ECS.Task.7](#mitigationecstask7)

**Scoping** (for [Mitigation.ECS.Task.7](#mitigationecstask7)): This implementation guidance recommends reading `$.tasks[*].tags[*].key` on each Amazon ECS task listed through `$.taskArns[*]` and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS task to an accountable owner and the business function it supports.

### Implementation.ECS.Task.12

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:RunTask`, `ecs:StartTask` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:task/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:task/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.Task.7](#mitigationecstask7)

**Scoping** (for [Mitigation.ECS.Task.7](#mitigationecstask7)): This implementation guidance recommends restricting `ecs:RunTask` and `ecs:StartTask` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS task without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.Task.13

*Guidance — Directive*

Configure the security groups associated with your Amazon ECS task following least-privilege best practices. Restrict both inbound and outbound rules to only the sources and destinations, protocols, and ports your task requires. Specify sources and destinations as IP address ranges, referenced security groups, or prefix lists. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change. In `awsvpc` network mode, these rules apply to the elastic network interface attached to the task.

To govern security groups at scale, use AWS Firewall Manager security group policies across an AWS Organization, scoped by account, organizational unit, or tag. A common security group policy applies a baseline group. Firewall Manager can update task network interfaces only for Amazon ECS services that use the rolling update (`ECS`) deployment controller. It can update them for tasks that use the EC2 launch type, but not for tasks on AWS Fargate. As a result, a common policy can't apply its baseline group to Fargate tasks, standalone tasks, or tasks under other deployment controllers. A content audit policy audits inbound and outbound rules against an allow-list or block-list. It flags non-compliant groups in a central dashboard and sends notifications. It can also automatically remediate them. A usage audit policy removes unused or redundant groups. Consolidating redundant groups changes their associations, so the same task network interface limits apply.

To verify unintended access permitted by your security groups, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended. For example, describe any path that reaches administrative ports 22 or 3389 from an unrestricted address. Then run an analysis with `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`, and review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.ECS.Task.8](#mitigationecstask8)

**Scoping** (for [Mitigation.ECS.Task.8](#mitigationecstask8)): This implementation guidance recommends restricting the inbound and outbound rules on the security groups in `$.networkConfiguration.awsvpcConfiguration.securityGroups[*]` to the required addresses, protocols, and ports. Excluding unrestricted addresses such as 0.0.0.0/0 or ::/0 from those inbound and outbound rules helps reduce the network exposure of your Amazon ECS task.

### Implementation.ECS.Task.14

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListTasks` with `$.cluster`; check `$.taskArns[*]`
3. Call `ecs:DescribeTasks` with `$.cluster`, `$.tasks[*]`; check `$.tasks[*].attachments[*].type` = `ElasticNetworkInterface`, `$.tasks[*].attachments[*].details[*].name` = `networkInterfaceId`, `$.tasks[*].attachments[*].details[*].value`
4. Call `ec2:DescribeNetworkInterfaces` with `$.NetworkInterfaceId[*]`; check `$.networkInterfaceSet[*].groupSet[*].groupId`
5. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
6. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.ECS.Task.8](#mitigationecstask8)

**Scoping** (for [Mitigation.ECS.Task.8](#mitigationecstask8)): This implementation guidance recommends resolving the security groups behind the interface at `$.tasks[*].attachments[*].details[*].value` and inspecting their inbound and outbound rules. Flagging rules that admit 0.0.0.0/0 or ::/0 on sensitive ports or all protocols helps you detect your Amazon ECS tasks with broader network exposure than they need.

### Implementation.ECS.TaskDefinition.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskDefinition`: `$.ContainerDefinitions[*].LogConfiguration.LogDriver`, `$.ContainerDefinitions[*].LogConfiguration.Options{key}`

**Implements:** [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)

**Scoping** (for [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)): This implementation guidance recommends validating that `$.ContainerDefinitions[*].LogConfiguration.LogDriver`, each container's log driver, is `awslogs` and that `$.ContainerDefinitions[*].LogConfiguration.Options{key}`, the log driver options, name the destination log group and its Region before provisioning. That check helps prevent an Amazon ECS task definition from being provisioned without configured container logging to CloudWatch Logs, which lets you review container output and detect errors.

### Implementation.ECS.TaskDefinition.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`, `$.logStreamName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `ecs:DeleteTaskDefinitions` — `$.taskDefinitions[*]`
- `ecs:DeregisterTaskDefinition` — `$.taskDefinition`
- `ecs:PutAccountSetting` — `$.name` = `defaultLogDriverMode`, `$.value`
- `ecs:PutAccountSettingDefault` — `$.name` = `defaultLogDriverMode`, `$.value`
- `ecs:RegisterTaskDefinition` — `$.containerDefinitions[*].logConfiguration.logDriver`, `$.containerDefinitions[*].logConfiguration.options{key}`, `$.family`
- `ecs:UpdateService` — `$.taskDefinition`

**Implements:** [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)

**Scoping** (for [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)): This implementation guidance recommends recording `RegisterTaskDefinition`, which sets `$.containerDefinitions[*].logConfiguration.logDriver`, each container's log driver, and `$.containerDefinitions[*].logConfiguration.options{key}`, the driver options, including the destination log group. Recording these calls alongside `DeleteTaskDefinitions` and the CloudWatch Logs operations `DeleteLogGroup`, `DeleteLogStream`, and `PutRetentionPolicy` helps you detect changes that disable or weaken container logging or erase its log records.

### Implementation.ECS.TaskDefinition.3

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::TaskDefinition`: `$.configuration.ContainerDefinitions[*].LogConfiguration.LogDriver`, `$.configuration.ContainerDefinitions[*].LogConfiguration.Options{key}`, `$.configuration.Status`

**Implements:** [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)

**Scoping** (for [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)): This implementation guidance recommends reading `$.configuration.ContainerDefinitions[*].LogConfiguration.LogDriver` (the log driver of each container definition), `$.configuration.ContainerDefinitions[*].LogConfiguration.Options{key}` (the log driver options, including the destination log group), and `$.configuration.Status` (the status of the task definition revision). Flagging active task definition revisions with a container definition that lacks the `awslogs` log driver or an `awslogs-group` option helps you detect gaps in container logging to CloudWatch Logs.

### Implementation.ECS.TaskDefinition.4

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalTag/dp:exclude:network`, `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.ECS.TaskDefinition.2](#mitigationecstaskdefinition2)

**Scoping** (for [Mitigation.ECS.TaskDefinition.2](#mitigationecstaskdefinition2)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:include:network`, `aws:SourceIp`, and `aws:SourceVpc` condition keys in a service control policy (SCP), pairing `aws:SourceVpc` with `aws:RequestedRegion`. These conditions help restrict requests made with your Amazon ECS task definition's task IAM role and task execution role credentials to expected networks, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Data perimeter** (for [Mitigation.ECS.TaskDefinition.2](#mitigationecstaskdefinition2)): Amazon ECS has no service-specific condition key that identifies calls made with task role credentials. Instead, tag your task role and task execution role with the `dp:include:network` tag key. The policy then uses `aws:PrincipalTag/dp:include:network` to reach only the roles you bring inside the network perimeter. If either role assumes another role, apply the same network perimeter restriction to that assumed role.
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [Network perimeter guidance for the data perimeter SCP examples](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/README.md#sidenforcenetworkperimetervpceorgid)

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalTag/tag-key` to apply the restriction to principals whose credentials can be used outside your expected networks, tagged with the selected key and value. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy with aws:VpceOrgID](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_vpceorgid_scp.json)
- [SCP sample: network perimeter policy with aws:SourceVpc](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/network_perimeter_sourcevpc_scp.json)

### Implementation.ECS.TaskDefinition.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskDefinition`: `$.TaskRoleArn`, `$.ExecutionRoleArn`

**Implements:** [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)

**Scoping** (for [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)): This implementation guidance recommends validating the trust policies of `$.TaskRoleArn` and `$.ExecutionRoleArn`, your task definition's task role and task execution role, including source conditions on service principals, before provisioning. This helps prevent the task definition from being provisioned when any trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.

The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.TaskDefinition.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListTaskDefinitions`; check `$.taskDefinitionArns[*]`
2. Call `ecs:DescribeTaskDefinition` with `$.taskDefinition`; check `$.taskDefinition.taskRoleArn`, `$.taskDefinition.executionRoleArn`
3. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result` = `PASS`
5. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result` = `PASS`

**Implements:** [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)

**Scoping** (for [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)): This implementation guidance recommends reading `$.taskDefinition.taskRoleArn` and `$.taskDefinition.executionRoleArn`, your task definition's task role and task execution role, then evaluating `$.Role.AssumeRolePolicyDocument`, each role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, or that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain. The first reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository. The second reference policy for `CheckNoNewAccess` encodes the source conditions you accept when the service principal assumes the role: one role-assumption statement per accepted combination of the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys, with placeholders for your account, organization, and resource values. The check fails when the trust policy grants role assumption that no statement of the reference allows, including grants that omit the condition keys or weaken them with `IfExists`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.TaskDefinition.7

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs-tasks.amazonaws.com` | `sts:AssumeRole` | `aws:SourceAccount` (alternatives: `aws:SourceOrgID`, `aws:SourceOrgPaths`), `aws:SourceArn` |

**Implements:** [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)

**Scoping** (for [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS task definition's task IAM role, `$.taskRoleArn`, to the `ecs-tasks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon ECS from assuming the role on behalf of tasks outside the sources you specify.

**Scoping** (for [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS task definition's task execution role, `$.executionRoleArn`, to the `ecs-tasks.amazonaws.com` service principal with the `aws:SourceArn` and `aws:SourceAccount` condition keys. These conditions help prevent Amazon ECS from assuming the role on behalf of tasks outside the sources you specify.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.ECS.TaskDefinition.8

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)

**Scoping** (for [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon ECS task definition's task role and task execution role, `$.taskRoleArn` and `$.executionRoleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.ECS.TaskDefinition.9

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskDefinition`: `$.ContainerDefinitions[*].Name`, `$.ContainerDefinitions[*].Image`, `$.ContainerDefinitions[*].Command[*]`, `$.ContainerDefinitions[*].Environment[*].Name`, `$.ContainerDefinitions[*].Environment[*].Value`, `$.TaskRoleArn`

**Implements:** [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4)

**Scoping** (for [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4)): This implementation guidance recommends validating that `$.ContainerDefinitions[*].Name` (the container name), `$.ContainerDefinitions[*].Image` (the container image), `$.ContainerDefinitions[*].Command[*]` (the container command arguments), `$.ContainerDefinitions[*].Environment[*].Name` (the environment variable names), `$.ContainerDefinitions[*].Environment[*].Value` (their values), and `$.TaskRoleArn` (the task IAM role) declare an AWS Distro for OpenTelemetry collector sidecar that uses the Amazon Managed Service for Prometheus collector configuration, the `AWS_PROMETHEUS_ENDPOINT` remote write endpoint, and a task IAM role before provisioning. That validation helps prevent an Amazon ECS task definition without an AWS Distro for OpenTelemetry (ADOT) sidecar container for metric collection from being provisioned.

### Implementation.ECS.TaskDefinition.10

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `ecs:DeleteTaskDefinitions` — `$.taskDefinitions[*]`
- `ecs:DeregisterTaskDefinition` — `$.taskDefinition`
- `ecs:RegisterTaskDefinition` — `$.containerDefinitions[*].command[*]`, `$.containerDefinitions[*].environment[*].name`, `$.containerDefinitions[*].environment[*].value`, `$.containerDefinitions[*].image`, `$.containerDefinitions[*].name`, `$.family`, `$.taskRoleArn`
- `ecs:RunTask` — `$.overrides.containerOverrides[*].command[*]`, `$.overrides.containerOverrides[*].environment[*].name`, `$.overrides.containerOverrides[*].environment[*].value`, `$.overrides.containerOverrides[*].name`, `$.overrides.taskRoleArn`, `$.taskDefinition`
- `ecs:StartTask` — `$.overrides.containerOverrides[*].command[*]`, `$.overrides.containerOverrides[*].environment[*].name`, `$.overrides.containerOverrides[*].environment[*].value`, `$.overrides.containerOverrides[*].name`, `$.overrides.taskRoleArn`, `$.taskDefinition`
- `ecs:UpdateService` — `$.taskDefinition`

**Implements:** [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4)

**Scoping** (for [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4)): This implementation guidance recommends recording the `RegisterTaskDefinition`, `DeregisterTaskDefinition`, `DeleteTaskDefinitions`, `UpdateService`, `RunTask`, and `StartTask` calls on your Amazon ECS task definition that remove the AWS Distro for OpenTelemetry (ADOT) sidecar container for metric collection or replace `$.containerDefinitions[*].command[*]`, the argument that selects the collector configuration, or `$.taskRoleArn`, the task IAM role the collector uses to export metrics to Amazon Managed Service for Prometheus, including through the `$.overrides.containerOverrides[*].command[*]` and `$.overrides.taskRoleArn` launch-time overrides. Recording those calls and their calling identity helps you detect a change that stops the detailed telemetry collected outside Amazon CloudWatch Metrics.

### Implementation.ECS.TaskDefinition.11

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::TaskDefinition`: `$.configuration.ContainerDefinitions[*].Name`, `$.configuration.ContainerDefinitions[*].Image`, `$.configuration.ContainerDefinitions[*].Command[*]`, `$.configuration.ContainerDefinitions[*].Environment[*].Name`, `$.configuration.ContainerDefinitions[*].Environment[*].Value`, `$.configuration.TaskRoleArn`, `$.configuration.Status`

**Implements:** [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4)

**Scoping** (for [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4)): This implementation guidance recommends flagging each Amazon ECS task definition whose `$.configuration.Status`, the task definition status, is `ACTIVE` and whose `$.configuration.ContainerDefinitions[*].Name`, the container name, `$.configuration.ContainerDefinitions[*].Image`, the container image, `$.configuration.ContainerDefinitions[*].Command[*]`, the container command arguments, `$.configuration.ContainerDefinitions[*].Environment[*].Name`, the environment variable name, `$.configuration.ContainerDefinitions[*].Environment[*].Value`, its value, and `$.configuration.TaskRoleArn`, the task IAM role, together declare no AWS Distro for OpenTelemetry (ADOT) sidecar container that exports metrics to Amazon Managed Service for Prometheus. Flagging them helps you detect resources whose recorded configuration does not enable an ADOT sidecar container for metric collection.

### Implementation.ECS.TaskDefinition.12

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskDefinition`: `$.Volumes[*].EFSVolumeConfiguration.TransitEncryption`

**Implements:** [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5)

**Scoping** (for [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5)): This implementation guidance recommends validating that `$.Volumes[*].EFSVolumeConfiguration.TransitEncryption`, the transit encryption setting of each Amazon EFS volume, is `ENABLED` before provisioning an Amazon ECS task definition. This check helps prevent an Amazon ECS task definition from being provisioned with an Amazon EFS volume whose omitted or `DISABLED` setting leaves its NFS traffic in cleartext.

### Implementation.ECS.TaskDefinition.13

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::TaskDefinition`: `$.configuration.Volumes[*].EfsVolumeConfiguration.TransitEncryption`, `$.configuration.Volumes[*].EfsVolumeConfiguration.TransitEncryptionPort`, `$.configuration.Volumes[*].EfsVolumeConfiguration.FileSystemId`, `$.configuration.Volumes[*].Name`, `$.configuration.Status`

**Implements:** [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5)

**Scoping** (for [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5)): This implementation guidance recommends reading `$.configuration.Volumes[*].EfsVolumeConfiguration.TransitEncryption` (the transit encryption setting), `$.configuration.Volumes[*].EfsVolumeConfiguration.TransitEncryptionPort` (the transit encryption port), `$.configuration.Volumes[*].EfsVolumeConfiguration.FileSystemId` (the Amazon EFS file system ID), `$.configuration.Volumes[*].Name` (the volume name), and `$.configuration.Status` (the revision status) in the configuration items of your Amazon ECS task definitions. Flagging Amazon ECS task definitions whose Amazon EFS volumes set transit encryption to `DISABLED` or omit the setting helps you detect NFS traffic that travels between the Amazon ECS host and the Amazon EFS server without TLS protection.

### Implementation.ECS.TaskDefinition.14

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskDefinition`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6)

**Scoping** (for [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon ECS task definition that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.ECS.TaskDefinition.15

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::TaskDefinition`: `$.tags[*].key`, `$.tags[*].value`, `$.configuration.Tags[*].Key`, `$.configuration.Tags[*].Value`

**Implements:** [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6)

**Scoping** (for [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6)): This implementation guidance recommends reading `$.tags[*].key` and `$.configuration.Tags[*].Key` on each task definition and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each task definition to an accountable owner and the business function it supports.

### Implementation.ECS.TaskDefinition.16

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:RegisterTaskDefinition` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:task-definition/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:task-definition/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6)

**Scoping** (for [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6)): This implementation guidance recommends restricting `ecs:RegisterTaskDefinition` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS task definition without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.TaskSet.1

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeTaskSets` with `$.cluster`, `$.service`; check `$.taskSets[*].loadBalancers[*].advancedConfiguration.roleArn`
4. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
5. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
6. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1)

**Scoping** (for [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1)): This implementation guidance recommends reading `$.taskSets[*].loadBalancers[*].advancedConfiguration.roleArn`, your task set's infrastructure role for load balancers, then evaluating `$.Role.AssumeRolePolicyDocument`, that role's trust policy, with IAM Access Analyzer. Flagging trust policies that allow public role assumption, or assumption beyond your reference policy, helps you detect overly permissive role assumption.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.ECS.TaskSet.2

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `ecs.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1)

**Scoping** (for [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1)): This implementation guidance recommends a trust policy that restricts assumption of your Amazon ECS task set's infrastructure role for load balancers, `$.loadBalancers[*].advancedConfiguration.roleArn`, to the `ecs.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.ECS.TaskSet.3

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1)

**Scoping** (for [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming your Amazon ECS task set's infrastructure role for load balancers, `$.loadBalancers[*].advancedConfiguration.roleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.ECS.TaskSet.4

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskSet`: `$.NetworkConfiguration.AwsVpcConfiguration.Subnets[*]`, `$.NetworkConfiguration.AwsVpcConfiguration.AssignPublicIp`

**Implements:** [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2)

**Scoping** (for [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2)): This implementation guidance recommends validating `$.NetworkConfiguration.AwsVpcConfiguration.Subnets[*]`, the subnets hosting each task's elastic network interface, by resolving each subnet's route table for a default route to an internet gateway before provisioning. Subnets carrying that route fail validation, which helps prevent an Amazon ECS task set from being provisioned in a public subnet.

### Implementation.ECS.TaskSet.5

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeTaskSets` with `$.cluster`, `$.service`; check `$.taskSets[*].networkConfiguration.awsvpcConfiguration.subnets[*]`, `$.taskSets[*].networkConfiguration.awsvpcConfiguration.assignPublicIp`
4. Call `ec2:DescribeRouteTables` with `$.Filters`; check `$.RouteTables[*].Routes[*].GatewayId`, `$.RouteTables[*].Routes[*].DestinationCidrBlock` = `0.0.0.0/0`, `$.RouteTables[*].Routes[*].DestinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2)

**Scoping** (for [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2)): This implementation guidance recommends reading `$.taskSets[*].networkConfiguration.awsvpcConfiguration.subnets[*]`, the subnets holding each task's network interface, then resolving each subnet's route table. Flagging an Amazon ECS task set with a subnet whose default route targets an internet gateway, read at `$.RouteTables[*].Routes[*].GatewayId`, helps you detect placement in a public subnet.

### Implementation.ECS.TaskSet.6

*Guidance — Directive*

Configure each Amazon ECS task set with two or more subnets that resolve to different Availability Zones. Supply those subnet IDs in the `awsvpc` network configuration subnet array when you create the task set. The task elastic network interfaces (ENIs) that Amazon ECS creates then exist in more than one Availability Zone. Network connectivity for the task set can therefore survive the loss of a single zone.

Subnet-to-AZ membership is a property of each subnet, not of the task set. Before you supply the subnet IDs, confirm that they belong to distinct Availability Zones. Call the Amazon EC2 `DescribeSubnets` operation and read the `AvailabilityZone` response field. A task set's network configuration cannot be changed after creation, so correcting it requires a replacement task set. Standardize on infrastructure-as-code review for task set creation. Confirm that the templates and modules supply at least two subnets from different Availability Zones.

To verify the deployed state, enumerate the task network interfaces with the Amazon EC2 `DescribeNetworkInterfaces` operation. Confirm that the attached ENIs' subnets span more than one Availability Zone. You can also call `DescribeTaskSets`, which returns the network configuration recorded for the task set.

To assess and track multi-AZ posture at scale, use AWS Resilience Hub. Include the task set's application resources in a Resilience Hub application and run a resiliency assessment. The assessment flags single-AZ concentration against your resiliency policy.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3)

**Scoping** (for [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3)): This implementation guidance recommends supplying two or more subnets from different Availability Zones in each Amazon ECS task set's `awsvpc` network configuration. Spreading the elastic network interfaces that Amazon ECS creates for the task set's tasks across zones helps the tasks' network connectivity survive the loss of a single Availability Zone.

### Implementation.ECS.TaskSet.7

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskSet`: `$.NetworkConfiguration.AwsVpcConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3)

**Scoping** (for [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3)): This implementation guidance recommends validating, before provisioning, that `$.NetworkConfiguration.AwsVpcConfiguration.Subnets[*]`, your task set's VPC subnets, lists two or more subnet entries. That check helps prevent a single-subnet task set from being provisioned, so your tasks' elastic network interfaces can span more than one Availability Zone.

### Implementation.ECS.TaskSet.8

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::TaskSet`: `$.configuration.NetworkConfiguration.AwsVpcConfiguration.Subnets[*]`

**Implements:** [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3)

**Scoping** (for [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3)): This implementation guidance recommends reading `$.configuration.NetworkConfiguration.AwsVpcConfiguration.Subnets[*]`, the subnets in which your task set launches its tasks, then resolving each subnet's Availability Zone. Flagging an awsvpc task set confined to a single Availability Zone helps you detect its tasks' elastic network interfaces that lack multi-zone redundancy.

### Implementation.ECS.TaskSet.9

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskSet`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4)

**Scoping** (for [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4)): This implementation guidance recommends validating, before provisioning, that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose tags. Requiring those tags helps prevent an Amazon ECS task set that omits them from being provisioned and helps keep ownership and business function attributable.

### Implementation.ECS.TaskSet.10

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::ECS::TaskSet`: `$.tags[*].key`, `$.tags[*].value`

**Implements:** [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4)

**Scoping** (for [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4)): This implementation guidance recommends reading `$.tags[*].key` on each Amazon ECS task set and flagging any that omit your required owner and purpose tag keys. Surfacing those resources helps you attribute each Amazon ECS task set to an accountable owner and the business function it supports.

### Implementation.ECS.TaskSet.11

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `ecs:CreateTaskSet` | `*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:TagResource` | `arn:aws:ecs:*:*:task-set/*/*/*` | `aws:RequestTag/owner`, `aws:RequestTag/purpose`, `aws:TagKeys` |
| `ecs:UntagResource` | `arn:aws:ecs:*:*:task-set/*/*/*` | `aws:TagKeys` |

**Implements:** [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4)

**Scoping** (for [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4)): This implementation guidance recommends restricting `ecs:CreateTaskSet` and `ecs:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys in a service control policy (SCP). These conditions help prevent creating or tagging an Amazon ECS task set without the owner and purpose tag keys, or removing them through `ecs:UntagResource`.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.ECS.TaskSet.12

*Guidance — Directive*

Configure the security groups associated with your Amazon ECS task set following least-privilege best practices. Restrict both inbound and outbound rules to only the sources and destinations, protocols, and ports that your tasks require. Specify those sources and destinations as IP address ranges, referenced security groups, or prefix lists. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change.

To govern security groups at scale, use AWS Firewall Manager security group policies across your organization in AWS Organizations. You can scope each policy by account, organizational unit, or tag. A common security group policy applies a baseline security group to the resources in scope. Firewall Manager currently can't update the network interfaces of tasks in Amazon ECS services that use the `CODE_DEPLOY` or `EXTERNAL` deployment controller. Because task sets run only in those services, a common security group policy can't apply its baseline security group to your task set's tasks. A content audit security group policy audits inbound and outbound rules against an allow-list or a block-list. It flags non-compliant security groups in a central dashboard, notifies you, and can automatically remediate them. A usage audit security group policy removes unused or redundant security groups.

To verify unintended access permitted by your security groups, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended. For example, describe any path that reaches administrative ports 22 or 3389 from an unrestricted address. Then run an analysis with `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`, and review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)

**Scoping** (for [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)): This implementation guidance recommends restricting the inbound and outbound rules on your Amazon ECS task set's security groups to the required addresses, protocols, and ports. Excluding unrestricted addresses such as 0.0.0.0/0 or ::/0 from those inbound and outbound rules helps reduce the network exposure of your Amazon ECS task set.

### Implementation.ECS.TaskSet.13

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::ECS::TaskSet`: `$.NetworkConfiguration.AwsVpcConfiguration.SecurityGroups[*]`

**Implements:** [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)

**Scoping** (for [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)): This implementation guidance recommends validating the inbound and outbound rules of the security groups resolved from `$.NetworkConfiguration.AwsVpcConfiguration.SecurityGroups[*]` before provisioning. Checking for rules that allow any address on unintended ports, directly or through a prefix list, helps prevent an Amazon ECS task set from being provisioned with such groups.

### Implementation.ECS.TaskSet.14

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `ecs:ListClusters`; check `$.clusterArns[*]`
2. Call `ecs:ListServices` with `$.cluster`; check `$.serviceArns[*]`
3. Call `ecs:DescribeTaskSets` with `$.cluster`, `$.service`; check `$.taskSets[*].taskSetArn`, `$.taskSets[*].networkConfiguration.awsvpcConfiguration.securityGroups[*]`
4. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
5. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)

**Scoping** (for [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5)): This implementation guidance recommends resolving the security groups recorded at `$.taskSets[*].networkConfiguration.awsvpcConfiguration.securityGroups[*]` and inspecting their inbound and outbound rules. Flagging rules that admit 0.0.0.0/0 or ::/0 on sensitive ports, broad port ranges, or all protocols helps you detect your task sets with broader network exposure than they need.

### ManagedControl.ECS.AccountSetting.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon Elastic Container Service clusters have container insights enabled

**Aliases:** `CONFIG.ECS.DT.11`

**Identifier:** `ECS_CONTAINER_INSIGHTS_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/ees96sz1zdqikhefz9zgips8c`

Checks if Amazon Elastic Container Service clusters have container insights enabled. The rule is NON_COMPLIANT if container insights are not enabled.

**Implements:** [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1)

### ManagedControl.ECS.CapacityProvider.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon ECS capacity providers have tags

**Aliases:** `CONFIG.ECS.DT.10`

**Identifier:** `ECS_CAPACITY_PROVIDER_TAGGED`

**ARN:** `arn:aws:controlcatalog:::control/5epkqrbdshw8vq9zcqhk9mz29`

Checks if Amazon ECS capacity providers have tags. Optionally, you can specify tag keys. The rule is NON_COMPLIANT if there are no tags or if the specified tag keys are not present. The rule does not check for tags starting with 'aws:'.

**Implements:** [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6)

### ManagedControl.ECS.Cluster.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon Elastic Container Service clusters have container insights enabled

**Aliases:** `CONFIG.ECS.DT.11`

**Identifier:** `ECS_CONTAINER_INSIGHTS_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/ees96sz1zdqikhefz9zgips8c`

Checks if Amazon Elastic Container Service clusters have container insights enabled. The rule is NON_COMPLIANT if container insights are not enabled.

**Implements:** [Mitigation.ECS.Cluster.5](#mitigationecscluster5)

### ManagedControl.ECS.Service.1

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require Amazon ECS services not to assign public IP addresses automatically

**Aliases:** `CT.ECS.PR.9`

**ARN:** `arn:aws:controlcatalog:::control/dpj7wsjmmm2xnxkh1d273lyqh`

This control checks whether your Amazon ECS services are configured to assign public IP addresses automatically.

**Implements:** [Mitigation.ECS.Service.4](#mitigationecsservice4)

### ManagedControl.ECS.TaskDefinition.1

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require an active Amazon ECS task definition to have a logging configuration

**Aliases:** `CT.ECS.PR.5`

**ARN:** `arn:aws:controlcatalog:::control/2nnlov4vuktsxvs8fpugcmc32`

This control checks whether Amazon Elastic Container Service (ECS) task definitions have a logging configuration specified.

**Implements:** [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)

### ManagedControl.ECS.TaskDefinition.2

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if logConfiguration is set on active ECS Task Definitions

**Aliases:** `CONFIG.ECS.DT.4`

**Identifier:** `ECS_TASK_DEFINITION_LOG_CONFIGURATION`

**ARN:** `arn:aws:controlcatalog:::control/dgygjy058dqn2hfzjgzzcoojt`

Checks if logConfiguration is set on active ECS Task Definitions. This rule is NON_COMPLIANT if an active ECSTaskDefinition does not have the logConfiguration resource defined or the value for logConfiguration is null in at least one container definition.

**Implements:** [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1)

### ManagedControl.ECS.TaskDefinition.3

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon ECS Task Definitions with EFS volumes have in-transit encryption enabled

**Aliases:** `CONFIG.ECS.DT.18`

**Identifier:** `ECS_TASK_DEFINITION_EFS_ENCRYPTION_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/915h84d0twsu7ueq4b7mie282`

Checks if Amazon ECS Task Definitions with EFS volumes have in-transit encryption enabled. The rule is NON_COMPLIANT if an ECS Task Definition contains an EFS volume without transit encryption enabled.

**Implements:** [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5)

---

## Appendix

### Resources

#### AccountSetting

*CONFIGURATION*

Amazon ECS account settings are named switches that turn Amazon ECS features on or off for your account. Each setting holds a single value. Settings apply per AWS Region, so you configure them separately in every Region you use.

- 4 operations, 2 threats, 1 mitigations

#### CapacityProvider

*CONFIGURATION*

An Amazon ECS capacity provider defines where and how the containers in a cluster run. It tells the Amazon ECS scheduler which pool of compute to draw from. It also governs how that pool grows and shrinks as workloads change.

- IAM resource ARN: `arn:${Partition}:ecs:${Region}:${Account}:capacity-provider/${CapacityProviderName}`
- CloudFormation: `AWS::ECS::CapacityProvider`
- AWS Config: `AWS::ECS::CapacityProvider`
- 7 operations, 11 threats, 8 mitigations

#### Cluster

*RESOURCE_GROUPING*

An Amazon ECS cluster is a Region-scoped grouping that provides infrastructure capacity for containerized applications. It is the boundary within which your container workloads are placed and run. Creating separate clusters lets you keep unrelated workloads apart and scope permissions to them.

- IAM resource ARN: `arn:${Partition}:ecs:${Region}:${Account}:cluster/${ClusterName}`
- CloudFormation: `AWS::ECS::Cluster`, `AWS::ECS::ClusterCapacityProviderAssociations`
- AWS Config: `AWS::ECS::Cluster`
- 17 operations, 8 threats, 5 mitigations

#### ContainerInstance

*COMPUTE_RESOURCE*

An Amazon ECS container instance is a compute host registered to an Amazon ECS cluster. It is an Amazon EC2 instance, or an on-premises server or virtual machine, running the Amazon ECS container agent. The agent connects the host to the Amazon ECS control plane. It reports available CPU, memory, and ports, and starts and stops containers for the scheduler. Registered instances provide the capacity where tasks and services run.

- IAM resource ARN: `arn:${Partition}:ecs:${Region}:${Account}:container-instance/${ClusterName}/${ContainerInstanceId}`
- CloudFormation: `AWS::ECS::ContainerInstance`
- 13 operations, 1 threats, 1 mitigations

#### Daemon

*COMPUTE_RESOURCE*

An Amazon ECS daemon deploys and maintains a cross-cutting software agent across your container infrastructure. It targets one or more Amazon ECS Managed Instances capacity providers in a cluster, and Amazon ECS places exactly one daemon task on every instance those providers launch. Agents for security monitoring, observability, logging, tracing, and networking are typical uses.

- IAM resource ARNs: `arn:${Partition}:ecs:${Region}:${Account}:daemon/${ClusterName}/${DaemonName}`, `arn:${Partition}:ecs:${Region}:${Account}:daemon-deployment/${ClusterName}/${DaemonName}/${DaemonDeploymentId}`, `arn:${Partition}:ecs:${Region}:${Account}:daemon-revision/${ClusterName}/${DaemonName}/${DaemonRevisionId}`
- CloudFormation: `AWS::ECS::Daemon`
- 10 operations, 1 threats, 1 mitigations

#### DaemonTaskDefinition

*CONFIGURATION*

An Amazon ECS daemon task definition is the blueprint for a managed daemon. It describes the container images, CPU and memory reservations, IAM roles, host mounts, and logging configuration for cross-cutting agents. These agents typically perform security monitoring, telemetry collection, observability, and log forwarding across container infrastructure.

- IAM resource ARN: `arn:${Partition}:ecs:${Region}:${Account}:daemon-task-definition/${DaemonTaskDefinitionFamilyName}:${DaemonTaskDefinitionRevisionNumber}`
- CloudFormation: `AWS::ECS::DaemonTaskDefinition`
- 6 operations, 5 threats, 4 mitigations

#### ExpressGatewayService

*COMPUTE_RESOURCE*

An Amazon ECS Express Mode service deploys a containerized web application on AWS Fargate using production-ready defaults. You supply a container image plus a task execution role and an infrastructure role. Amazon ECS then provisions and configures the supporting infrastructure on your behalf. This includes a cluster and service, an Application Load Balancer with an HTTPS listener and certificate, target groups, security groups, auto scaling, and a log group.

- IAM resource ARN: `arn:${Partition}:ecs:${Region}:${Account}:service/${ClusterName}/${ServiceName}`
- CloudFormation: `AWS::ECS::ExpressGatewayService`
- AWS Config: `AWS::ECS::Service`
- 4 operations, 11 threats, 7 mitigations

#### Service

*COMPUTE_RESOURCE*

An Amazon ECS service runs and maintains a chosen number of copies of a task definition inside an ECS cluster. When a task stops or is reported unhealthy, the service scheduler launches a replacement, so the workload keeps running without manual intervention. Services suit long-running application components such as web front ends, APIs, and background workers.

- IAM resource ARNs: `arn:${Partition}:ecs:${Region}:${Account}:service/${ClusterName}/${ServiceName}`, `arn:${Partition}:ecs:${Region}:${Account}:service-deployment/${ClusterName}/${ServiceName}/${ServiceDeploymentId}`, `arn:${Partition}:ecs:${Region}:${Account}:service-revision/${ClusterName}/${ServiceName}/${ServiceRevisionId}`
- CloudFormation: `AWS::ECS::Service`, `AWS::ECS::PrimaryTaskSet`
- AWS Config: `AWS::ECS::Service`
- 15 operations, 20 threats, 11 mitigations

#### Task

*COMPUTE_RESOURCE*

An Amazon ECS task is a running instantiation of a task definition inside a cluster. It launches one or more containers together as a single unit of work. Tasks run on the capacity you choose: serverless Fargate, Amazon EC2 instances, ECS Managed Instances, or registered on-premises servers.

- IAM resource ARN: `arn:${Partition}:ecs:${Region}:${Account}:task/${ClusterName}/${TaskId}`
- CloudFormation: `AWS::ECS::Task`
- 11 operations, 14 threats, 8 mitigations

#### TaskDefinition

*CONFIGURATION*

An Amazon ECS task definition is the blueprint for a containerized application. It is a JSON document that declares one or more containers and the resources they need. Amazon ECS uses it as the template whenever it launches tasks or services.

- IAM resource ARN: `arn:${Partition}:ecs:${Region}:${Account}:task-definition/${TaskDefinitionFamilyName}:${TaskDefinitionRevisionNumber}`
- CloudFormation: `AWS::ECS::TaskDefinition`
- AWS Config: `AWS::ECS::TaskDefinition`
- 9 operations, 9 threats, 6 mitigations

#### TaskSet

*COMPUTE_RESOURCE*

An Amazon ECS task set is a deployment unit inside an Amazon ECS service. It groups tasks that all run the same task definition, and it carries its own launch type or capacity provider strategy, platform version, network configuration, load balancer targets, and service discovery registrations.

- IAM resource ARN: `arn:${Partition}:ecs:${Region}:${Account}:task-set/${ClusterName}/${ServiceName}/${TaskSetId}`
- CloudFormation: `AWS::ECS::TaskSet`
- AWS Config: `AWS::ECS::TaskSet`
- 7 operations, 9 threats, 5 mitigations

### Operations

| Operation | Analyses | Resources | Mitigations |
|---|---|---|---|
| `ContinueServiceDeployment` | DOCUMENTATION | [Service](#service) |  |
| `CreateCapacityProvider` | DOCUMENTATION | [CapacityProvider](#capacityprovider) | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1), [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2), [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3), [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4), [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5), [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6), [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7), [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8) |
| `CreateCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1), [Mitigation.ECS.Cluster.1](#mitigationecscluster1), [Mitigation.ECS.Cluster.2](#mitigationecscluster2), [Mitigation.ECS.Cluster.3](#mitigationecscluster3), [Mitigation.ECS.Cluster.4](#mitigationecscluster4), [Mitigation.ECS.Cluster.5](#mitigationecscluster5) |
| `CreateDaemon` | DOCUMENTATION | [Daemon](#daemon) | [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1), [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1) |
| `CreateExpressGatewayService` | DOCUMENTATION | [ExpressGatewayService](#expressgatewayservice) | [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1), [Mitigation.ECS.ExpressGatewayService.2](#mitigationecsexpressgatewayservice2), [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3), [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4), [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5), [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6), [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7) |
| `CreateService` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) | [Mitigation.ECS.Service.1](#mitigationecsservice1), [Mitigation.ECS.Service.2](#mitigationecsservice2), [Mitigation.ECS.Service.3](#mitigationecsservice3), [Mitigation.ECS.Service.4](#mitigationecsservice4), [Mitigation.ECS.Service.5](#mitigationecsservice5), [Mitigation.ECS.Service.6](#mitigationecsservice6), [Mitigation.ECS.Service.7](#mitigationecsservice7), [Mitigation.ECS.Service.8](#mitigationecsservice8), [Mitigation.ECS.Service.9](#mitigationecsservice9), [Mitigation.ECS.Service.10](#mitigationecsservice10), [Mitigation.ECS.Service.11](#mitigationecsservice11), [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1) |
| `CreateTaskSet` | DOCUMENTATION, DATA_PERIMETER | [TaskSet](#taskset) | [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1), [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2), [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3), [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4), [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5) |
| `DeleteAccountSetting` | DOCUMENTATION, DATA_PERIMETER | [AccountSetting](#accountsetting) | [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1) |
| `DeleteAttributes` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster), [ContainerInstance](#containerinstance) |  |
| `DeleteCapacityProvider` | DOCUMENTATION, DATA_PERIMETER | [CapacityProvider](#capacityprovider) |  |
| `DeleteCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `DeleteDaemon` | DOCUMENTATION | [Daemon](#daemon) |  |
| `DeleteDaemonTaskDefinition` | DOCUMENTATION | [DaemonTaskDefinition](#daemontaskdefinition) |  |
| `DeleteExpressGatewayService` | DOCUMENTATION | [ExpressGatewayService](#expressgatewayservice) |  |
| `DeleteService` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) |  |
| `DeleteTaskDefinitions` | DOCUMENTATION, DATA_PERIMETER | [TaskDefinition](#taskdefinition) |  |
| `DeleteTaskSet` | DOCUMENTATION, DATA_PERIMETER | [TaskSet](#taskset) |  |
| `DeregisterContainerInstance` | DOCUMENTATION, DATA_PERIMETER | [ContainerInstance](#containerinstance) |  |
| `DeregisterTaskDefinition` | DOCUMENTATION, DATA_PERIMETER | [TaskDefinition](#taskdefinition) | [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4) |
| `DescribeCapacityProviders` | DOCUMENTATION, DATA_PERIMETER | [CapacityProvider](#capacityprovider) | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1), [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2), [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3), [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4), [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5), [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6), [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7), [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8) |
| `DescribeClusters` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.ECS.Cluster.1](#mitigationecscluster1), [Mitigation.ECS.Cluster.2](#mitigationecscluster2), [Mitigation.ECS.Cluster.3](#mitigationecscluster3), [Mitigation.ECS.Cluster.4](#mitigationecscluster4), [Mitigation.ECS.Cluster.5](#mitigationecscluster5) |
| `DescribeContainerInstances` | DOCUMENTATION, DATA_PERIMETER | [ContainerInstance](#containerinstance) | [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1) |
| `DescribeDaemon` | DOCUMENTATION | [Daemon](#daemon) | [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1) |
| `DescribeDaemonDeployments` | DOCUMENTATION | [Daemon](#daemon) |  |
| `DescribeDaemonRevisions` | DOCUMENTATION | [Daemon](#daemon) |  |
| `DescribeDaemonTaskDefinition` | DOCUMENTATION | [DaemonTaskDefinition](#daemontaskdefinition) | [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1), [Mitigation.ECS.DaemonTaskDefinition.2](#mitigationecsdaemontaskdefinition2), [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3) |
| `DescribeExpressGatewayService` | DOCUMENTATION | [ExpressGatewayService](#expressgatewayservice) | [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1), [Mitigation.ECS.ExpressGatewayService.2](#mitigationecsexpressgatewayservice2), [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3), [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4), [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5), [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6), [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7) |
| `DescribeServiceDeployments` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) | [Mitigation.ECS.Service.2](#mitigationecsservice2) |
| `DescribeServiceRevisions` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) | [Mitigation.ECS.Service.1](#mitigationecsservice1), [Mitigation.ECS.Service.2](#mitigationecsservice2), [Mitigation.ECS.Service.3](#mitigationecsservice3), [Mitigation.ECS.Service.4](#mitigationecsservice4), [Mitigation.ECS.Service.5](#mitigationecsservice5), [Mitigation.ECS.Service.6](#mitigationecsservice6), [Mitigation.ECS.Service.7](#mitigationecsservice7), [Mitigation.ECS.Service.8](#mitigationecsservice8), [Mitigation.ECS.Service.10](#mitigationecsservice10), [Mitigation.ECS.Service.11](#mitigationecsservice11) |
| `DescribeServices` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) | [Mitigation.ECS.Service.1](#mitigationecsservice1), [Mitigation.ECS.Service.2](#mitigationecsservice2), [Mitigation.ECS.Service.3](#mitigationecsservice3), [Mitigation.ECS.Service.4](#mitigationecsservice4), [Mitigation.ECS.Service.5](#mitigationecsservice5), [Mitigation.ECS.Service.6](#mitigationecsservice6), [Mitigation.ECS.Service.7](#mitigationecsservice7), [Mitigation.ECS.Service.8](#mitigationecsservice8), [Mitigation.ECS.Service.9](#mitigationecsservice9), [Mitigation.ECS.Service.10](#mitigationecsservice10), [Mitigation.ECS.Service.11](#mitigationecsservice11) |
| `DescribeTaskDefinition` | DOCUMENTATION, DATA_PERIMETER | [TaskDefinition](#taskdefinition) | [Mitigation.ECS.1](#mitigationecs1), [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1), [Mitigation.ECS.TaskDefinition.2](#mitigationecstaskdefinition2), [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3), [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4), [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5), [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6) |
| `DescribeTaskSets` | DOCUMENTATION, DATA_PERIMETER | [TaskSet](#taskset) | [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1), [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2), [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3), [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4), [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5) |
| `DescribeTasks` | DOCUMENTATION, DATA_PERIMETER | [Task](#task) | [Mitigation.ECS.Task.1](#mitigationecstask1), [Mitigation.ECS.Task.2](#mitigationecstask2), [Mitigation.ECS.Task.3](#mitigationecstask3), [Mitigation.ECS.Task.4](#mitigationecstask4), [Mitigation.ECS.Task.5](#mitigationecstask5), [Mitigation.ECS.Task.6](#mitigationecstask6), [Mitigation.ECS.Task.7](#mitigationecstask7), [Mitigation.ECS.Task.8](#mitigationecstask8) |
| `DiscoverPollEndpoint` | DOCUMENTATION, DATA_PERIMETER | [ContainerInstance](#containerinstance) | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1) |
| `ExecuteCommand` | DOCUMENTATION | [Cluster](#cluster), [Task](#task) | [Mitigation.ECS.1](#mitigationecs1), [Mitigation.ECS.2](#mitigationecs2), [Mitigation.ECS.Cluster.1](#mitigationecscluster1), [Mitigation.ECS.Task.4](#mitigationecstask4), [Mitigation.ECS.Task.8](#mitigationecstask8), [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1) |
| `GetTaskProtection` | DOCUMENTATION, DATA_PERIMETER | [Task](#task) |  |
| `ListAccountSettings` | DOCUMENTATION, DATA_PERIMETER | [AccountSetting](#accountsetting) | [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1) |
| `ListAttributes` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster), [ContainerInstance](#containerinstance) |  |
| `ListClusters` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `ListContainerInstances` | DOCUMENTATION, DATA_PERIMETER | [ContainerInstance](#containerinstance) |  |
| `ListDaemonDeployments` | DOCUMENTATION | [Daemon](#daemon) |  |
| `ListDaemonTaskDefinitions` | DOCUMENTATION | [DaemonTaskDefinition](#daemontaskdefinition) | [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1) |
| `ListDaemons` | DOCUMENTATION | [Daemon](#daemon) | [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1) |
| `ListServiceDeployments` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) |  |
| `ListServices` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) |  |
| `ListServicesByNamespace` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) |  |
| `ListTagsForResource` | DOCUMENTATION, DATA_PERIMETER | [CapacityProvider](#capacityprovider), [Cluster](#cluster), [ContainerInstance](#containerinstance), [Service](#service), [Task](#task), [TaskDefinition](#taskdefinition), [TaskSet](#taskset) | [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6), [Mitigation.ECS.Cluster.3](#mitigationecscluster3), [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1), [Mitigation.ECS.Service.9](#mitigationecsservice9), [Mitigation.ECS.Task.7](#mitigationecstask7), [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6), [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4) |
| `ListTaskDefinitionFamilies` | DOCUMENTATION, DATA_PERIMETER | [TaskDefinition](#taskdefinition) |  |
| `ListTaskDefinitions` | DOCUMENTATION, DATA_PERIMETER | [TaskDefinition](#taskdefinition) |  |
| `ListTasks` | DOCUMENTATION, DATA_PERIMETER | [Task](#task) | [Mitigation.ECS.Task.5](#mitigationecstask5), [Mitigation.ECS.Task.6](#mitigationecstask6) |
| `PutAccountSetting` | DOCUMENTATION, DATA_PERIMETER | [AccountSetting](#accountsetting) | [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1), [Mitigation.ECS.Cluster.5](#mitigationecscluster5) |
| `PutAccountSettingDefault` | DOCUMENTATION, DATA_PERIMETER | [AccountSetting](#accountsetting) | [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1), [Mitigation.ECS.Cluster.5](#mitigationecscluster5) |
| `PutAttributes` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster), [ContainerInstance](#containerinstance) | [Mitigation.ECS.1](#mitigationecs1) |
| `PutClusterCapacityProviders` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) |  |
| `RegisterContainerInstance` | DOCUMENTATION, DATA_PERIMETER | [ContainerInstance](#containerinstance) | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1), [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1) |
| `RegisterDaemonTaskDefinition` | DOCUMENTATION | [DaemonTaskDefinition](#daemontaskdefinition) | [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1), [Mitigation.ECS.DaemonTaskDefinition.2](#mitigationecsdaemontaskdefinition2), [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3), [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4) |
| `RegisterTaskDefinition` | DOCUMENTATION, DATA_PERIMETER | [TaskDefinition](#taskdefinition) | [Mitigation.ECS.1](#mitigationecs1), [Mitigation.ECS.2](#mitigationecs2), [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1), [Mitigation.ECS.TaskDefinition.2](#mitigationecstaskdefinition2), [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3), [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4), [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5), [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6) |
| `RunTask` | DOCUMENTATION, DATA_PERIMETER | [Task](#task) | [Mitigation.ECS.1](#mitigationecs1), [Mitigation.ECS.2](#mitigationecs2), [Mitigation.ECS.Task.1](#mitigationecstask1), [Mitigation.ECS.Task.2](#mitigationecstask2), [Mitigation.ECS.Task.3](#mitigationecstask3), [Mitigation.ECS.Task.4](#mitigationecstask4), [Mitigation.ECS.Task.5](#mitigationecstask5), [Mitigation.ECS.Task.6](#mitigationecstask6), [Mitigation.ECS.Task.7](#mitigationecstask7), [Mitigation.ECS.Task.8](#mitigationecstask8), [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1), [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4) |
| `StartTask` | DOCUMENTATION, DATA_PERIMETER | [Task](#task) | [Mitigation.ECS.Task.1](#mitigationecstask1), [Mitigation.ECS.Task.2](#mitigationecstask2), [Mitigation.ECS.Task.3](#mitigationecstask3), [Mitigation.ECS.Task.4](#mitigationecstask4), [Mitigation.ECS.Task.5](#mitigationecstask5), [Mitigation.ECS.Task.6](#mitigationecstask6), [Mitigation.ECS.Task.7](#mitigationecstask7), [Mitigation.ECS.Task.8](#mitigationecstask8), [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1) |
| `StopServiceDeployment` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) | [Mitigation.ECS.Service.1](#mitigationecsservice1) |
| `StopTask` | DOCUMENTATION, DATA_PERIMETER | [Task](#task) |  |
| `SubmitAttachmentStateChanges` | DOCUMENTATION | [Cluster](#cluster) | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1) |
| `SubmitContainerStateChange` | DOCUMENTATION | [Cluster](#cluster) |  |
| `SubmitTaskStateChange` | DOCUMENTATION | [Cluster](#cluster) | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1) |
| `TagResource` | DOCUMENTATION, DATA_PERIMETER | [CapacityProvider](#capacityprovider), [Cluster](#cluster), [ContainerInstance](#containerinstance), [Daemon](#daemon), [DaemonTaskDefinition](#daemontaskdefinition), [Service](#service), [Task](#task), [TaskDefinition](#taskdefinition), [TaskSet](#taskset) | [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6), [Mitigation.ECS.Cluster.3](#mitigationecscluster3), [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1), [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1), [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4), [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6), [Mitigation.ECS.Service.9](#mitigationecsservice9), [Mitigation.ECS.Task.7](#mitigationecstask7), [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6), [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4) |
| `UntagResource` | DOCUMENTATION, DATA_PERIMETER | [CapacityProvider](#capacityprovider), [Cluster](#cluster), [ContainerInstance](#containerinstance), [Daemon](#daemon), [DaemonTaskDefinition](#daemontaskdefinition), [Service](#service), [Task](#task), [TaskDefinition](#taskdefinition), [TaskSet](#taskset) | [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6), [Mitigation.ECS.Cluster.3](#mitigationecscluster3), [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1), [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4), [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6), [Mitigation.ECS.Service.9](#mitigationecsservice9), [Mitigation.ECS.Task.7](#mitigationecstask7), [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6), [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4) |
| `UpdateCapacityProvider` | DOCUMENTATION | [CapacityProvider](#capacityprovider) | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1), [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2), [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3), [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4), [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5), [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6), [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7), [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8) |
| `UpdateCluster` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.ECS.Cluster.1](#mitigationecscluster1), [Mitigation.ECS.Cluster.2](#mitigationecscluster2), [Mitigation.ECS.Cluster.4](#mitigationecscluster4), [Mitigation.ECS.Cluster.5](#mitigationecscluster5) |
| `UpdateClusterSettings` | DOCUMENTATION, DATA_PERIMETER | [Cluster](#cluster) | [Mitigation.ECS.Cluster.5](#mitigationecscluster5) |
| `UpdateContainerAgent` | DOCUMENTATION, DATA_PERIMETER | [ContainerInstance](#containerinstance) |  |
| `UpdateContainerInstancesState` | DOCUMENTATION, DATA_PERIMETER | [ContainerInstance](#containerinstance) |  |
| `UpdateDaemon` | DOCUMENTATION | [Daemon](#daemon) | [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1), [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1) |
| `UpdateExpressGatewayService` | DOCUMENTATION | [ExpressGatewayService](#expressgatewayservice) | [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1), [Mitigation.ECS.ExpressGatewayService.2](#mitigationecsexpressgatewayservice2), [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3), [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4), [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5), [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7) |
| `UpdateService` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) | [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5), [Mitigation.ECS.Service.1](#mitigationecsservice1), [Mitigation.ECS.Service.2](#mitigationecsservice2), [Mitigation.ECS.Service.3](#mitigationecsservice3), [Mitigation.ECS.Service.4](#mitigationecsservice4), [Mitigation.ECS.Service.5](#mitigationecsservice5), [Mitigation.ECS.Service.6](#mitigationecsservice6), [Mitigation.ECS.Service.7](#mitigationecsservice7), [Mitigation.ECS.Service.8](#mitigationecsservice8), [Mitigation.ECS.Service.10](#mitigationecsservice10), [Mitigation.ECS.Service.11](#mitigationecsservice11), [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1), [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4) |
| `UpdateServicePrimaryTaskSet` | DOCUMENTATION, DATA_PERIMETER | [Service](#service) |  |
| `UpdateTaskProtection` | DOCUMENTATION, DATA_PERIMETER | [Task](#task) | [Mitigation.ECS.Task.1](#mitigationecstask1) |
| `UpdateTaskSet` | DOCUMENTATION, DATA_PERIMETER | [TaskSet](#taskset) |  |

Mitigations attach to an operation directly (the operation appears in the mitigation's request parameters) or through a threat whose request parameters include the operation.

### Goal Coverage (17 of 38)

| Mitigation Goal | Covered | Mitigations |
|---|---|---|
| [AI Services Opt-Out Policy](../mitigation-goals.md#ai-services-opt-out-policy) | — |  |
| [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup) | — |  |
| [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native) | — |  |
| [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery) | yes | [Mitigation.ECS.Cluster.1](#mitigationecscluster1), [Mitigation.ECS.DaemonTaskDefinition.1](#mitigationecsdaemontaskdefinition1), [Mitigation.ECS.ExpressGatewayService.1](#mitigationecsexpressgatewayservice1), [Mitigation.ECS.Service.1](#mitigationecsservice1), [Mitigation.ECS.TaskDefinition.1](#mitigationecstaskdefinition1) |
| [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2) | — |  |
| [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest) | yes | [Mitigation.ECS.Service.3](#mitigationecsservice3), [Mitigation.ECS.Task.3](#mitigationecstask3) |
| [Cross-Region Replication](../mitigation-goals.md#cross-region-replication) | — |  |
| [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies) | — |  |
| [Customer Code Execution - Govern Code Content](../mitigation-goals.md#customer-code-execution---govern-code-content) | — |  |
| [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage) | yes | [Mitigation.ECS.Cluster.4](#mitigationecscluster4) |
| [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection) | — |  |
| [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics) | yes | [Mitigation.ECS.AccountSetting.1](#mitigationecsaccountsetting1), [Mitigation.ECS.CapacityProvider.7](#mitigationecscapacityprovider7), [Mitigation.ECS.Cluster.5](#mitigationecscluster5), [Mitigation.ECS.Service.10](#mitigationecsservice10) |
| [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics) | yes | [Mitigation.ECS.TaskDefinition.4](#mitigationecstaskdefinition4) |
| [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy) | — |  |
| [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature) | yes | [Mitigation.ECS.Service.8](#mitigationecsservice8), [Mitigation.ECS.TaskDefinition.5](#mitigationecstaskdefinition5) |
| [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication) | — |  |
| [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network) | yes | [Mitigation.ECS.2](#mitigationecs2) |
| [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources) | — |  |
| [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption) | yes | [Mitigation.ECS.Cluster.2](#mitigationecscluster2), [Mitigation.ECS.Service.6](#mitigationecsservice6), [Mitigation.ECS.Task.5](#mitigationecstask5) |
| [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment) | yes | [Mitigation.ECS.Service.5](#mitigationecsservice5) |
| [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment) | yes | [Mitigation.ECS.CapacityProvider.5](#mitigationecscapacityprovider5), [Mitigation.ECS.ExpressGatewayService.5](#mitigationecsexpressgatewayservice5), [Mitigation.ECS.Service.7](#mitigationecsservice7), [Mitigation.ECS.Task.6](#mitigationecstask6), [Mitigation.ECS.TaskSet.3](#mitigationecstaskset3) |
| [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks) | yes | [Mitigation.ECS.CapacityProvider.1](#mitigationecscapacityprovider1), [Mitigation.ECS.DaemonTaskDefinition.2](#mitigationecsdaemontaskdefinition2), [Mitigation.ECS.ExpressGatewayService.2](#mitigationecsexpressgatewayservice2), [Mitigation.ECS.Task.1](#mitigationecstask1), [Mitigation.ECS.TaskDefinition.2](#mitigationecstaskdefinition2) |
| [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks) | — |  |
| [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement) | yes | [Mitigation.ECS.CapacityProvider.3](#mitigationecscapacityprovider3), [Mitigation.ECS.ExpressGatewayService.4](#mitigationecsexpressgatewayservice4), [Mitigation.ECS.Service.4](#mitigationecsservice4), [Mitigation.ECS.Task.4](#mitigationecstask4), [Mitigation.ECS.TaskSet.2](#mitigationecstaskset2) |
| [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets) | — |  |
| [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources) | — |  |
| [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network) | yes | [Mitigation.ECS.1](#mitigationecs1) |
| [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition) | — |  |
| [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention) | — |  |
| [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose) | yes | [Mitigation.ECS.CapacityProvider.6](#mitigationecscapacityprovider6), [Mitigation.ECS.Cluster.3](#mitigationecscluster3), [Mitigation.ECS.ContainerInstance.1](#mitigationecscontainerinstance1), [Mitigation.ECS.Daemon.1](#mitigationecsdaemon1), [Mitigation.ECS.DaemonTaskDefinition.4](#mitigationecsdaemontaskdefinition4), [Mitigation.ECS.ExpressGatewayService.6](#mitigationecsexpressgatewayservice6), [Mitigation.ECS.Service.9](#mitigationecsservice9), [Mitigation.ECS.Task.7](#mitigationecstask7), [Mitigation.ECS.TaskDefinition.6](#mitigationecstaskdefinition6), [Mitigation.ECS.TaskSet.4](#mitigationecstaskset4) |
| [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role) | yes | [Mitigation.ECS.CapacityProvider.2](#mitigationecscapacityprovider2), [Mitigation.ECS.DaemonTaskDefinition.3](#mitigationecsdaemontaskdefinition3), [Mitigation.ECS.ExpressGatewayService.3](#mitigationecsexpressgatewayservice3), [Mitigation.ECS.Service.2](#mitigationecsservice2), [Mitigation.ECS.Task.2](#mitigationecstask2), [Mitigation.ECS.TaskDefinition.3](#mitigationecstaskdefinition3), [Mitigation.ECS.TaskSet.1](#mitigationecstaskset1) |
| [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access) | yes | [Mitigation.ECS.CapacityProvider.8](#mitigationecscapacityprovider8), [Mitigation.ECS.ExpressGatewayService.7](#mitigationecsexpressgatewayservice7), [Mitigation.ECS.Service.11](#mitigationecsservice11), [Mitigation.ECS.Task.8](#mitigationecstask8), [Mitigation.ECS.TaskSet.5](#mitigationecstaskset5) |
| [Service Principal Presigned URL](../mitigation-goals.md#service-principal-presigned-url) | — |  |
| [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration) | — |  |
| [Turn on Block Public Access (BPA)](../mitigation-goals.md#turn-on-block-public-access-bpa) | — |  |
| [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention) | — |  |
| [Use Nitro instances](../mitigation-goals.md#use-nitro-instances) | yes | [Mitigation.ECS.CapacityProvider.4](#mitigationecscapacityprovider4) |
| [VPC Deployment](../mitigation-goals.md#vpc-deployment) | — |  |

### Impact Rollup

| Impact | Threats |
|---|---|
| [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery) | [Threat.ECS.CapacityProvider.5](#threatecscapacityprovider5), [Threat.ECS.CapacityProvider.7](#threatecscapacityprovider7), [Threat.ECS.ExpressGatewayService.4](#threatecsexpressgatewayservice4), [Threat.ECS.ExpressGatewayService.5](#threatecsexpressgatewayservice5), [Threat.ECS.Service.10](#threatecsservice10), [Threat.ECS.Service.11](#threatecsservice11), [Threat.ECS.Service.15](#threatecsservice15), [Threat.ECS.Service.18](#threatecsservice18), [Threat.ECS.Task.8](#threatecstask8), [Threat.ECS.Task.9](#threatecstask9), [Threat.ECS.TaskDefinition.7](#threatecstaskdefinition7), [Threat.ECS.TaskDefinition.9](#threatecstaskdefinition9), [Threat.ECS.TaskSet.4](#threatecstaskset4), [Threat.ECS.TaskSet.5](#threatecstaskset5) |
| [Data Collection](../threat-impacts.md#data-collection) | [Threat.ECS.CapacityProvider.6](#threatecscapacityprovider6), [Threat.ECS.CapacityProvider.7](#threatecscapacityprovider7), [Threat.ECS.CapacityProvider.8](#threatecscapacityprovider8), [Threat.ECS.Cluster.4](#threatecscluster4), [Threat.ECS.Cluster.6](#threatecscluster6), [Threat.ECS.DaemonTaskDefinition.2](#threatecsdaemontaskdefinition2), [Threat.ECS.DaemonTaskDefinition.3](#threatecsdaemontaskdefinition3), [Threat.ECS.ExpressGatewayService.6](#threatecsexpressgatewayservice6), [Threat.ECS.ExpressGatewayService.7](#threatecsexpressgatewayservice7), [Threat.ECS.Service.4](#threatecsservice4), [Threat.ECS.Service.6](#threatecsservice6), [Threat.ECS.Service.7](#threatecsservice7), [Threat.ECS.Service.12](#threatecsservice12), [Threat.ECS.Service.14](#threatecsservice14), [Threat.ECS.Service.15](#threatecsservice15), [Threat.ECS.Service.18](#threatecsservice18), [Threat.ECS.Task.3](#threatecstask3), [Threat.ECS.Task.4](#threatecstask4), [Threat.ECS.Task.5](#threatecstask5), [Threat.ECS.Task.10](#threatecstask10), [Threat.ECS.Task.11](#threatecstask11), [Threat.ECS.TaskDefinition.4](#threatecstaskdefinition4), [Threat.ECS.TaskDefinition.5](#threatecstaskdefinition5), [Threat.ECS.TaskDefinition.7](#threatecstaskdefinition7), [Threat.ECS.TaskDefinition.9](#threatecstaskdefinition9) |
| [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Threat.ECS.Cluster.2](#threatecscluster2), [Threat.ECS.Service.2](#threatecsservice2), [Threat.ECS.Task.1](#threatecstask1) |
| [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Threat.ECS.1](#threatecs1), [Threat.ECS.2](#threatecs2), [Threat.ECS.CapacityProvider.8](#threatecscapacityprovider8), [Threat.ECS.CapacityProvider.9](#threatecscapacityprovider9), [Threat.ECS.CapacityProvider.10](#threatecscapacityprovider10), [Threat.ECS.Cluster.6](#threatecscluster6), [Threat.ECS.DaemonTaskDefinition.2](#threatecsdaemontaskdefinition2), [Threat.ECS.DaemonTaskDefinition.3](#threatecsdaemontaskdefinition3), [Threat.ECS.ExpressGatewayService.6](#threatecsexpressgatewayservice6), [Threat.ECS.ExpressGatewayService.7](#threatecsexpressgatewayservice7), [Threat.ECS.ExpressGatewayService.9](#threatecsexpressgatewayservice9), [Threat.ECS.ExpressGatewayService.10](#threatecsexpressgatewayservice10), [Threat.ECS.Service.6](#threatecsservice6), [Threat.ECS.Service.7](#threatecsservice7), [Threat.ECS.Service.12](#threatecsservice12), [Threat.ECS.Service.14](#threatecsservice14), [Threat.ECS.Service.17](#threatecsservice17), [Threat.ECS.Service.19](#threatecsservice19), [Threat.ECS.Task.4](#threatecstask4), [Threat.ECS.Task.5](#threatecstask5), [Threat.ECS.Task.10](#threatecstask10), [Threat.ECS.Task.11](#threatecstask11), [Threat.ECS.Task.12](#threatecstask12), [Threat.ECS.Task.13](#threatecstask13), [Threat.ECS.TaskDefinition.4](#threatecstaskdefinition4), [Threat.ECS.TaskDefinition.5](#threatecstaskdefinition5), [Threat.ECS.TaskSet.6](#threatecstaskset6), [Threat.ECS.TaskSet.7](#threatecstaskset7), [Threat.ECS.TaskSet.8](#threatecstaskset8) |
| [Defense Evasion](../threat-impacts.md#defense-evasion) | [Threat.ECS.AccountSetting.1](#threatecsaccountsetting1), [Threat.ECS.AccountSetting.2](#threatecsaccountsetting2), [Threat.ECS.CapacityProvider.1](#threatecscapacityprovider1), [Threat.ECS.CapacityProvider.2](#threatecscapacityprovider2), [Threat.ECS.CapacityProvider.3](#threatecscapacityprovider3), [Threat.ECS.CapacityProvider.9](#threatecscapacityprovider9), [Threat.ECS.CapacityProvider.10](#threatecscapacityprovider10), [Threat.ECS.Cluster.1](#threatecscluster1), [Threat.ECS.Cluster.3](#threatecscluster3), [Threat.ECS.Cluster.4](#threatecscluster4), [Threat.ECS.Cluster.5](#threatecscluster5), [Threat.ECS.Cluster.7](#threatecscluster7), [Threat.ECS.Cluster.8](#threatecscluster8), [Threat.ECS.ContainerInstance.1](#threatecscontainerinstance1), [Threat.ECS.Daemon.1](#threatecsdaemon1), [Threat.ECS.DaemonTaskDefinition.1](#threatecsdaemontaskdefinition1), [Threat.ECS.DaemonTaskDefinition.4](#threatecsdaemontaskdefinition4), [Threat.ECS.DaemonTaskDefinition.5](#threatecsdaemontaskdefinition5), [Threat.ECS.ExpressGatewayService.1](#threatecsexpressgatewayservice1), [Threat.ECS.ExpressGatewayService.8](#threatecsexpressgatewayservice8), [Threat.ECS.ExpressGatewayService.9](#threatecsexpressgatewayservice9), [Threat.ECS.ExpressGatewayService.10](#threatecsexpressgatewayservice10), [Threat.ECS.Service.1](#threatecsservice1), [Threat.ECS.Service.3](#threatecsservice3), [Threat.ECS.Service.4](#threatecsservice4), [Threat.ECS.Service.5](#threatecsservice5), [Threat.ECS.Service.13](#threatecsservice13), [Threat.ECS.Service.16](#threatecsservice16), [Threat.ECS.Service.19](#threatecsservice19), [Threat.ECS.Task.2](#threatecstask2), [Threat.ECS.Task.3](#threatecstask3), [Threat.ECS.Task.12](#threatecstask12), [Threat.ECS.Task.13](#threatecstask13), [Threat.ECS.TaskDefinition.1](#threatecstaskdefinition1), [Threat.ECS.TaskDefinition.2](#threatecstaskdefinition2), [Threat.ECS.TaskDefinition.3](#threatecstaskdefinition3), [Threat.ECS.TaskDefinition.6](#threatecstaskdefinition6), [Threat.ECS.TaskDefinition.8](#threatecstaskdefinition8), [Threat.ECS.TaskSet.1](#threatecstaskset1), [Threat.ECS.TaskSet.8](#threatecstaskset8) |
| [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Threat.ECS.CapacityProvider.4](#threatecscapacityprovider4), [Threat.ECS.CapacityProvider.11](#threatecscapacityprovider11), [Threat.ECS.ExpressGatewayService.2](#threatecsexpressgatewayservice2), [Threat.ECS.ExpressGatewayService.11](#threatecsexpressgatewayservice11), [Threat.ECS.Service.8](#threatecsservice8), [Threat.ECS.Service.20](#threatecsservice20), [Threat.ECS.Task.6](#threatecstask6), [Threat.ECS.Task.14](#threatecstask14), [Threat.ECS.TaskSet.2](#threatecstaskset2), [Threat.ECS.TaskSet.9](#threatecstaskset9) |
| [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Threat.ECS.Service.2](#threatecsservice2) |
| [Initial Access](../threat-impacts.md#initial-access) | [Threat.ECS.CapacityProvider.5](#threatecscapacityprovider5), [Threat.ECS.CapacityProvider.7](#threatecscapacityprovider7), [Threat.ECS.DaemonTaskDefinition.2](#threatecsdaemontaskdefinition2), [Threat.ECS.ExpressGatewayService.4](#threatecsexpressgatewayservice4), [Threat.ECS.ExpressGatewayService.5](#threatecsexpressgatewayservice5), [Threat.ECS.ExpressGatewayService.6](#threatecsexpressgatewayservice6), [Threat.ECS.Service.10](#threatecsservice10), [Threat.ECS.Service.11](#threatecsservice11), [Threat.ECS.Service.12](#threatecsservice12), [Threat.ECS.Task.8](#threatecstask8), [Threat.ECS.Task.9](#threatecstask9), [Threat.ECS.Task.10](#threatecstask10), [Threat.ECS.TaskDefinition.4](#threatecstaskdefinition4), [Threat.ECS.TaskSet.4](#threatecstaskset4), [Threat.ECS.TaskSet.5](#threatecstaskset5), [Threat.ECS.TaskSet.6](#threatecstaskset6) |
| [Lateral Movement](../threat-impacts.md#lateral-movement) | [Threat.ECS.ExpressGatewayService.3](#threatecsexpressgatewayservice3), [Threat.ECS.Service.9](#threatecsservice9), [Threat.ECS.Task.7](#threatecstask7), [Threat.ECS.TaskSet.3](#threatecstaskset3) |
