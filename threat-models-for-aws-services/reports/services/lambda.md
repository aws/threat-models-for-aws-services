# AWS Lambda (lambda) Threat Model

[Threat Summary](#threat-summary) · [Threats](#threats) · [Mitigations](#mitigations) · [Implementations](#implementations) · [Appendix](#appendix)

## Overview

AWS Lambda is a serverless compute service that runs code without requiring you to provision or manage servers. Lambda manages the underlying infrastructure, including server maintenance, capacity provisioning, automatic scaling, and patching. You supply code as a .zip archive or a container image, and Lambda runs it inside an isolated, service-managed execution environment.

52 threats and 39 mitigations across 6 resource types and 88 API operations. 21 of 38 mitigation goals are covered, implemented by 116 implementations.

Source service model: [lambda](https://github.com/aws/api-models-aws/blob/a0767ac42e279dfc8b311564c33436f8abfd3031/models/lambda/service/2015-03-31/lambda-2015-03-31.json)

---

## Threat Summary

### [Cross-service confused deputy problem through resource-based policies](../potential-threats.md#cross-service-confused-deputy-problem-through-resource-based-policies)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.Function.17](#threatlambdafunction17) | Untrusted IAM principal, with access to configure an AWS service that has been granted access to an AWS Lambda function through its resource-based policy, may be able to cause that AWS service to interact with the function using the principal's data or context. | [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13) |

### [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.11](#threatlambdacapacityprovider11) | Threat actor, with control of an AWS Lambda capacity provider configured with overly permissive outbound security group rules, can use the capacity provider to interact with ungoverned network destinations. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8) |
| [Threat.LAMBDA.EventSourceMapping.12](#threatlambdaeventsourcemapping12) | Threat actor, with control of an AWS Lambda event source mapping configured with overly permissive outbound security group rules, can use the event source mapping to interact with ungoverned network destinations, directing its event poller's outbound connections and the record batches they carry to endpoints outside the intended network boundary. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7) |
| [Threat.LAMBDA.Function.20](#threatlambdafunction20) | Threat actor, with control of an AWS Lambda function configured with overly permissive outbound security group rules, can use the function to interact with ungoverned network destinations. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17) |

### [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.1](#threatlambdacapacityprovider1) | IAM principal, with IAM permissions to create or modify data in an AWS Lambda capacity provider, can encrypt the Amazon EBS volumes of the Managed Instances that Lambda provisions for the capacity provider with key material inaccessible to the AWS account owner. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5) |
| [Threat.LAMBDA.EventSourceMapping.3](#threatlambdaeventsourcemapping3) | IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping, can encrypt the filter criteria object with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3) |
| [Threat.LAMBDA.Function.3](#threatlambdafunction3) | IAM principal, with IAM permissions to create or modify data in an AWS Lambda function, can encrypt the function's environment variables, .zip deployment package, SnapStart snapshots, and durable execution payload data with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8) |

### [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.EventSourceMapping.6](#threatlambdaeventsourcemapping6) | IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping that supports detailed CloudWatch metrics through the `EventCount`, `ErrorCount`, and `KafkaMetrics` event source mapping metric groups, can create or configure the event source mapping without any event source mapping metric group enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6) |
| [Threat.LAMBDA.Function.7](#threatlambdafunction7) | IAM principal, with IAM permissions to create or modify an AWS Lambda function that supports CloudWatch Lambda Insights enhanced monitoring and CloudWatch Application Signals, can create or configure the function without CloudWatch Lambda Insights enhanced monitoring or CloudWatch Application Signals enabled. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16) |

### [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.EventSourceMapping.9](#threatlambdaeventsourcemapping9) | IAM principal, with IAM permissions to use an AWS Lambda event source mapping, can interact with the event source mapping without a record of their activities being persisted by event poller system logging to CloudWatch Logs. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1) |

### [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.1](#threatlambda1) | Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make AWS Lambda service API calls. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.2](#mitigationlambda2) |

### [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.Function.12](#threatlambdafunction12) | Untrusted IAM principal, that has been granted access to my AWS Lambda function through its resource-based policy, can use my function, invoking it with a chosen `$.Payload` and `$.Qualifier` to run my code under its execution role, reading its environment variables and deployment package with `GetFunction`, replacing what it runs with `UpdateFunctionCode`, or deleting it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12) |
| [Threat.LAMBDA.Layer.3](#threatlambdalayer3) | Untrusted IAM principal, that has been granted access to my AWS Lambda layer through its resource-based policy, can use my layer to download the layer archive and to attach the layer version to functions in accounts I do not control. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |

### [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.9](#threatlambdacapacityprovider9) | Untrusted IAM principal, that has been granted access to my AWS Lambda capacity provider's operator IAM role through its trust policy, can assume my operator role and act with its permissions. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access), [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2) |
| [Threat.LAMBDA.Function.13](#threatlambdafunction13) | Untrusted IAM principal, that has been granted access to my AWS Lambda function's IAM execution role through its trust policy, can assume my execution role and act with its permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3) |

### [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.5](#threatlambdacapacityprovider5) | Threat actor, who controls a VPC-connected resource in the same VPC as an AWS Lambda capacity provider, can interact with the capacity provider's Managed Instance network interfaces over HTTPS or HTTP through overly permissive network rules and move laterally within the VPC. | [Lateral Movement](../threat-impacts.md#lateral-movement) | [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8) |

### [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.3](#threatlambdacapacityprovider3) | IAM principal, with IAM permissions to create or modify an AWS Lambda capacity provider, can encrypt the capacity provider with an AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5) |
| [Threat.LAMBDA.EventSourceMapping.5](#threatlambdaeventsourcemapping5) | IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping's filter criteria, can encrypt the filter criteria with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3) |
| [Threat.LAMBDA.Function.5](#threatlambdafunction5) | IAM principal, with IAM permissions to create or modify an AWS Lambda function, can encrypt the function with an AWS owned or AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8) |

### [Logging disablement](../potential-threats.md#logging-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.10](#threatlambdacapacityprovider10) | IAM principal, with IAM permissions to modify an AWS Lambda capacity provider, can disable system logging to Amazon CloudWatch Logs for the capacity provider. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1) |
| [Threat.LAMBDA.EventSourceMapping.11](#threatlambdaeventsourcemapping11) | IAM principal, with IAM permissions to modify an AWS Lambda event source mapping, can disable system logging to CloudWatch Logs for the event source mapping. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1) |
| [Threat.LAMBDA.Function.18](#threatlambdafunction18) | IAM principal, with IAM permissions to modify an AWS Lambda function, can disable CloudWatch Logs delivery of system and application logs for the function. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1) |

### [Monitoring disablement](../potential-threats.md#monitoring-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.EventSourceMapping.2](#threatlambdaeventsourcemapping2) | IAM principal, with IAM permissions to modify an AWS Lambda event source mapping, can disable the `EventCount`, `ErrorCount`, and `KafkaMetrics` metric groups for the event source mapping. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6) |
| [Threat.LAMBDA.Function.2](#threatlambdafunction2) | IAM principal, with IAM permissions to modify an AWS Lambda function or its execution role, can disable CloudWatch Lambda Insights enhanced monitoring, CloudWatch Application Signals, and AWS X-Ray active tracing for the function. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16) |

### [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.6](#threatlambdacapacityprovider6) | Threat actor, with network access to an AWS Lambda capacity provider that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the compute instances managed by the capacity provider. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8) |

### [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.Function.14](#threatlambdafunction14) | Threat actor, with possession of credentials for my AWS Lambda function execution role, can use my function execution role while on an unexpected network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2) |

### [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.Function.10](#threatlambdafunction10) | IAM principal, with IAM permissions to use my AWS Lambda function and access to an unexpected network, can interact with my function from an unexpected network. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5) |

### [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.7](#threatlambdacapacityprovider7) | Threat actor, with network access to an AWS Lambda capacity provider that has been placed in a public subnet, can access the capacity provider directly from an ungoverned network. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3) |

### [Resource Configuration - Data movement through an untrusted configured target](../potential-threats.md#resource-configuration---data-movement-through-an-untrusted-configured-target)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CodeSigningConfig.1](#threatlambdacodesigningconfig1) | My IAM principal, with IAM permissions to create or modify my AWS Lambda code signing config, can configure my code signing config to use an untrusted AWS Signer signing profile version to validate the signature of code packages and layers deployed to the functions that reference the configuration. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Malicious Code Execution](../threat-impacts.md#malicious-code-execution) | [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1) |

### [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.2](#threatlambdacapacityprovider2) | IAM principal, with IAM permissions to create or modify an AWS Lambda capacity provider, can configure the capacity provider without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7) |
| [Threat.LAMBDA.CodeSigningConfig.2](#threatlambdacodesigningconfig2) | IAM principal, with IAM permissions to create or modify an AWS Lambda code signing config, can configure the code signing config without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2) |
| [Threat.LAMBDA.EventSourceMapping.4](#threatlambdaeventsourcemapping4) | IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping, can configure the event source mapping without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5) |
| [Threat.LAMBDA.Function.4](#threatlambdafunction4) | IAM principal, with IAM permissions to create or modify an AWS Lambda function, can configure the function without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15) |

### [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.2](#threatlambda2) | IAM principal, with IAM permissions on untrusted AWS Lambda resources, can interact with those untrusted resources from a governed VPC network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.1](#mitigationlambda1) |

### [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.Function.8](#threatlambdafunction8) | My IAM principal, with IAM permissions to invoke, modify, or delete an untrusted AWS Lambda function, can interact with the untrusted function, including sending data to it in the invocation payload or receiving data from it in the invocation response. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6) |
| [Threat.LAMBDA.Layer.2](#threatlambdalayer2) | My IAM principal, with IAM permissions to read, use, or modify an untrusted AWS Lambda layer, can interact with the untrusted layer, including sending data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1) |

### [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.4](#threatlambdacapacityprovider4) | IAM principal, with IAM permissions to create an AWS Lambda capacity provider, can create the capacity provider in a single Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6) |
| [Threat.LAMBDA.EventSourceMapping.7](#threatlambdaeventsourcemapping7) | IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping, can create or modify the event source mapping with subnets that all resolve to a single Availability Zone, placing the event poller's network interfaces in that one Availability Zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4) |
| [Threat.LAMBDA.Function.9](#threatlambdafunction9) | IAM principal, with IAM permissions to create or modify an AWS Lambda function, can create the function in a single Availability Zone by attaching it to VPC subnets that all resolve to one zone. | [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9) |

### [Unauthorized data access through AWS managed KMS key](../potential-threats.md#unauthorized-data-access-through-aws-managed-kms-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.Function.15](#threatlambdafunction15) | IAM principal, with unintended IAM permissions to read environment variables from an AWS Lambda function that is encrypted with an AWS managed KMS key in the same account as the principal, can read the environment variables in the function. | [Credential Theft](../threat-impacts.md#credential-theft), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8) |

### [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.EventSourceMapping.10](#threatlambdaeventsourcemapping10) | IAM principal, with unintended IAM permissions to read the filter patterns from an AWS Lambda event source mapping's filter criteria that has been encrypted with an AWS owned KMS key, can read the filter patterns in the filter criteria. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3) |
| [Threat.LAMBDA.Function.16](#threatlambdafunction16) | IAM principal, with unintended IAM permissions to read the .zip deployment package and function configuration details from an AWS Lambda function that has been encrypted with an AWS owned KMS key, can read the .zip deployment package and function configuration details in the function. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8) |

### [Unauthorized data access through service principal presigned URL](../potential-threats.md#unauthorized-data-access-through-service-principal-presigned-url)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.Function.19](#threatlambdafunction19) | IAM principal, with permissions to read an AWS Lambda function, which returns an Amazon S3 presigned URL signed by the AWS Lambda service principal for a service-owned bucket, can use the presigned URL to read the function's deployment package's data in the service-owned bucket while on an unexpected network. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14) |

### [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.EventSourceMapping.8](#threatlambdaeventsourcemapping8) | IAM principal, with unintended IAM permissions to read the filter patterns from an AWS Lambda event source mapping's filter criteria that has not been encrypted at rest with a KMS customer managed key, can read plaintext filter patterns and the business values they embed in the filter criteria. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3) |
| [Threat.LAMBDA.Function.11](#threatlambdafunction11) | IAM principal, with unintended IAM permissions to read environment variables, deployment package code and durable execution payloads from an AWS Lambda function that has not been encrypted at rest with a KMS customer managed key, can read plaintext environment variables, deployment package code and durable execution payloads in the function. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8) |

### [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.12](#threatlambdacapacityprovider12) | Threat actor, who can execute code in an AWS Lambda capacity provider that is attached to a subnet with a route to an internet gateway, can send the capacity provider's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3) |
| [Threat.LAMBDA.Function.21](#threatlambdafunction21) | Threat actor, who can execute code in an AWS Lambda function that is attached to a subnet with a route to an internet gateway, can send the function's outbound traffic directly to the public internet instead of through a governed network egress path. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4) |

### [Unencrypted instance-to-instance traffic through instance type selection](../potential-threats.md#unencrypted-instance-to-instance-traffic-through-instance-type-selection)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.CapacityProvider.8](#threatlambdacapacityprovider8) | IAM principal, with IAM permissions to create or update an AWS Lambda capacity provider, can configure the capacity provider to use an EC2 instance type that does not provide automatic encryption of instance-to-instance traffic. | [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4) |

### [Ungoverned network traffic from non-VPC resources](../potential-threats.md#ungoverned-network-traffic-from-non-vpc-resources)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.EventSourceMapping.1](#threatlambdaeventsourcemapping1) | IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping that is not attached to a VPC, can use the event source mapping to send and receive traffic outside the network controls a VPC would enforce by pointing the poller at publicly reachable brokers, schema registries, or on-failure destinations. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2) |
| [Threat.LAMBDA.Function.1](#threatlambdafunction1) | IAM principal, with IAM permissions to create, modify or invoke an AWS Lambda function that is not attached to a VPC, can use the function to send and receive traffic outside the network controls a VPC would enforce. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7) |

### [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.LAMBDA.Function.6](#threatlambdafunction6) | Threat actor, with a method for enumerating public AWS Lambda functions, or prior knowledge of an existing public function, can access a function that has been configured to allow public access, either through a function URL that permits unauthenticated invocation or through a resource-based policy that grants a wildcard principal, and invoke the function or read its configuration. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11) |
| [Threat.LAMBDA.Layer.1](#threatlambdalayer1) | Threat actor, with a method for enumerating public AWS Lambda layers, or prior knowledge of an existing public layer, can access a layer that has been configured to allow public access, reading the layer version and retrieving a download link to its .zip archive, or attaching the layer version to a function in their own account. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2) |

---

## Threats

Each scoping statement below describes the exposure the threat's mapped mitigations address. It assumes those mitigations are not applied and makes no claim about your resources.

### Threat.LAMBDA.1

Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make AWS Lambda service API calls.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.2](#mitigationlambda2)

**Frameworks:** MITRE ATT&CK [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)): An IAM principal from outside your organization can use the interface VPC endpoint (powered by AWS PrivateLink) in your VPC with its own credentials. That access enables transfer of data from your VPC to Lambda resources in an AWS account outside your organization.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:AddPermission` | `$.FunctionName`, `$.Principal` |
| `lambda:DeleteFunction` | `$.FunctionName` |
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload` |
| `lambda:PublishLayerVersion` | `$.Content.ZipFile`, `$.LayerName` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.S3Bucket`, `$.ZipFile` |
| `lambda:UpdateFunctionConfiguration` | `$.Environment.Variables{key}`, `$.FunctionName` |

### Threat.LAMBDA.2

IAM principal, with IAM permissions on untrusted AWS Lambda resources, can interact with those untrusted resources from a governed VPC network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.1](#mitigationlambda1)

**Frameworks:** MITRE ATT&CK [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)): A principal on your VPC network can reach AWS Lambda functions and layer versions in accounts outside your organization. That access enables invoking a cross-account function with your data as input or downloading its deployment package and layer archives.

**Data perimeter:** `GetLayerVersion` might require access to service-owned layers, which are layers that are managed in service accounts.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:AddPermission` | `$.FunctionName`, `$.Principal` |
| `lambda:DeleteFunction` | `$.FunctionName` |
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetLayerVersionByArn` | `$.Arn` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.ZipFile` |
| `lambda:UpdateFunctionConfiguration` | `$.Environment.Variables{key}`, `$.FunctionName` |

### Threat.LAMBDA.CapacityProvider.1

IAM principal, with IAM permissions to create or modify data in an AWS Lambda capacity provider, can encrypt the Amazon EBS volumes of the Managed Instances that Lambda provisions for the capacity provider with key material inaccessible to the AWS account owner.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/), [T1490](https://attack.mitre.org/techniques/T1490/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal that creates your capacity provider selects the encryption key for the Amazon EBS volumes of its Managed Instances. If that key material sits outside your control, its holder can later revoke or delete it and leave those volumes unrecoverable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.KmsKeyArn` |

### Threat.LAMBDA.CapacityProvider.2

IAM principal, with IAM permissions to create or modify an AWS Lambda capacity provider, can configure the capacity provider without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7)

**Frameworks:** MITRE ATT&CK [T1578.005](https://attack.mitre.org/techniques/T1578/005/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An AWS Lambda capacity provider is an account-level resource that supports tagging at creation and afterwards. An IAM principal with permissions to create or modify one can omit required tags and may leave the resource without ownership attribution.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.PropagateTags.ExplicitTags{key}`, `$.PropagateTags.Mode`, `$.Tags{key}` |
| `lambda:TagResource` | `$.Resource`, `$.Tags{key}` |
| `lambda:UntagResource` | `$.Resource`, `$.TagKeys[*]` |
| `lambda:UpdateCapacityProvider` | `$.CapacityProviderName`, `$.PropagateTags.ExplicitTags{key}`, `$.PropagateTags.Mode` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.LAMBDA.CapacityProvider.3

IAM principal, with IAM permissions to create or modify an AWS Lambda capacity provider, can encrypt the capacity provider with an AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal with permissions to create or modify a capacity provider can encrypt its data with an AWS managed KMS key. You cannot change an AWS managed key's policy or govern its lifecycle.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.KmsKeyArn` |

### Threat.LAMBDA.CapacityProvider.4

IAM principal, with IAM permissions to create an AWS Lambda capacity provider, can create the capacity provider in a single Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6)

**Frameworks:** MITRE ATT&CK [T1499](https://attack.mitre.org/techniques/T1499/)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): `CreateCapacityProvider` accepts as few as one subnet in the VPC configuration subnet list, `$.VpcConfig.SubnetIds[*]`. A capacity provider confined to one Availability Zone may become unavailable to the function versions bound to it during a zonal impairment.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.VpcConfig.SubnetIds[*]` |

### Threat.LAMBDA.CapacityProvider.5

Threat actor, who controls a VPC-connected resource in the same VPC as an AWS Lambda capacity provider, can interact with the capacity provider's Managed Instance network interfaces over HTTPS or HTTP through overly permissive network rules and move laterally within the VPC.

**Impacts:** [Lateral Movement](../threat-impacts.md#lateral-movement)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)

**Frameworks:** MITRE ATT&CK [T1021](https://attack.mitre.org/techniques/T1021/)

**Protocols:** `HTTPS`, `HTTP`

**Scoping** (for [Lateral movement through permissive network rules](../potential-threats.md#lateral-movement-through-permissive-network-rules)): Your capacity provider launches Amazon EC2 Managed Instances with network interfaces in the VPC subnets you specify. A threat actor controlling another resource in that VPC can reach those interfaces through overly permissive network rules and move laterally.

### Threat.LAMBDA.CapacityProvider.6

Threat actor, with network access to an AWS Lambda capacity provider that permits traffic to services and administrative ports from unrestricted sources, can reach listening services and administrative ports on the compute instances managed by the capacity provider.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/), [T1133](https://attack.mitre.org/techniques/T1133/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTP`, `HTTPS`

**Scoping** (for [Network exposure through overly-permissive inbound security group rules](../potential-threats.md#network-exposure-through-overly-permissive-inbound-security-group-rules)): Security groups associated with a capacity provider admit inbound traffic to the Amazon EC2 instances that run your function code. An inbound rule permitting an unrestricted source range makes listening services and administrative ports on those instances reachable over the network.

### Threat.LAMBDA.CapacityProvider.7

Threat actor, with network access to an AWS Lambda capacity provider that has been placed in a public subnet, can access the capacity provider directly from an ungoverned network.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3)

**Frameworks:** MITRE ATT&CK [T1046](https://attack.mitre.org/techniques/T1046/)

**Protocols:** `TCP`, `UDP`, `ICMP`

**Scoping** (for [Public subnet resource exposure](../potential-threats.md#public-subnet-resource-exposure)): A capacity provider launches Amazon EC2 managed instances and their network interfaces into the subnets you choose at creation time. If you choose a public subnet, a threat actor may be able to access the capacity provider directly from an ungoverned network.

### Threat.LAMBDA.CapacityProvider.8

IAM principal, with IAM permissions to create or update an AWS Lambda capacity provider, can configure the capacity provider to use an EC2 instance type that does not provide automatic encryption of instance-to-instance traffic.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Scoping** (for [Unencrypted instance-to-instance traffic through instance type selection](../potential-threats.md#unencrypted-instance-to-instance-traffic-through-instance-type-selection)): A capacity provider's allowed instance types setting (`$.InstanceRequirements.AllowedInstanceTypes[*]`) accepts Amazon EC2 instance types that do not provide automatic encryption of instance-to-instance traffic. If such a type is selected, traffic between the capacity provider's instances and other instances you operate can travel without that encryption.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.InstanceRequirements.AllowedInstanceTypes[*]`, `$.InstanceRequirements.ExcludedInstanceTypes[*]` |

### Threat.LAMBDA.CapacityProvider.9

Untrusted IAM principal, that has been granted access to my AWS Lambda capacity provider's operator IAM role through its trust policy, can assume my operator role and act with its permissions.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Initial Access](../threat-impacts.md#initial-access), [Privilege Escalation](../threat-impacts.md#privilege-escalation)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1580](https://attack.mitre.org/techniques/T1580/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Every AWS Lambda capacity provider designates an operator role whose trust policy you author yourself. That trust policy can allow an untrusted principal to assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.PermissionsConfig.CapacityProviderOperatorRoleArn` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.LAMBDA.CapacityProvider.10

IAM principal, with IAM permissions to modify an AWS Lambda capacity provider, can disable system logging to Amazon CloudWatch Logs for the capacity provider.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): The create and update operations for a capacity provider both accept the CloudWatch log group destination for system logs, `$.TelemetryConfig.LoggingConfig.LogGroup`. A principal able to modify a capacity provider can use that setting to end system log delivery to Amazon CloudWatch Logs.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.TelemetryConfig.LoggingConfig.LogGroup` |
| `lambda:UpdateCapacityProvider` | `$.CapacityProviderName`, `$.TelemetryConfig.LoggingConfig.LogGroup` |

### Threat.LAMBDA.CapacityProvider.11

Threat actor, with control of an AWS Lambda capacity provider configured with overly permissive outbound security group rules, can use the capacity provider to interact with ungoverned network destinations.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1567](https://attack.mitre.org/techniques/T1567/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): Outbound rules on the security groups attached to a capacity provider control which destinations its network traffic can reach. Where those rules permit unrestricted outbound traffic, the capacity provider can interact with ungoverned network destinations.

### Threat.LAMBDA.CapacityProvider.12

Threat actor, who can execute code in an AWS Lambda capacity provider that is attached to a subnet with a route to an internet gateway, can send the capacity provider's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): All egress from functions on a capacity provider leaves through managed Amazon EC2 instance network interfaces in the subnets you specify. Where those subnets route to an internet gateway, code running on the pool can reach the public internet directly.

### Threat.LAMBDA.CodeSigningConfig.1

My IAM principal, with IAM permissions to create or modify my AWS Lambda code signing config, can configure my code signing config to use an untrusted AWS Signer signing profile version to validate the signature of code packages and layers deployed to the functions that reference the configuration.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Malicious Code Execution](../threat-impacts.md#malicious-code-execution)

**Mitigations:** [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/), [T1648](https://attack.mitre.org/techniques/T1648/)

**Scoping** (for [Resource Configuration - Data movement through an untrusted configured target](../potential-threats.md#resource-configuration---data-movement-through-an-untrusted-configured-target)): An IAM principal that can create or modify your code signing config sets its allowed publishers list (`$.AllowedPublishers.SigningProfileVersionArns[*]`). At deployment time, AWS Lambda validates code packages and layers against those AWS Signer signing profiles, including any owned by accounts outside your organization.

**Data perimeter:** `CreateCodeSigningConfig` allows you to specify a signing profile that does not belong to your organization as the value for the `AllowedPublishers.SigningProfileVersionArns` request parameter. Because the subsequent call against the signing profile is performed by the service principal, it is not restricted with `aws:ResourceOrgID` implemented in a service control policy.

**Data perimeter:** `UpdateCodeSigningConfig` allows you to specify an AWS Signer signing profile that does not belong to your organization as the value for the `AllowedPublishers.SigningProfileVersionArns` request parameter. Because the subsequent call against the AWS Signer signing profile is performed by the service principal, it is not restricted with `aws:ResourceOrgID` implemented in a service control policy.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCodeSigningConfig` | `$.AllowedPublishers.SigningProfileVersionArns[*]` |
| `lambda:UpdateCodeSigningConfig` | `$.AllowedPublishers.SigningProfileVersionArns[*]`, `$.CodeSigningConfigArn` |

### Threat.LAMBDA.CodeSigningConfig.2

IAM principal, with IAM permissions to create or modify an AWS Lambda code signing config, can configure the code signing config without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal can create a Lambda code signing configuration without tags, and call `UntagResource` to remove tag keys from one already tagged. The configuration then could carry no required owner or purpose tag, yet it still decides which signing profiles are trusted to sign your function code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateCodeSigningConfig` | `$.Tags{key}` |
| `lambda:UntagResource` | `$.Resource`, `$.TagKeys[*]` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.LAMBDA.EventSourceMapping.1

IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping that is not attached to a VPC, can use the event source mapping to send and receive traffic outside the network controls a VPC would enforce by pointing the poller at publicly reachable brokers, schema registries, or on-failure destinations.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1537](https://attack.mitre.org/techniques/T1537/), [T1562.007](https://attack.mitre.org/techniques/T1562/007/), [T1567](https://attack.mitre.org/techniques/T1567/)

**Scoping** (for [Ungoverned network traffic from non-VPC resources](../potential-threats.md#ungoverned-network-traffic-from-non-vpc-resources)): An IAM principal with permissions to create or modify an AWS Lambda event source mapping can omit its VPC subnet and security group configuration. Its event pollers can send and receive traffic outside the network controls a VPC would enforce, reaching public brokers, schema registries, and on-failure destinations.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.AmazonManagedKafkaEventSourceConfig.SchemaRegistryConfig.SchemaRegistryURI`, `$.DestinationConfig.OnFailure.Destination`, `$.Enabled`, `$.EventSourceArn`, `$.FunctionName`, `$.SelfManagedEventSource.Endpoints{key}[*]`, `$.SelfManagedKafkaEventSourceConfig.SchemaRegistryConfig.SchemaRegistryURI`, `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI` |
| `lambda:UpdateEventSourceMapping` | `$.AmazonManagedKafkaEventSourceConfig.SchemaRegistryConfig.SchemaRegistryURI`, `$.DestinationConfig.OnFailure.Destination`, `$.Enabled`, `$.SelfManagedKafkaEventSourceConfig.SchemaRegistryConfig.SchemaRegistryURI`, `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.2

IAM principal, with IAM permissions to modify an AWS Lambda event source mapping, can disable the `EventCount`, `ErrorCount`, and `KafkaMetrics` metric groups for the event source mapping.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): The `EventCount`, `ErrorCount`, and `KafkaMetrics` metric groups determine the detailed CloudWatch metrics an event source mapping publishes. A principal calling `UpdateEventSourceMapping` can remove those groups from the mapping.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:UpdateEventSourceMapping` | `$.MetricsConfig.Metrics[*]`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.3

IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping, can encrypt the filter criteria object with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal that can create or modify your event source mapping sets `$.KMSKeyArn`, the customer managed key encrypting its filter criteria. That principal can name a key you do not administer, taking control of the key policy, its lifecycle, and decryption of the stored criteria.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.KMSKeyArn` |
| `lambda:UpdateEventSourceMapping` | `$.KMSKeyArn`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.4

IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping, can configure the event source mapping without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An event source mapping is a taggable AWS Lambda resource that an IAM principal with create or modify permissions can configure without required tags. An untagged mapping may not be covered by your ownership attribution and cost allocation processes.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.Tags{key}` |
| `lambda:UntagResource` | `$.Resource`, `$.TagKeys[*]` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.LAMBDA.EventSourceMapping.5

IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping's filter criteria, can encrypt the filter criteria with an AWS owned KMS key, limiting your ability to govern the key's policy and lifecycle and to audit key usage through CloudTrail.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3)

**Frameworks:** MITRE ATT&CK [T1119](https://attack.mitre.org/techniques/T1119/)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): A principal that creates or modifies an event source mapping's filter criteria can leave them encrypted under an AWS owned key outside your account. You cannot then govern that key's permissions, rotation, or deletion, or audit its use.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.KMSKeyArn` |
| `lambda:UpdateEventSourceMapping` | `$.KMSKeyArn`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.6

IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping that supports detailed CloudWatch metrics through the `EventCount`, `ErrorCount`, and `KafkaMetrics` event source mapping metric groups, can create or configure the event source mapping without any event source mapping metric group enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): An IAM principal with permissions to create or modify your event source mapping can leave the `EventCount`, `ErrorCount`, and `KafkaMetrics` metric groups unselected. The mapping then runs without the detailed stage-level poller measurements those groups publish.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.DestinationConfig.OnFailure.Destination`, `$.EventSourceArn`, `$.FilterCriteria.Filters[*].Pattern`, `$.FunctionName`, `$.MetricsConfig.Metrics[*]` |
| `lambda:UpdateEventSourceMapping` | `$.FilterCriteria.Filters[*].Pattern`, `$.MetricsConfig.Metrics[*]`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.7

IAM principal, with IAM permissions to create or modify an AWS Lambda event source mapping, can create or modify the event source mapping with subnets that all resolve to a single Availability Zone, placing the event poller's network interfaces in that one Availability Zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4)

**Frameworks:** MITRE ATT&CK [T1499](https://attack.mitre.org/techniques/T1499/)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create or modify an event source mapping can select subnets that all resolve to a single Availability Zone. Impairment of that zone can then interrupt polling and downstream function invocation.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI` |
| `lambda:UpdateEventSourceMapping` | `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.8

IAM principal, with unintended IAM permissions to read the filter patterns from an AWS Lambda event source mapping's filter criteria that has not been encrypted at rest with a KMS customer managed key, can read plaintext filter patterns and the business values they embed in the filter criteria.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1119](https://attack.mitre.org/techniques/T1119/)

**Scoping** (for [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)): An event source mapping stores filter criteria patterns that can embed literal business values. A principal holding only `lambda:GetEventSourceMapping` or `lambda:ListEventSourceMappings` permission can read those patterns in plaintext.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.KMSKeyArn` |
| `lambda:GetEventSourceMapping` | `$.UUID` |
| `lambda:ListEventSourceMappings` | `$.EventSourceArn`, `$.FunctionName` |
| `lambda:UpdateEventSourceMapping` | `$.KMSKeyArn`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.9

IAM principal, with IAM permissions to use an AWS Lambda event source mapping, can interact with the event source mapping without a record of their activities being persisted by event poller system logging to CloudWatch Logs.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): System logging for event pollers is optional, so your event source mapping can operate without sending poller logs to CloudWatch Logs. A threat actor can then create, change, disable, or delete the mapping without those poller activities being persisted.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.EventSourceArn`, `$.FunctionName`, `$.SelfManagedEventSource.Endpoints{key}[*]`, `$.SourceAccessConfigurations[*].Type`, `$.StartingPosition`, `$.Topics[*]` |
| `lambda:DeleteEventSourceMapping` | `$.UUID` |
| `lambda:UpdateEventSourceMapping` | `$.Enabled`, `$.FilterCriteria.Filters[*].Pattern`, `$.FunctionName`, `$.SourceAccessConfigurations[*].URI`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.10

IAM principal, with unintended IAM permissions to read the filter patterns from an AWS Lambda event source mapping's filter criteria that has been encrypted with an AWS owned KMS key, can read the filter patterns in the filter criteria.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)): By default, your event source mapping's filter criteria is encrypted at rest with an AWS owned key. An IAM principal with unintended read permissions can retrieve the stored filter patterns in plaintext.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:GetEventSourceMapping` | `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.11

IAM principal, with IAM permissions to modify an AWS Lambda event source mapping, can disable system logging to CloudWatch Logs for the event source mapping.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/), [T1578.005](https://attack.mitre.org/techniques/T1578/005/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): Permissions to modify your event source mapping extend to the system log delivery it configures for Amazon CloudWatch Logs. A principal that can disable that delivery can leave subsequent event poller activity unrecorded at the resource level.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:UpdateEventSourceMapping` | `$.Enabled`, `$.LoggingConfig.SystemLogLevel`, `$.ProvisionedPollerConfig.MaximumPollers`, `$.ProvisionedPollerConfig.MinimumPollers`, `$.UUID` |

### Threat.LAMBDA.EventSourceMapping.12

Threat actor, with control of an AWS Lambda event source mapping configured with overly permissive outbound security group rules, can use the event source mapping to interact with ungoverned network destinations, directing its event poller's outbound connections and the record batches they carry to endpoints outside the intended network boundary.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): Your event source mapping's pollers carry polled record batches over outbound connections from network interfaces in your subnets. Overly permissive outbound rules on the security groups you author let those connections reach destinations beyond your intended network boundary.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.AmazonManagedKafkaEventSourceConfig.SchemaRegistryConfig.SchemaRegistryURI`, `$.EventSourceArn`, `$.SelfManagedEventSource.Endpoints{key}[*]`, `$.SelfManagedKafkaEventSourceConfig.SchemaRegistryConfig.SchemaRegistryURI` |
| `lambda:UpdateEventSourceMapping` | `$.AmazonManagedKafkaEventSourceConfig.SchemaRegistryConfig.SchemaRegistryURI`, `$.SelfManagedKafkaEventSourceConfig.SchemaRegistryConfig.SchemaRegistryURI`, `$.UUID` |

### Threat.LAMBDA.Function.1

IAM principal, with IAM permissions to create, modify or invoke an AWS Lambda function that is not attached to a VPC, can use the function to send and receive traffic outside the network controls a VPC would enforce.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1562.007](https://attack.mitre.org/techniques/T1562/007/), [T1567](https://attack.mitre.org/techniques/T1567/), [T1578.002](https://attack.mitre.org/techniques/T1578/002/)

**Scoping** (for [Ungoverned network traffic from non-VPC resources](../potential-threats.md#ungoverned-network-traffic-from-non-vpc-resources)): Your function does not run in your VPCs when `$.VpcConfig.SubnetIds[*]` and `$.VpcConfig.SecurityGroupIds[*]` are empty. An IAM principal with create, modify, or invoke permissions can then use it to send and receive traffic on unexpected networks, including the public internet.

**Data perimeter:** `CreateFunction` allows you to associate functions with an Amazon VPC to run your code.

**Data perimeter:** `UpdateFunctionConfiguration` allows you to associate functions with an Amazon VPC to run your code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Code.ImageUri`, `$.Code.ZipFile`, `$.FunctionName`, `$.VpcConfig.SecurityGroupIds[*]`, `$.VpcConfig.SubnetIds[*]` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload`, `$.X-Amz-Invocation-Type` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.ImageUri`, `$.ZipFile` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.VpcConfig.Ipv6AllowedForDualStack`, `$.VpcConfig.SecurityGroupIds[*]`, `$.VpcConfig.SubnetIds[*]` |

### Threat.LAMBDA.Function.2

IAM principal, with IAM permissions to modify an AWS Lambda function or its execution role, can disable CloudWatch Lambda Insights enhanced monitoring, CloudWatch Application Signals, and AWS X-Ray active tracing for the function.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Monitoring disablement](../potential-threats.md#monitoring-disablement)): CloudWatch Lambda Insights enhanced monitoring, CloudWatch Application Signals, and AWS X-Ray active tracing are opt-in features you enable on a Lambda function. An IAM principal with permissions to modify the function or its execution role can turn this monitoring off.

**Request parameters**

| Operation | Parameters |
|---|---|
| `iam:DetachRolePolicy` | `$.PolicyArn`, `$.RoleName` |
| `lambda:UpdateFunctionConfiguration` | `$.Environment.Variables{key}`, `$.FunctionName`, `$.Layers[*]`, `$.Role`, `$.TracingConfig.Mode` |

### Threat.LAMBDA.Function.3

IAM principal, with IAM permissions to create or modify data in an AWS Lambda function, can encrypt the function's environment variables, .zip deployment package, SnapStart snapshots, and durable execution payload data with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/), [T1490](https://attack.mitre.org/techniques/T1490/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): `$.KMSKeyArn`, `$.Code.SourceKMSKeyArn`, and `$.DurableConfig.KMSKeyArn` accept any symmetric customer managed key ARN, including keys backed by external or imported key material. A principal that creates or updates the function can therefore bind its encrypted state to key material the account owner cannot administer.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Code.SourceKMSKeyArn`, `$.DurableConfig.KMSKeyArn`, `$.FunctionName`, `$.KMSKeyArn` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.SourceKMSKeyArn` |
| `lambda:UpdateFunctionConfiguration` | `$.DurableConfig.KMSKeyArn`, `$.FunctionName`, `$.KMSKeyArn` |

### Threat.LAMBDA.Function.4

IAM principal, with IAM permissions to create or modify an AWS Lambda function, can configure the function without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15)

**Frameworks:** MITRE ATT&CK [T1562](https://attack.mitre.org/techniques/T1562/), [T1578](https://attack.mitre.org/techniques/T1578/)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): Lambda applies tags at the function level, so the function itself is the taggable unit. A principal that creates or modifies a function without your required tags leaves it outside ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.Tags{key}` |
| `lambda:UntagResource` | `$.Resource`, `$.TagKeys[*]` |
| `resource-groups-tagging-api:UntagResources` | `$.ResourceARNList[*]`, `$.TagKeys[*]` |

### Threat.LAMBDA.Function.5

IAM principal, with IAM permissions to create or modify an AWS Lambda function, can encrypt the function with an AWS owned or AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)

**Frameworks:** MITRE ATT&CK [T1530](https://attack.mitre.org/techniques/T1530/), [T1552.001](https://attack.mitre.org/techniques/T1552/001/)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): A principal able to create or modify a function can have the function's state encrypted under an AWS owned or AWS managed key. Both key classes reduce your control over the key's lifecycle and policy, and an AWS owned key leaves you no key-usage audit trail.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Code.SourceKMSKeyArn`, `$.DurableConfig.KMSKeyArn`, `$.FunctionName`, `$.KMSKeyArn` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.SourceKMSKeyArn` |
| `lambda:UpdateFunctionConfiguration` | `$.DurableConfig.KMSKeyArn`, `$.FunctionName`, `$.KMSKeyArn` |

### Threat.LAMBDA.Function.6

Threat actor, with a method for enumerating public AWS Lambda functions, or prior knowledge of an existing public function, can access a function that has been configured to allow public access, either through a function URL that permits unauthenticated invocation or through a resource-based policy that grants a wildcard principal, and invoke the function or read its configuration.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1190](https://attack.mitre.org/techniques/T1190/)

**Protocols:** `HTTPS`

**Scoping** (for [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)): A function URL that allows unauthenticated access, or a resource-based policy with a wildcard principal, exposes your function to callers outside your account. A threat actor who discovers the function can then invoke it or read its configuration.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload`, `$.Qualifier`, `$.X-Amz-Invocation-Type`, `$.X-Amz-Log-Type` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload`, `$.Qualifier` |

### Threat.LAMBDA.Function.7

IAM principal, with IAM permissions to create or modify an AWS Lambda function that supports CloudWatch Lambda Insights enhanced monitoring and CloudWatch Application Signals, can create or configure the function without CloudWatch Lambda Insights enhanced monitoring or CloudWatch Application Signals enabled.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16)

**Frameworks:** MITRE ATT&CK [T1562.001](https://attack.mitre.org/techniques/T1562/001/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Defense evasion through unconfigured enhanced monitoring](../potential-threats.md#defense-evasion-through-unconfigured-enhanced-monitoring)): A function can be created and invoked with only the default invocation metrics, such as counts, errors, and duration. Without CloudWatch Lambda Insights system-level measurements or CloudWatch Application Signals latency and fault metrics, anomalous resource use may be harder to detect.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Environment.Variables{key}`, `$.FunctionName`, `$.Layers[*]`, `$.TracingConfig.Mode` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload` |
| `lambda:UpdateFunctionCode` | `$.ImageUri`, `$.ZipFile` |
| `lambda:UpdateFunctionConfiguration` | `$.Environment.Variables{key}`, `$.FunctionName`, `$.Layers[*]`, `$.TracingConfig.Mode` |

### Threat.LAMBDA.Function.8

My IAM principal, with IAM permissions to invoke, modify, or delete an untrusted AWS Lambda function, can interact with the untrusted function, including sending data to it in the invocation payload or receiving data from it in the invocation response.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6)

**Frameworks:** MITRE ATT&CK [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): The function name or ARN on `Invoke`, `$.FunctionName`, can address a function in an account outside your organization. Your principal can then send sensitive data to that function in the invocation payload, `$.Payload`, and receive data in its response.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:DeleteFunction` | `$.FunctionName` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload` |
| `lambda:PutFunctionEventInvokeConfig` | `$.DestinationConfig.OnFailure.Destination`, `$.DestinationConfig.OnSuccess.Destination` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.S3Bucket`, `$.ZipFile` |

### Threat.LAMBDA.Function.9

IAM principal, with IAM permissions to create or modify an AWS Lambda function, can create the function in a single Availability Zone by attaching it to VPC subnets that all resolve to one zone.

**Impacts:** [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service)

**Mitigations:** [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9)

**Frameworks:** MITRE ATT&CK [T1499](https://attack.mitre.org/techniques/T1499/), [T1578.005](https://attack.mitre.org/techniques/T1578/005/)

**Scoping** (for [Service disruption from single availability zone dependency](../potential-threats.md#service-disruption-from-single-availability-zone-dependency)): An IAM principal with permissions to create or modify your function can attach it to VPC subnets that all reside in one Availability Zone. An impairment of that zone can then disrupt the function's network path, making the workload it serves unavailable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.VpcConfig.SubnetIds[*]` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.VpcConfig.SubnetIds[*]` |

### Threat.LAMBDA.Function.10

IAM principal, with IAM permissions to use my AWS Lambda function and access to an unexpected network, can interact with my function from an unexpected network.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1562.007](https://attack.mitre.org/techniques/T1562/007/)

**Scoping** (for [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)): An IAM principal with permissions to use your function can invoke, modify, or delete it from an unexpected network. AWS Lambda API calls against your function can succeed from unexpected networks as well as from your expected networks.

**Data perimeter:** `AddPermission` allows you to apply a resource-based policy to grant access to a function. The service currently does not support resource control policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:DeleteFunction` | `$.FunctionName` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.ZipFile` |

### Threat.LAMBDA.Function.11

IAM principal, with unintended IAM permissions to read environment variables, deployment package code and durable execution payloads from an AWS Lambda function that has not been encrypted at rest with a KMS customer managed key, can read plaintext environment variables, deployment package code and durable execution payloads in the function.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Unauthorized data access without customer managed KMS key encryption](../potential-threats.md#unauthorized-data-access-without-customer-managed-kms-key-encryption)): The function's environment variables, deployment package, and durable execution payloads can be encrypted under default keys you do not govern. `GetFunction`, `GetFunctionConfiguration`, and `GetDurableExecution` then return that data in plaintext to any principal holding those read permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Code.SourceKMSKeyArn`, `$.DurableConfig.KMSKeyArn`, `$.KMSKeyArn` |
| `lambda:GetDurableExecution` | `$.DurableExecutionArn`, `$.IncludeExecutionData` |
| `lambda:GetDurableExecutionHistory` | `$.DurableExecutionArn`, `$.IncludeExecutionData` |
| `lambda:GetFunction` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName`, `$.Qualifier` |
| `lambda:ListFunctions` | `$.FunctionVersion` |
| `lambda:ListVersionsByFunction` | `$.FunctionName` |
| `lambda:UpdateFunctionCode` | `$.SourceKMSKeyArn` |
| `lambda:UpdateFunctionConfiguration` | `$.DurableConfig.KMSKeyArn`, `$.KMSKeyArn` |

### Threat.LAMBDA.Function.12

Untrusted IAM principal, that has been granted access to my AWS Lambda function through its resource-based policy, can use my function, invoking it with a chosen `$.Payload` and `$.Qualifier` to run my code under its execution role, reading its environment variables and deployment package with `GetFunction`, replacing what it runs with `UpdateFunctionCode`, or deleting it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)): A resource-based policy on a Lambda function can grant actions such as `lambda:InvokeFunction`, `lambda:GetFunction`, and `lambda:UpdateFunctionCode` to an IAM principal outside your organization. That principal can then run your code under the execution role, read the environment variables and deployment package, or replace that code.

**Data perimeter:** `AddPermission` allows you to apply a resource-based policy to grant access to a function. The service currently does not support resource control policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:DeleteFunction` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetFunction` | `$.FunctionName`, `$.Qualifier` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload`, `$.Qualifier` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.ImageUri`, `$.ZipFile` |

### Threat.LAMBDA.Function.13

Untrusted IAM principal, that has been granted access to my AWS Lambda function's IAM execution role through its trust policy, can assume my execution role and act with its permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): You author the trust policy of the IAM execution role that Lambda assumes for your function. Principals outside your organization that the trust policy names may be able to assume the execution role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Role` |
| `lambda:UpdateFunctionConfiguration` | `$.Role` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.LAMBDA.Function.14

Threat actor, with possession of credentials for my AWS Lambda function execution role, can use my function execution role while on an unexpected network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Network Perimeter - Credential use from unexpected network](../potential-threats.md#network-perimeter---credential-use-from-unexpected-network)): AWS Lambda generates temporary credentials by assuming the IAM execution role you specify in the `$.Role` request parameter. A threat actor who obtains those credentials can use that execution role while on an unexpected network.

**Data perimeter:** `CreateFunction` allows you to specify a service role, referred to as the function's execution role, for your Lambda function operations. The service uses the role to make AWS API calls from your code.

**Data perimeter:** `UpdateFunctionConfiguration` allows you to specify a service role, referred to as the function's execution role, for your Lambda function operations. The service uses the role to make AWS API calls from your code.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:AddPermission` | `$.FunctionName`, `$.Principal` |
| `lambda:DeleteFunction` | `$.FunctionName` |
| `lambda:Invoke` | `$.FunctionName`, `$.Payload` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.ImageUri`, `$.ZipFile` |
| `lambda:UpdateFunctionConfiguration` | `$.Environment.Variables{key}`, `$.FunctionName`, `$.Role` |

### Threat.LAMBDA.Function.15

IAM principal, with unintended IAM permissions to read environment variables from an AWS Lambda function that is encrypted with an AWS managed KMS key in the same account as the principal, can read the environment variables in the function.

**Impacts:** [Credential Theft](../threat-impacts.md#credential-theft), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1552](https://attack.mitre.org/techniques/T1552/)

**Scoping** (for [Unauthorized data access through AWS managed KMS key](../potential-threats.md#unauthorized-data-access-through-aws-managed-kms-key)): By default, your function's environment variables are encrypted with the `aws/lambda` AWS managed key. A same-account IAM principal with unintended read permissions on the function receives those variables already decrypted in the `GetFunctionConfiguration` response.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName`, `$.Qualifier` |
| `lambda:ListFunctions` | `$.FunctionVersion` |
| `lambda:ListVersionsByFunction` | `$.FunctionName` |

### Threat.LAMBDA.Function.16

IAM principal, with unintended IAM permissions to read the .zip deployment package and function configuration details from an AWS Lambda function that has been encrypted with an AWS owned KMS key, can read the .zip deployment package and function configuration details in the function.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1530](https://attack.mitre.org/techniques/T1530/)

**Scoping** (for [Unauthorized data access through AWS owned KMS key](../potential-threats.md#unauthorized-data-access-through-aws-owned-kms-key)): By default, your function's .zip deployment package and configuration details are encrypted with an AWS owned KMS key. A principal holding `lambda:GetFunction` or `lambda:GetFunctionConfiguration` on the function can retrieve that code and those configuration details.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName`, `$.Qualifier` |

### Threat.LAMBDA.Function.17

Untrusted IAM principal, with access to configure an AWS service that has been granted access to an AWS Lambda function through its resource-based policy, may be able to cause that AWS service to interact with the function using the principal's data or context.

**Impacts:** [Privilege Escalation](../threat-impacts.md#privilege-escalation)

**Mitigations:** [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1548.005](https://attack.mitre.org/techniques/T1548/005/)

**Scoping** (for [Cross-service confused deputy problem through resource-based policies](../potential-threats.md#cross-service-confused-deputy-problem-through-resource-based-policies)): Your function's resource-based policy may grant `lambda:InvokeFunction` to AWS service principals such as `s3.amazonaws.com` or `sns.amazonaws.com`. A principal in another account can configure that service to invoke your function with event payloads they control.

**Request parameters**

| Operation | Parameters |
|---|---|
| `eventbridge:PutTargets` | `$.Rule`, `$.Targets[*].Arn` |
| `lambda:AddPermission` | `$.FunctionName`, `$.Principal` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `s3:PutBucketNotificationConfiguration` | `$.Bucket`, `$.NotificationConfiguration.CloudFunctionConfiguration[*].CloudFunction` |
| `sns:Subscribe` | `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

### Threat.LAMBDA.Function.18

IAM principal, with IAM permissions to modify an AWS Lambda function, can disable CloudWatch Logs delivery of system and application logs for the function.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): A principal with permissions to modify your function can stop delivery of its system and application logs to CloudWatch Logs. Invocations continue to run while no log records are delivered for that later activity.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.LoggingConfig.LogGroup` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.LoggingConfig.LogGroup` |

### Threat.LAMBDA.Function.19

IAM principal, with permissions to read an AWS Lambda function, which returns an Amazon S3 presigned URL signed by the AWS Lambda service principal for a service-owned bucket, can use the presigned URL to read the function's deployment package's data in the service-owned bucket while on an unexpected network.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1550.001](https://attack.mitre.org/techniques/T1550/001/)

**Scoping** (for [Unauthorized data access through service principal presigned URL](../potential-threats.md#unauthorized-data-access-through-service-principal-presigned-url)): Calling `GetFunction` for the function named in `$.FunctionName` returns an Amazon S3 presigned URL that AWS Lambda creates with a service principal identity. Anyone in possession of the URL can download your function's deployment package from a service-owned bucket without further authorization.

**Data perimeter:** `GetFunction` returns an Amazon S3 presigned URL that users can use to download AWS Lambda function deployment packages from service-owned buckets. The presigned URL is created with the AWS Lambda service principal.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName`, `$.Qualifier` |

### Threat.LAMBDA.Function.20

Threat actor, with control of an AWS Lambda function configured with overly permissive outbound security group rules, can use the function to interact with ungoverned network destinations.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Protocols:** `HTTPS`, `HTTP`, `DNS`

**Scoping** (for [Data exfiltration through overly-permissive outbound security group rules](../potential-threats.md#data-exfiltration-through-overly-permissive-outbound-security-group-rules)): A threat actor who controls a VPC-attached function can run code that opens outbound network connections. When an outbound security group rule permits an unrestricted destination range, those connections can reach arbitrary external endpoints.

### Threat.LAMBDA.Function.21

Threat actor, who can execute code in an AWS Lambda function that is attached to a subnet with a route to an internet gateway, can send the function's outbound traffic directly to the public internet instead of through a governed network egress path.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/)

**Scoping** (for [Uncontrolled internet egress through public-subnet routing](../potential-threats.md#uncontrolled-internet-egress-through-public-subnet-routing)): A threat actor able to execute code in a function can open outbound connections from the subnets you attach it to. If that subnet's route table has a route to an internet gateway, the traffic reaches the public internet outside a governed egress path.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:Invoke` | `$.FunctionName`, `$.Payload`, `$.X-Amz-Invocation-Type` |
| `lambda:InvokeWithResponseStream` | `$.FunctionName`, `$.Payload` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.ImageUri`, `$.ZipFile` |
| `lambda:UpdateFunctionConfiguration` | `$.Environment.Variables{key}` |

### Threat.LAMBDA.Layer.1

Threat actor, with a method for enumerating public AWS Lambda layers, or prior knowledge of an existing public layer, can access a layer that has been configured to allow public access, reading the layer version and retrieving a download link to its .zip archive, or attaching the layer version to a function in their own account.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2)

**Frameworks:** MITRE ATT&CK [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)): A layer version can be configured to allow public access. Any AWS account holder who discovers its ARN can then download the layer archive or attach the layer to their own function.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Layers[*]` |
| `lambda:GetLayerVersion` | `$.LayerName`, `$.VersionNumber` |
| `lambda:GetLayerVersionByArn` | `$.Arn` |
| `lambda:UpdateFunctionConfiguration` | `$.Layers[*]` |

### Threat.LAMBDA.Layer.2

My IAM principal, with IAM permissions to read, use, or modify an untrusted AWS Lambda layer, can interact with the untrusted layer, including sending data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1)

**Frameworks:** MITRE ATT&CK [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): When an account outside your organization shares a layer version with you, your IAM principals can call `GetLayerVersion` or `GetLayerVersionByArn` to download its archive. If they pass that layer version in `$.Layers[*]` on `CreateFunction` or `UpdateFunctionConfiguration`, AWS Lambda extracts its contents into your function's execution environment.

**Data perimeter:** `GetLayerVersion` might require access to service-owned layers, which are layers that are managed in service accounts.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:AddLayerVersionPermission` | `$.LayerName`, `$.OrganizationId`, `$.Principal` |
| `lambda:CreateFunction` | `$.Layers[*]` |
| `lambda:DeleteLayerVersion` | `$.LayerName`, `$.VersionNumber` |
| `lambda:GetLayerVersion` | `$.LayerName`, `$.VersionNumber` |
| `lambda:GetLayerVersionByArn` | `$.Arn` |
| `lambda:PublishLayerVersion` | `$.Content.S3Bucket`, `$.Content.S3Key`, `$.Content.S3ObjectStorageMode` |
| `lambda:RemoveLayerVersionPermission` | `$.StatementId` |
| `lambda:UpdateFunctionConfiguration` | `$.Layers[*]` |

### Threat.LAMBDA.Layer.3

Untrusted IAM principal, that has been granted access to my AWS Lambda layer through its resource-based policy, can use my layer to download the layer archive and to attach the layer version to functions in accounts I do not control.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1530](https://attack.mitre.org/techniques/T1530/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)): The resource-based policy on your Lambda layer version can name another account, the account root user, or a wildcard principal. An identity outside your organization with that grant can call `GetLayerVersion` to download the archive, or add the version to a function it controls.

**Data perimeter:** `AddLayerVersionPermission` allows you to apply a resource-based policy to grant access to a layer version. The service currently does not support resource control policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Layers[*]` |
| `lambda:GetLayerVersion` | `$.LayerName`, `$.VersionNumber` |
| `lambda:GetLayerVersionByArn` | `$.Arn` |
| `lambda:UpdateFunctionConfiguration` | `$.Layers[*]` |

---

## Mitigations

Each scoping statement below names the capability the mitigation goal governs on the entity and the objective of governing it. Each mitigation's implementations describe how to govern or monitor that capability.

### Mitigation.LAMBDA.1

AWS Lambda service API calls on a governed VPC network should only be made against trusted resources or AWS service-owned resources

**Goal:** [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)

**Addresses:** [Threat.LAMBDA.2](#threatlambda2)

**Implemented by:** [Implementation.LAMBDA.1](#implementationlambda1)

**Scoping** (for [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)): You can call the AWS Lambda API from your VPC through an interface VPC endpoint powered by AWS PrivateLink. Calls over that network path can address Lambda resources in any account, including accounts outside your organization.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.LAMBDA.2

AWS Lambda service API calls on a governed VPC network should only be made by trusted identities or AWS services

**Goal:** [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)

**Addresses:** [Threat.LAMBDA.1](#threatlambda1)

**Implemented by:** [Implementation.LAMBDA.2](#implementationlambda2)

**Scoping** (for [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)): You can call the AWS Lambda API from your VPC through an interface VPC endpoint powered by AWS PrivateLink. Calls over that network path can be made by IAM principals from any account, including accounts outside your organization.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.LAMBDA.CapacityProvider.1

An AWS Lambda capacity provider should be configured to send system logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.LAMBDA.CapacityProvider.10](#threatlambdacapacityprovider10)

**Implemented by:** [Implementation.LAMBDA.CapacityProvider.1](#implementationlambdacapacityprovider1), [Implementation.LAMBDA.CapacityProvider.2](#implementationlambdacapacityprovider2), [Implementation.LAMBDA.CapacityProvider.3](#implementationlambdacapacityprovider3)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): A capacity provider produces system logs about the compute resources that Lambda provisions and manages for you. You can configure delivery of those logs to CloudWatch Logs on the capacity provider itself.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.TelemetryConfig.LoggingConfig.LogGroup`, `$.TelemetryConfig.LoggingConfig.SystemLogLevel` |
| `lambda:UpdateCapacityProvider` | `$.CapacityProviderName`, `$.TelemetryConfig.LoggingConfig.LogGroup`, `$.TelemetryConfig.LoggingConfig.SystemLogLevel` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetCapacityProvider` | `$.CapacityProviderName` |

### Mitigation.LAMBDA.CapacityProvider.2

An AWS Lambda capacity provider's operator role should be assumed only by AWS Lambda acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.LAMBDA.CapacityProvider.9](#threatlambdacapacityprovider9)

**Implemented by:** [Implementation.LAMBDA.CapacityProvider.4](#implementationlambdacapacityprovider4), [Implementation.LAMBDA.CapacityProvider.5](#implementationlambdacapacityprovider5), [Implementation.LAMBDA.CapacityProvider.6](#implementationlambdacapacityprovider6), [Implementation.LAMBDA.CapacityProvider.7](#implementationlambdacapacityprovider7)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): Your capacity provider's operator role, set through the `$.PermissionsConfig.CapacityProviderOperatorRoleArn` request parameter, has a trust policy that specifies which principals can assume it. Allowing only AWS Lambda, acting on behalf of trusted resources, to assume this role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.PermissionsConfig.CapacityProviderOperatorRoleArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetCapacityProvider` | `$.CapacityProviderName` |
| `lambda:ListCapacityProviders` | *(any parameter)* |

### Mitigation.LAMBDA.CapacityProvider.3

An AWS Lambda capacity provider should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.LAMBDA.CapacityProvider.7](#threatlambdacapacityprovider7), [Threat.LAMBDA.CapacityProvider.12](#threatlambdacapacityprovider12)

**Implemented by:** [Implementation.LAMBDA.CapacityProvider.8](#implementationlambdacapacityprovider8), [Implementation.LAMBDA.CapacityProvider.9](#implementationlambdacapacityprovider9)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): You select the subnets for a capacity provider through `$.VpcConfig.SubnetIds[*]`. Private subnet placement helps reduce the network exposure of that capacity provider.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.VpcConfig.SubnetIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetCapacityProvider` | `$.CapacityProviderName` |
| `lambda:ListCapacityProviders` | *(any parameter)* |

### Mitigation.LAMBDA.CapacityProvider.4

An AWS Lambda capacity provider should use an EC2 instance type that supports automatic encryption of instance-to-instance traffic

**Goal:** [Use Nitro instances](../mitigation-goals.md#use-nitro-instances)

**Addresses:** [Threat.LAMBDA.CapacityProvider.8](#threatlambdacapacityprovider8)

**Implemented by:** [Implementation.LAMBDA.CapacityProvider.10](#implementationlambdacapacityprovider10), [Implementation.LAMBDA.CapacityProvider.11](#implementationlambdacapacityprovider11)

**Scoping** (for [Use Nitro instances](../mitigation-goals.md#use-nitro-instances)): A capacity provider runs your function code on Amazon EC2 instances placed in your own VPC. The instance type backing that pool determines whether traffic to other instances you operate is automatically encrypted in the Nitro System hardware.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.InstanceRequirements.AllowedInstanceTypes[*]`, `$.InstanceRequirements.ExcludedInstanceTypes[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetCapacityProvider` | `$.CapacityProviderName` |
| `lambda:ListCapacityProviders` | *(any parameter)* |

### Mitigation.LAMBDA.CapacityProvider.5

An AWS Lambda capacity provider should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.LAMBDA.CapacityProvider.1](#threatlambdacapacityprovider1), [Threat.LAMBDA.CapacityProvider.3](#threatlambdacapacityprovider3)

**Implemented by:** [Implementation.LAMBDA.CapacityProvider.12](#implementationlambdacapacityprovider12), [Implementation.LAMBDA.CapacityProvider.13](#implementationlambdacapacityprovider13)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): A capacity provider accepts a KMS key ARN through `$.KmsKeyArn`, which encrypts the Amazon EBS storage on the Managed Instances it provisions. When you omit this parameter, encryption defaults to an AWS managed key instead of a key you control.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.KmsKeyArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetCapacityProvider` | `$.CapacityProviderName` |
| `lambda:ListCapacityProviders` | *(any parameter)* |

### Mitigation.LAMBDA.CapacityProvider.6

An AWS Lambda capacity provider should launch its instances in subnets in more than one Availability Zone

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.LAMBDA.CapacityProvider.4](#threatlambdacapacityprovider4)

**Implemented by:** [Implementation.LAMBDA.CapacityProvider.14](#implementationlambdacapacityprovider14), [Implementation.LAMBDA.CapacityProvider.15](#implementationlambdacapacityprovider15), [Implementation.LAMBDA.CapacityProvider.16](#implementationlambdacapacityprovider16)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): A capacity provider defines the VPC subnets where Lambda launches and manages the instances that run your functions. Lambda creates and attaches network interfaces for those instances, placing them in the Availability Zones of those subnets.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.VpcConfig.SubnetIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetCapacityProvider` | `$.CapacityProviderName` |
| `lambda:ListCapacityProviders` | *(any parameter)* |

### Mitigation.LAMBDA.CapacityProvider.7

An AWS Lambda capacity provider should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.LAMBDA.CapacityProvider.2](#threatlambdacapacityprovider2)

**Implemented by:** [Implementation.LAMBDA.CapacityProvider.17](#implementationlambdacapacityprovider17), [Implementation.LAMBDA.CapacityProvider.18](#implementationlambdacapacityprovider18), [Implementation.LAMBDA.CapacityProvider.19](#implementationlambdacapacityprovider19)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): A capacity provider is a durably persisted, account-level resource with its own ARN, so it supports resource tagging directly. Tags that record owner and purpose help you identify who is accountable for each compute pool.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.Tags{key}` |
| `lambda:TagResource` | `$.Resource`, `$.Tags{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:ListTags` | `$.Resource` |

### Mitigation.LAMBDA.CapacityProvider.8

An AWS Lambda capacity provider should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.LAMBDA.CapacityProvider.5](#threatlambdacapacityprovider5), [Threat.LAMBDA.CapacityProvider.6](#threatlambdacapacityprovider6), [Threat.LAMBDA.CapacityProvider.11](#threatlambdacapacityprovider11)

**Implemented by:** [Implementation.LAMBDA.CapacityProvider.20](#implementationlambdacapacityprovider20), [Implementation.LAMBDA.CapacityProvider.21](#implementationlambdacapacityprovider21), [Implementation.LAMBDA.CapacityProvider.22](#implementationlambdacapacityprovider22)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): You supply security groups for a capacity provider through `$.VpcConfig.SecurityGroupIds[*]`. Those groups control network access for the compute instances that run your function code.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCapacityProvider` | `$.CapacityProviderName`, `$.VpcConfig.SecurityGroupIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetCapacityProvider` | `$.CapacityProviderName` |

### Mitigation.LAMBDA.CodeSigningConfig.1

An AWS Lambda code signing config should only be configured with a trusted AWS Signer signing profile version

**Goal:** [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets)

**Addresses:** [Threat.LAMBDA.CodeSigningConfig.1](#threatlambdacodesigningconfig1)

**Implemented by:** [Implementation.LAMBDA.CodeSigningConfig.1](#implementationlambdacodesigningconfig1), [Implementation.LAMBDA.CodeSigningConfig.2](#implementationlambdacodesigningconfig2), [Implementation.LAMBDA.CodeSigningConfig.3](#implementationlambdacodesigningconfig3)

**Scoping** (for [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets)): A code signing config sets its allowed publishers through `$.AllowedPublishers.SigningProfileVersionArns[*]`. Each entry names an AWS Signer signing profile version that can belong to a different account.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCodeSigningConfig` | `$.AllowedPublishers.SigningProfileVersionArns[*]` |
| `lambda:UpdateCodeSigningConfig` | `$.AllowedPublishers.SigningProfileVersionArns[*]`, `$.CodeSigningConfigArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetCodeSigningConfig` | `$.CodeSigningConfigArn` |
| `lambda:ListCodeSigningConfigs` | *(any parameter)* |

### Mitigation.LAMBDA.CodeSigningConfig.2

An AWS Lambda code signing config should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.LAMBDA.CodeSigningConfig.2](#threatlambdacodesigningconfig2)

**Implemented by:** [Implementation.LAMBDA.CodeSigningConfig.4](#implementationlambdacodesigningconfig4), [Implementation.LAMBDA.CodeSigningConfig.5](#implementationlambdacodesigningconfig5), [Implementation.LAMBDA.CodeSigningConfig.6](#implementationlambdacodesigningconfig6)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): A code signing config is a durable, independently addressable Lambda resource with its own Amazon Resource Name. Lambda supports native tags on this resource to record owner and purpose information.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateCodeSigningConfig` | `$.Tags{key}` |
| `lambda:TagResource` | `$.Resource`, `$.Tags{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:ListTags` | `$.Resource` |

### Mitigation.LAMBDA.EventSourceMapping.1

An AWS Lambda event source mapping should be configured to send system logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.LAMBDA.EventSourceMapping.9](#threatlambdaeventsourcemapping9), [Threat.LAMBDA.EventSourceMapping.11](#threatlambdaeventsourcemapping11)

**Implemented by:** [Implementation.LAMBDA.EventSourceMapping.1](#implementationlambdaeventsourcemapping1), [Implementation.LAMBDA.EventSourceMapping.2](#implementationlambdaeventsourcemapping2), [Implementation.LAMBDA.EventSourceMapping.3](#implementationlambdaeventsourcemapping3)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): An event source mapping supports system-level logging for its event pollers, with those records delivered to CloudWatch Logs. Each mapping has its own log stream for reviewing poller behavior.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.LoggingConfig.SystemLogLevel` |
| `lambda:UpdateEventSourceMapping` | `$.LoggingConfig.SystemLogLevel`, `$.UUID` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetEventSourceMapping` | `$.UUID` |
| `lambda:ListEventSourceMappings` | `$.FunctionName` |

### Mitigation.LAMBDA.EventSourceMapping.2

An AWS Lambda event source mapping should be in an Amazon VPC

**Goal:** [VPC Deployment](../mitigation-goals.md#vpc-deployment)

**Addresses:** [Threat.LAMBDA.EventSourceMapping.1](#threatlambdaeventsourcemapping1)

**Implemented by:** [Implementation.LAMBDA.EventSourceMapping.4](#implementationlambdaeventsourcemapping4), [Implementation.LAMBDA.EventSourceMapping.5](#implementationlambdaeventsourcemapping5)

**Scoping** (for [VPC Deployment](../mitigation-goals.md#vpc-deployment)): An event source mapping can reach a private event source through network interfaces that Lambda places in your VPC subnets. This network placement belongs to the mapping itself, and is separate from the target function's VPC configuration.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI` |
| `lambda:UpdateEventSourceMapping` | `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI`, `$.UUID` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetEventSourceMapping` | `$.UUID` |
| `lambda:ListEventSourceMappings` | `$.EventSourceArn`, `$.FunctionName` |

### Mitigation.LAMBDA.EventSourceMapping.3

An AWS Lambda event source mapping's filter criteria should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.LAMBDA.EventSourceMapping.3](#threatlambdaeventsourcemapping3), [Threat.LAMBDA.EventSourceMapping.5](#threatlambdaeventsourcemapping5), [Threat.LAMBDA.EventSourceMapping.8](#threatlambdaeventsourcemapping8), [Threat.LAMBDA.EventSourceMapping.10](#threatlambdaeventsourcemapping10)

**Implemented by:** [Implementation.LAMBDA.EventSourceMapping.6](#implementationlambdaeventsourcemapping6), [Implementation.LAMBDA.EventSourceMapping.7](#implementationlambdaeventsourcemapping7)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): An event source mapping can hold a filter criteria object that determines which records Lambda passes to your function. By default, that object is not encrypted with a key that you manage.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.KMSKeyArn` |
| `lambda:UpdateEventSourceMapping` | `$.KMSKeyArn`, `$.UUID` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetEventSourceMapping` | `$.UUID` |
| `lambda:ListEventSourceMappings` | `$.FunctionName` |

### Mitigation.LAMBDA.EventSourceMapping.4

An AWS Lambda event source mapping should have Elastic Network Interfaces (ENIs) in more than one Availability Zone (AZ)

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.LAMBDA.EventSourceMapping.7](#threatlambdaeventsourcemapping7)

**Implemented by:** [Implementation.LAMBDA.EventSourceMapping.4](#implementationlambdaeventsourcemapping4), [Implementation.LAMBDA.EventSourceMapping.8](#implementationlambdaeventsourcemapping8), [Implementation.LAMBDA.EventSourceMapping.9](#implementationlambdaeventsourcemapping9)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): An event source mapping supports a VPC subnet source access configuration for connecting to your event source. That configuration determines the Availability Zones of the mapping's network interfaces.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI` |
| `lambda:UpdateEventSourceMapping` | `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI`, `$.UUID` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetEventSourceMapping` | `$.UUID` |
| `lambda:ListEventSourceMappings` | `$.FunctionName` |

### Mitigation.LAMBDA.EventSourceMapping.5

An AWS Lambda event source mapping should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.LAMBDA.EventSourceMapping.4](#threatlambdaeventsourcemapping4)

**Implemented by:** [Implementation.LAMBDA.EventSourceMapping.10](#implementationlambdaeventsourcemapping10), [Implementation.LAMBDA.EventSourceMapping.11](#implementationlambdaeventsourcemapping11), [Implementation.LAMBDA.EventSourceMapping.12](#implementationlambdaeventsourcemapping12)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): An event source mapping is a persistent AWS Lambda resource that supports tags on its own ARN. It does not inherit tags from the function it invokes.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.Tags{key}` |
| `lambda:TagResource` | `$.Resource`, `$.Tags{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:ListTags` | `$.Resource` |

### Mitigation.LAMBDA.EventSourceMapping.6

An AWS Lambda event source mapping should have the `EventCount`, `ErrorCount`, and `KafkaMetrics` event source mapping metric groups enabled to collect detailed CloudWatch metrics

**Goal:** [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)

**Addresses:** [Threat.LAMBDA.EventSourceMapping.2](#threatlambdaeventsourcemapping2), [Threat.LAMBDA.EventSourceMapping.6](#threatlambdaeventsourcemapping6)

**Implemented by:** [Implementation.LAMBDA.EventSourceMapping.13](#implementationlambdaeventsourcemapping13), [Implementation.LAMBDA.EventSourceMapping.14](#implementationlambdaeventsourcemapping14), [Implementation.LAMBDA.EventSourceMapping.15](#implementationlambdaeventsourcemapping15)

**Scoping** (for [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)): An event source mapping supports an opt-in metrics configuration offering the `EventCount`, `ErrorCount`, and `KafkaMetrics` metric groups. Lambda publishes the groups you select to CloudWatch.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.MetricsConfig.Metrics[*]` |
| `lambda:UpdateEventSourceMapping` | `$.MetricsConfig.Metrics[*]`, `$.UUID` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetEventSourceMapping` | `$.UUID` |
| `lambda:ListEventSourceMappings` | `$.FunctionName` |

### Mitigation.LAMBDA.EventSourceMapping.7

An AWS Lambda event source mapping should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.LAMBDA.EventSourceMapping.12](#threatlambdaeventsourcemapping12)

**Implemented by:** [Implementation.LAMBDA.EventSourceMapping.16](#implementationlambdaeventsourcemapping16), [Implementation.LAMBDA.EventSourceMapping.17](#implementationlambdaeventsourcemapping17), [Implementation.LAMBDA.EventSourceMapping.18](#implementationlambdaeventsourcemapping18)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): An event source mapping for a self-managed Apache Kafka cluster can include your VPC security groups as `VPC_SECURITY_GROUP` source access configurations. These groups control inbound and outbound traffic on the network interfaces that Lambda creates for the mapping.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateEventSourceMapping` | `$.FunctionName`, `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI` |
| `lambda:UpdateEventSourceMapping` | `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI`, `$.UUID` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetEventSourceMapping` | `$.UUID` |
| `lambda:ListEventSourceMappings` | `$.EventSourceArn`, `$.FunctionName` |

### Mitigation.LAMBDA.Function.1

An AWS Lambda function should be configured to send application and system logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.LAMBDA.Function.18](#threatlambdafunction18)

**Implemented by:** [Implementation.LAMBDA.Function.1](#implementationlambdafunction1), [Implementation.LAMBDA.Function.2](#implementationlambdafunction2), [Implementation.LAMBDA.Function.3](#implementationlambdafunction3), [ManagedControl.LAMBDA.Function.1](#managedcontrollambdafunction1)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): A Lambda function runs your code and emits both application and system logs that it can send to CloudWatch Logs. System logs record per-invocation activity, and application logs carry the diagnostic output your code emits.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.LoggingConfig.ApplicationLogLevel`, `$.LoggingConfig.LogFormat`, `$.LoggingConfig.LogGroup`, `$.LoggingConfig.SystemLogLevel` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.LoggingConfig.ApplicationLogLevel`, `$.LoggingConfig.LogFormat`, `$.LoggingConfig.LogGroup`, `$.LoggingConfig.SystemLogLevel` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName` |
| `lambda:ListFunctions` | *(any parameter)* |

### Mitigation.LAMBDA.Function.2

An AWS Lambda function's execution role should be restricted to accessing resources only from expected networks

**Goal:** [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)

**Addresses:** [Threat.LAMBDA.Function.14](#threatlambdafunction14)

**Implemented by:** [Implementation.LAMBDA.Function.4](#implementationlambdafunction4)

**Scoping** (for [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks)): Your function calls AWS services using the IAM execution role you specify through the `$.Role` request parameter. Lambda provides that role's temporary credentials to your function code during each invocation.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.Role` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.Role` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName` |
| `lambda:ListFunctions` | *(any parameter)* |
| `lambda:ListVersionsByFunction` | `$.FunctionName` |

### Mitigation.LAMBDA.Function.3

An AWS Lambda function's IAM execution role should be assumed only by AWS Lambda acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.LAMBDA.Function.13](#threatlambdafunction13)

**Implemented by:** [Implementation.LAMBDA.Function.5](#implementationlambdafunction5), [Implementation.LAMBDA.Function.6](#implementationlambdafunction6), [Implementation.LAMBDA.Function.7](#implementationlambdafunction7), [Implementation.LAMBDA.Function.8](#implementationlambdafunction8)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): Your Lambda function's IAM execution role, set through the `$.Role` request parameter, has a trust policy you author to control who can assume it. Allowing only AWS Lambda, acting on behalf of trusted resources, to assume the role helps prevent an unintended principal from assuming it.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.Role` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.Role` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName`, `$.Qualifier` |
| `lambda:ListVersionsByFunction` | `$.FunctionName` |

### Mitigation.LAMBDA.Function.4

An AWS Lambda function should be placed only in private subnets

**Goal:** [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)

**Addresses:** [Threat.LAMBDA.Function.21](#threatlambdafunction21)

**Implemented by:** [Implementation.LAMBDA.Function.9](#implementationlambdafunction9), [Implementation.LAMBDA.Function.10](#implementationlambdafunction10), [ManagedControl.LAMBDA.Function.2](#managedcontrollambdafunction2), [ManagedControl.LAMBDA.Function.3](#managedcontrollambdafunction3)

**Scoping** (for [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement)): You attach a function to subnets in your VPC through `$.VpcConfig.SubnetIds[*]`, so subnet placement is a property you control on the function itself. Because you choose those subnets, you can limit a function's placement to private subnets.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.VpcConfig.Ipv6AllowedForDualStack`, `$.VpcConfig.SubnetIds[*]` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.VpcConfig.Ipv6AllowedForDualStack`, `$.VpcConfig.SubnetIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName` |
| `lambda:ListFunctions` | *(any parameter)* |

### Mitigation.LAMBDA.Function.5

An AWS Lambda function should be accessed only from expected networks

**Goal:** [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)

**Addresses:** [Threat.LAMBDA.Function.10](#threatlambdafunction10)

**Implemented by:** [Implementation.LAMBDA.Function.11](#implementationlambdafunction11), [Implementation.LAMBDA.Function.12](#implementationlambdafunction12), [Implementation.LAMBDA.Function.13](#implementationlambdafunction13), [ManagedControl.LAMBDA.Function.4](#managedcontrollambdafunction4)

**Scoping** (for [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)): AWS Lambda API calls against your function can arrive from your VPCs and on-premises networks or from unexpected networks. Restricting where requests to your function originate from enforces the network perimeter and helps keep your function accessible only from expected networks.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetPolicy` | `$.FunctionName` |
| `lambda:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.LAMBDA.Function.6

IAM principals should be restricted to accessing only trusted AWS Lambda function resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.LAMBDA.Function.8](#threatlambdafunction8)

**Implemented by:** [Implementation.LAMBDA.Function.14](#implementationlambdafunction14)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): Your identities can direct AWS Lambda API calls, including invocations, at function ARNs in accounts outside your organization. The resource perimeter objective is that your identities can access only trusted resources.

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

### Mitigation.LAMBDA.Function.7

An AWS Lambda function should be in an Amazon VPC

**Goal:** [VPC Deployment](../mitigation-goals.md#vpc-deployment)

**Addresses:** [Threat.LAMBDA.Function.1](#threatlambdafunction1)

**Implemented by:** [Implementation.LAMBDA.Function.15](#implementationlambdafunction15), [Implementation.LAMBDA.Function.16](#implementationlambdafunction16), [Implementation.LAMBDA.Function.17](#implementationlambdafunction17), [Implementation.LAMBDA.Function.18](#implementationlambdafunction18), [ManagedControl.LAMBDA.Function.2](#managedcontrollambdafunction2), [ManagedControl.LAMBDA.Function.3](#managedcontrollambdafunction3)

**Scoping** (for [VPC Deployment](../mitigation-goals.md#vpc-deployment)): By default, an AWS Lambda function runs outside your VPC with public internet access. VPC attachment is an optional, reversible part of a function's configuration.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.VpcConfig.SecurityGroupIds[*]`, `$.VpcConfig.SubnetIds[*]` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.VpcConfig.SecurityGroupIds[*]`, `$.VpcConfig.SubnetIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName`, `$.Qualifier` |
| `lambda:ListFunctions` | *(any parameter)* |

### Mitigation.LAMBDA.Function.8

An AWS Lambda function should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.LAMBDA.Function.3](#threatlambdafunction3), [Threat.LAMBDA.Function.5](#threatlambdafunction5), [Threat.LAMBDA.Function.11](#threatlambdafunction11), [Threat.LAMBDA.Function.15](#threatlambdafunction15), [Threat.LAMBDA.Function.16](#threatlambdafunction16)

**Implemented by:** [Implementation.LAMBDA.Function.19](#implementationlambdafunction19), [Implementation.LAMBDA.Function.20](#implementationlambdafunction20)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): A Lambda function accepts customer managed key identifiers directly on its own create and update operations. These keys encrypt the function's environment variables, SnapStart snapshots, .zip deployment package, optimized container image, and durable execution payload data.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Code.SourceKMSKeyArn`, `$.DurableConfig.KMSKeyArn`, `$.FunctionName`, `$.KMSKeyArn` |
| `lambda:UpdateFunctionCode` | `$.FunctionName`, `$.SourceKMSKeyArn` |
| `lambda:UpdateFunctionConfiguration` | `$.DurableConfig.KMSKeyArn`, `$.FunctionName`, `$.KMSKeyArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName` |
| `lambda:ListVersionsByFunction` | `$.FunctionName` |

### Mitigation.LAMBDA.Function.9

An AWS Lambda function should have Elastic Network Interfaces (ENIs) in more than one Availability Zone (AZ)

**Goal:** [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)

**Addresses:** [Threat.LAMBDA.Function.9](#threatlambdafunction9)

**Implemented by:** [Implementation.LAMBDA.Function.21](#implementationlambdafunction21), [Implementation.LAMBDA.Function.22](#implementationlambdafunction22), [Implementation.LAMBDA.Function.23](#implementationlambdafunction23), [ManagedControl.LAMBDA.Function.5](#managedcontrollambdafunction5)

**Scoping** (for [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment)): When you connect a function to a VPC, Lambda creates a Hyperplane ENI for each subnet and security group combination you configure. The subnets you list in `$.VpcConfig.SubnetIds[*]` therefore determine which Availability Zones those network interfaces occupy.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.VpcConfig.SubnetIds[*]` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.VpcConfig.SubnetIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName` |
| `lambda:ListFunctions` | *(any parameter)* |

### Mitigation.LAMBDA.Function.10

An AWS Lambda function's resource-based policy should limit permissions to a subset of IAM principals, rather than granting access to all IAM principals

**Goal:** [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)

**Addresses:** [Threat.LAMBDA.Function.6](#threatlambdafunction6)

**Implemented by:** [Implementation.LAMBDA.Function.24](#implementationlambdafunction24), [Implementation.LAMBDA.Function.25](#implementationlambdafunction25), [Implementation.LAMBDA.Function.26](#implementationlambdafunction26), [Implementation.LAMBDA.Function.27](#implementationlambdafunction27), [ManagedControl.LAMBDA.Function.6](#managedcontrollambdafunction6), [ManagedControl.LAMBDA.Function.7](#managedcontrollambdafunction7)

**Scoping** (for [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)): An AWS Lambda function has its own resource-based policy that grants IAM principals permission to invoke it. This policy can be written to grant invoke access to all IAM principals rather than a named subset.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:AddPermission` | `$.FunctionName`, `$.Principal` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `lambda:RemovePermission` | `$.FunctionName`, `$.StatementId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetPolicy` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.LAMBDA.Function.11

An AWS Lambda function should not allow unauthenticated access through its function URL or resource-based policy

**Goal:** [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention)

**Addresses:** [Threat.LAMBDA.Function.6](#threatlambdafunction6)

**Implemented by:** [Implementation.LAMBDA.Function.28](#implementationlambdafunction28), [Implementation.LAMBDA.Function.29](#implementationlambdafunction29), [Implementation.LAMBDA.Function.30](#implementationlambdafunction30), [Implementation.LAMBDA.Function.31](#implementationlambdafunction31), [Implementation.LAMBDA.Function.32](#implementationlambdafunction32), [ManagedControl.LAMBDA.Function.4](#managedcontrollambdafunction4), [ManagedControl.LAMBDA.Function.6](#managedcontrollambdafunction6), [ManagedControl.LAMBDA.Function.7](#managedcontrollambdafunction7)

**Scoping** (for [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention)): A Lambda function can expose a function URL, a dedicated HTTPS endpoint whose `$.AuthType` setting controls whether Lambda authenticates callers. That setting and your function's resource-based policy together determine whether identities outside your organization can invoke your function.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:AddPermission` | `$.FunctionName`, `$.FunctionUrlAuthType`, `$.Principal` |
| `lambda:CreateFunctionUrlConfig` | `$.AuthType`, `$.FunctionName` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `lambda:UpdateFunctionUrlConfig` | `$.AuthType`, `$.FunctionName` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunctionUrlConfig` | `$.FunctionName` |
| `lambda:GetPolicy` | `$.FunctionName` |
| `lambda:GetResourcePolicy` | `$.ResourceArn` |
| `lambda:ListFunctionUrlConfigs` | `$.FunctionName` |

### Mitigation.LAMBDA.Function.12

An AWS Lambda function should be accessed only by trusted identities or AWS services acting on behalf of trusted resources

**Goal:** [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)

**Addresses:** [Threat.LAMBDA.Function.12](#threatlambdafunction12)

**Implemented by:** [Implementation.LAMBDA.Function.13](#implementationlambdafunction13), [Implementation.LAMBDA.Function.33](#implementationlambdafunction33), [Implementation.LAMBDA.Function.34](#implementationlambdafunction34), [Implementation.LAMBDA.Function.35](#implementationlambdafunction35), [Implementation.LAMBDA.Function.36](#implementationlambdafunction36), [Implementation.LAMBDA.Function.37](#implementationlambdafunction37), [Implementation.LAMBDA.Function.38](#implementationlambdafunction38), [ManagedControl.LAMBDA.Function.4](#managedcontrollambdafunction4), [ManagedControl.LAMBDA.Function.6](#managedcontrollambdafunction6), [ManagedControl.LAMBDA.Function.7](#managedcontrollambdafunction7)

**Scoping** (for [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)): Your Lambda function supports a resource-based policy that defines which identities and AWS services can invoke or manage it. Because access to the function is authorized through IAM, you can restrict it to trusted identities.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:AddPermission` | `$.FunctionName`, `$.Principal`, `$.PrincipalOrgID`, `$.SourceAccount`, `$.SourceArn` |
| `lambda:DeleteResourcePolicy` | `$.ResourceArn` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `lambda:RemovePermission` | `$.FunctionName`, `$.StatementId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetPolicy` | `$.FunctionName`, `$.Qualifier` |
| `lambda:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.LAMBDA.Function.13

An AWS Lambda function should be accessed by an AWS service principal, such as the Amazon S3 service principal, only when the service principal is acting on behalf of a trusted resource, AWS account, or organization

**Goal:** [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies)

**Addresses:** [Threat.LAMBDA.Function.17](#threatlambdafunction17)

**Implemented by:** [Implementation.LAMBDA.Function.39](#implementationlambdafunction39), [Implementation.LAMBDA.Function.40](#implementationlambdafunction40), [Implementation.LAMBDA.Function.41](#implementationlambdafunction41), [Implementation.LAMBDA.Function.42](#implementationlambdafunction42), [ManagedControl.LAMBDA.Function.6](#managedcontrollambdafunction6), [ManagedControl.LAMBDA.Function.7](#managedcontrollambdafunction7)

**Scoping** (for [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies)): You can attach a resource-based policy to an AWS Lambda function that grants an AWS service principal permission to invoke it. A grant to a service principal alone does not identify the resource or account the service is acting on behalf of.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:AddPermission` | `$.FunctionName`, `$.Principal`, `$.SourceAccount`, `$.SourceArn` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetPolicy` | `$.FunctionName` |
| `lambda:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.LAMBDA.Function.14

Permissions to generate presigned URLs signed by the AWS Lambda service principal that grant access to a Lambda function's deployment package should be restricted to select principals

**Goal:** [Service Principal Presigned URL](../mitigation-goals.md#service-principal-presigned-url)

**Addresses:** [Threat.LAMBDA.Function.19](#threatlambdafunction19)

**Implemented by:** [Implementation.LAMBDA.Function.43](#implementationlambdafunction43), [Implementation.LAMBDA.Function.44](#implementationlambdafunction44)

**Scoping** (for [Service Principal Presigned URL](../mitigation-goals.md#service-principal-presigned-url)): `GetFunction` returns an Amazon S3 presigned URL for downloading your function's deployment package from a service-owned bucket. The presigned URL is created with an AWS Lambda service principal identity, and you cannot apply a bucket policy to that AWS-managed bucket.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:AddPermission` | `$.Action`, `$.FunctionName`, `$.Principal`, `$.PrincipalOrgID` |
| `lambda:PutResourcePolicy` | `$.Policy`, `$.ResourceArn` |
| `lambda:RemovePermission` | `$.FunctionName`, `$.StatementId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetPolicy` | `$.FunctionName` |
| `lambda:GetResourcePolicy` | `$.ResourceArn` |

### Mitigation.LAMBDA.Function.15

An AWS Lambda function should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.LAMBDA.Function.4](#threatlambdafunction4)

**Implemented by:** [Implementation.LAMBDA.Function.45](#implementationlambdafunction45), [Implementation.LAMBDA.Function.46](#implementationlambdafunction46), [Implementation.LAMBDA.Function.47](#implementationlambdafunction47)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): You can tag a Lambda function when you call `CreateFunction` or later by calling `TagResource` with the function ARN. Tagging each function with owner and purpose values helps you attribute it to a responsible team and project.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.Tags{key}` |
| `lambda:TagResource` | `$.Resource`, `$.Tags{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:ListTags` | `$.Resource` |

### Mitigation.LAMBDA.Function.16

An AWS Lambda function should have CloudWatch Lambda Insights enhanced monitoring and CloudWatch Application Signals enabled to collect detailed CloudWatch metrics

**Goal:** [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)

**Addresses:** [Threat.LAMBDA.Function.2](#threatlambdafunction2), [Threat.LAMBDA.Function.7](#threatlambdafunction7)

**Implemented by:** [Implementation.LAMBDA.Function.48](#implementationlambdafunction48), [Implementation.LAMBDA.Function.49](#implementationlambdafunction49), [Implementation.LAMBDA.Function.50](#implementationlambdafunction50)

**Scoping** (for [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics)): Amazon CloudWatch Lambda Insights can publish system-level metrics, such as CPU time, memory, `/tmp` disk, and network usage, beyond your function's default invocation metrics. Amazon CloudWatch Application Signals can also publish request-level metrics that the default metrics do not include.

**Reference:** If another monitoring platform already collects equivalent detailed metrics for this resource, that platform can provide the same visibility for anomaly detection and incident response. Enable this feature when no equivalent collection is in place.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.Environment.Variables{key}`, `$.FunctionName`, `$.Layers[*]` |
| `lambda:UpdateFunctionConfiguration` | `$.Environment.Variables{key}`, `$.FunctionName`, `$.Layers[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName` |

### Mitigation.LAMBDA.Function.17

An AWS Lambda function should be associated only with security groups whose inbound and outbound rules restrict traffic to only the required sources, destinations, protocols, and ports

**Goal:** [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)

**Addresses:** [Threat.LAMBDA.Function.20](#threatlambdafunction20)

**Implemented by:** [Implementation.LAMBDA.Function.51](#implementationlambdafunction51), [Implementation.LAMBDA.Function.52](#implementationlambdafunction52), [Implementation.LAMBDA.Function.53](#implementationlambdafunction53)

**Scoping** (for [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access)): A Lambda function that connects to your VPC is associated with security groups you choose through `$.VpcConfig.SecurityGroupIds[*]`. Those groups control the network traffic allowed for your function, including the outbound connections it initiates.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:CreateFunction` | `$.FunctionName`, `$.VpcConfig.SecurityGroupIds[*]` |
| `lambda:UpdateFunctionConfiguration` | `$.FunctionName`, `$.VpcConfig.SecurityGroupIds[*]` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetFunction` | `$.FunctionName` |
| `lambda:GetFunctionConfiguration` | `$.FunctionName` |
| `lambda:ListFunctions` | *(any parameter)* |
| `lambda:ListVersionsByFunction` | `$.FunctionName` |

### Mitigation.LAMBDA.Layer.1

IAM principals should be restricted to accessing only trusted AWS Lambda layer resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.LAMBDA.Layer.2](#threatlambdalayer2)

**Implemented by:** [Implementation.LAMBDA.Layer.1](#implementationlambdalayer1)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): Layer versions can be shared and consumed by other accounts, so your identities can read or attach one owned outside your organization. A resource perimeter helps your identities access only trusted resources: those owned by your AWS accounts or by AWS services acting on your behalf.

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

### Mitigation.LAMBDA.Layer.2

An AWS Lambda layer should limit permissions to a subset of IAM principals, rather than granting access to all IAM principals

**Goal:** [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)

**Addresses:** [Threat.LAMBDA.Layer.1](#threatlambdalayer1)

**Implemented by:** [Implementation.LAMBDA.Layer.2](#implementationlambdalayer2), [Implementation.LAMBDA.Layer.3](#implementationlambdalayer3), [ManagedControl.LAMBDA.Layer.1](#managedcontrollambdalayer1)

**Scoping** (for [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)): A Lambda layer version can carry its own resource-based policy that grants layer usage permission to other accounts. That policy accepts a wildcard principal covering all AWS accounts.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:AddLayerVersionPermission` | `$.LayerName`, `$.OrganizationId`, `$.Principal`, `$.VersionNumber` |
| `lambda:RemoveLayerVersionPermission` | `$.LayerName`, `$.StatementId`, `$.VersionNumber` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetLayerVersionPolicy` | `$.LayerName`, `$.VersionNumber` |
| `lambda:ListLayerVersions` | `$.LayerName` |

### Mitigation.LAMBDA.Layer.3

An AWS Lambda layer should be accessed only by trusted identities or AWS services acting on behalf of trusted resources

**Goal:** [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)

**Addresses:** [Threat.LAMBDA.Layer.3](#threatlambdalayer3)

**Implemented by:** [Implementation.LAMBDA.Layer.2](#implementationlambdalayer2), [Implementation.LAMBDA.Layer.4](#implementationlambdalayer4), [Implementation.LAMBDA.Layer.5](#implementationlambdalayer5), [Implementation.LAMBDA.Layer.6](#implementationlambdalayer6), [Implementation.LAMBDA.Layer.7](#implementationlambdalayer7), [ManagedControl.LAMBDA.Layer.1](#managedcontrollambdalayer1)

**Scoping** (for [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)): Each version of your layer carries its own resource-based policy that can grant other accounts permission to retrieve the layer archive. Sharing can also extend to every account in an organization, or to all AWS accounts.

**Configure via**

| Operation | Parameters |
|---|---|
| `lambda:AddLayerVersionPermission` | `$.LayerName`, `$.OrganizationId`, `$.Principal`, `$.VersionNumber` |
| `lambda:RemoveLayerVersionPermission` | `$.LayerName`, `$.StatementId`, `$.VersionNumber` |

**Verify via**

| Operation | Parameters |
|---|---|
| `lambda:GetLayerVersionPolicy` | `$.LayerName`, `$.VersionNumber` |
| `lambda:ListLayerVersions` | `$.LayerName` |

---

## Implementations

### Implementation.LAMBDA.1

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |
| `*` | `arn:aws:lambda:*:${Account}:layer:*` | `aws:PrincipalOrgID` |

**Implements:** [Mitigation.LAMBDA.1](#mitigationlambda1)

**Scoping** (for [Mitigation.LAMBDA.1](#mitigationlambda1)): This policy enforces `aws:ResourceOrgID` to match your organization for AWS Lambda service API calls made through your interface VPC endpoint (powered by AWS PrivateLink). Requests from that network to Lambda resources outside your organization are denied, except for the AWS-owned Lambda layer ARNs you explicitly list.

**Data perimeter** (for [Mitigation.LAMBDA.1](#mitigationlambda1)): Some AWS services publish Lambda extensions as layers owned by an AWS service account. That account sits outside your organization, so `aws:ResourceOrgID` does not match requests to those layers. List the service-owned layer ARNs in a separate statement, scoped with `aws:PrincipalOrgID`. This keeps your functions able to consume those layers while the resource perimeter stays in place.
- [Service-specific VPC endpoint policy example for AWS Lambda](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/lambda_endpoint_policy.json)
- [Service-owned resources that AWS services access from your networks](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_owned_resources.md)

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [VPC endpoint policy sample: AWS Lambda data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/lambda_endpoint_policy.json)

### Implementation.LAMBDA.2

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.LAMBDA.2](#mitigationlambda2)

**Scoping** (for [Mitigation.LAMBDA.2](#mitigationlambda2)): This VPC endpoint policy governs AWS Lambda API calls with the `aws:PrincipalOrgID` condition key, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, principals outside your organization are restricted from making AWS Lambda API calls through your network.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to exempt AWS services that use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [VPC endpoint policy sample: AWS Lambda data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/lambda_endpoint_policy.json)

### Implementation.LAMBDA.CapacityProvider.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CapacityProvider`: `$.TelemetryConfig.LoggingConfig.LogGroup`, `$.TelemetryConfig.LoggingConfig.SystemLogLevel`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1)): This implementation validates `$.TelemetryConfig.LoggingConfig.LogGroup` and `$.TelemetryConfig.LoggingConfig.SystemLogLevel` before a capacity provider is created. Requiring a declared log group and a log level that keeps the needed detail helps preserve capacity provider system logs for investigating activity on your managed instances.

### Implementation.LAMBDA.CapacityProvider.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `lambda:CreateCapacityProvider` — `$.CapacityProviderName`, `$.TelemetryConfig.LoggingConfig.LogGroup`, `$.TelemetryConfig.LoggingConfig.SystemLogLevel`
- `lambda:UpdateCapacityProvider` — `$.CapacityProviderName`, `$.TelemetryConfig.LoggingConfig.LogGroup`, `$.TelemetryConfig.LoggingConfig.SystemLogLevel`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1)): This implementation records `UpdateCapacityProvider` and `CreateCapacityProvider` calls that set `$.TelemetryConfig.LoggingConfig.LogGroup`, your log group destination, and `$.TelemetryConfig.LoggingConfig.SystemLogLevel`, your system log level. Recording these calls helps you detect an unintended or unexpected change to your capacity provider logging configuration.

### Implementation.LAMBDA.CapacityProvider.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListCapacityProviders`; check `$.CapacityProviders[*].CapacityProviderArn`, `$.NextMarker`
2. Call `lambda:GetCapacityProvider` with `$.CapacityProviderName`; check `$.CapacityProvider.TelemetryConfig.LoggingConfig.LogGroup`, `$.CapacityProvider.TelemetryConfig.LoggingConfig.SystemLogLevel`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1)): This periodic rule enumerates your capacity providers and reads `$.CapacityProvider.TelemetryConfig.LoggingConfig.LogGroup` and `$.CapacityProvider.TelemetryConfig.LoggingConfig.SystemLogLevel`. It flags providers that do not send system logs to your intended log group and detail level, helping preserve visibility into managed instance activity.

### Implementation.LAMBDA.CapacityProvider.4

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CapacityProvider`: `$.PermissionsConfig.CapacityProviderOperatorRoleArn`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)): This implementation reads `$.Role.AssumeRolePolicyDocument`, the trust policy of the operator role named in `$.PermissionsConfig.CapacityProviderOperatorRoleArn`. Creation of the capacity provider fails when that trust policy grants public access, or grants role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.LAMBDA.CapacityProvider.5

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListCapacityProviders`; check `$.CapacityProviders[*].PermissionsConfig.CapacityProviderOperatorRoleArn`
2. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
3. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
4. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)): This periodic check runs IAM Access Analyzer checks over the trust policy at `$.Role.AssumeRolePolicyDocument` for the operator role recorded at `$.CapacityProviders[*].PermissionsConfig.CapacityProviderOperatorRoleArn`. Flagging a trust policy that can grant public role assumption, or assumption beyond your reference policy, helps you detect over-permissive grants.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.LAMBDA.CapacityProvider.6

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `lambda.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)): This implementation guidance recommends a trust policy on `$.PermissionsConfig.CapacityProviderOperatorRoleArn`, your capacity provider operator role, that restricts assumption to the AWS Lambda service principal `lambda.amazonaws.com`.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.LAMBDA.CapacityProvider.7

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2)): This implementation governs `sts:AssumeRole` through the `aws:PrincipalOrgID` condition key, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those requests with `aws:SourceOrgID`. As a result, principals outside your organization are restricted from assuming the capacity provider operator role at `$.PermissionsConfig.CapacityProviderOperatorRoleArn`.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.LAMBDA.CapacityProvider.8

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CapacityProvider`: `$.VpcConfig.SubnetIds[*]`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3)): This implementation validates each subnet in `$.VpcConfig.SubnetIds[*]` at create time, resolving that subnet's route table for a default route to an internet gateway. Subnets carrying that route fail validation, so your capacity provider instances are not placed in public subnets as determined by that test.

### Implementation.LAMBDA.CapacityProvider.9

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListCapacityProviders`; check `$.CapacityProviders[*].CapacityProviderArn`
2. Call `lambda:GetCapacityProvider` with `$.CapacityProviderName`; check `$.CapacityProvider.VpcConfig.SubnetIds[*]`
3. Call `ec2:DescribeRouteTables` with `$.Filters`; check `$.RouteTables[*].Routes[*].GatewayId`, `$.RouteTables[*].Routes[*].DestinationCidrBlock` = `0.0.0.0/0`, `$.RouteTables[*].Routes[*].DestinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3)): This periodic check reads `$.CapacityProvider.VpcConfig.SubnetIds[*]`, the subnets where your capacity provider launches compute instances, then resolves each subnet's route table. It flags a default route whose gateway target (`$.RouteTables[*].Routes[*].GatewayId`) is an internet gateway, surfacing capacity providers configured with internet-routable subnets.

### Implementation.LAMBDA.CapacityProvider.10

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CapacityProvider`: `$.CapacityProviderName`, `$.InstanceRequirements.AllowedInstanceTypes[*]`, `$.InstanceRequirements.ExcludedInstanceTypes[*]`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4)): This hook validates `$.InstanceRequirements.AllowedInstanceTypes[*]` at create time, so a capacity provider permitting instance types without automatic instance-to-instance encryption is not created. That helps provide an additional layer of encryption in transit between your Managed Instances without application changes.

### Implementation.LAMBDA.CapacityProvider.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListCapacityProviders`; check `$.CapacityProviders[*].CapacityProviderArn`
2. Call `lambda:GetCapacityProvider` with `$.CapacityProviderName`; check `$.CapacityProvider.InstanceRequirements.AllowedInstanceTypes[*]`, `$.CapacityProvider.InstanceRequirements.ExcludedInstanceTypes[*]`
3. Call `ec2:DescribeInstanceTypes` with `$.InstanceType[*]`; check `$.instanceTypeSet[*].instanceType`, `$.instanceTypeSet[*].networkInfo.encryptionInTransitSupported`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4)): This check compares the allowed instance types of your AWS Lambda capacity provider at `$.CapacityProvider.InstanceRequirements.AllowedInstanceTypes[*]` with EC2 encryption-in-transit support at `$.instanceTypeSet[*].networkInfo.encryptionInTransitSupported`. It flags a capacity provider that can launch instance types without automatic encryption of instance-to-instance traffic.

### Implementation.LAMBDA.CapacityProvider.12

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CapacityProvider`: `$.KmsKeyArn`, `$.CapacityProviderName`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5)): This implementation validates `$.KmsKeyArn`, the KMS key ARN that encrypts your capacity provider's resources, in CloudFormation stack operations. Requiring your own key at create time helps address the mitigation by giving you control over the key lifecycle and permissions.

### Implementation.LAMBDA.CapacityProvider.13

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListCapacityProviders`; check `$.CapacityProviders[*].CapacityProviderArn`, `$.CapacityProviders[*].KmsKeyArn`
2. Call `lambda:GetCapacityProvider` with `$.CapacityProviderName`; check `$.CapacityProvider.KmsKeyArn`
3. Call `kms:DescribeKey` with `$.KeyId`; check `$.KeyMetadata.KeyManager` = `CUSTOMER`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5)): This periodic check reads `$.CapacityProvider.KmsKeyArn` for each capacity provider and resolves the referenced key through `$.KeyMetadata.KeyManager`. It flags capacity providers whose managed instance volumes fall back to an AWS managed key, helping you retain control of key policy and rotation.

### Implementation.LAMBDA.CapacityProvider.14

*Guidance — Directive*

Configure your capacity provider's VpcConfig.SubnetIds with two or more subnets that resolve to different Availability Zones. Lambda launches managed instances and attaches elastic network interfaces (ENIs) in those subnets. Spanning at least two zones helps network connectivity for your functions survive the loss of a single zone.

Availability Zone membership is a property of each subnet, not of the capacity provider. Before you supply subnet IDs, call the Amazon EC2 DescribeSubnets operation and confirm that the AvailabilityZone field differs across the subnets you chose. Standardize on infrastructure-as-code review so the templates and modules that create capacity providers supply at least two subnets from different zones. Do this at create time, because UpdateCapacityProvider does not accept VpcConfig. Changing the network placement of a live capacity provider requires replacing it.

To verify the deployed state, call GetCapacityProvider and review the subnet IDs it returns. You can also enumerate the attached network interfaces with the Amazon EC2 DescribeNetworkInterfaces operation. Confirm that their subnets span more than one zone. Note that Lambda hides managed instances and their network interfaces from Amazon EC2 list operations by default. The account-level managed resource visibility setting controls this behavior, and direct queries by resource ID still return hidden managed resources.

To assess and track multi-AZ posture at scale, use AWS Resilience Hub. Add the application resources that depend on your capacity provider to a Resilience Hub application. Run a resiliency assessment, which flags single-AZ concentration against your resiliency policy.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6)): This recommendation directs you to supply `$.VpcConfig.SubnetIds[*]` values drawn from two or more Availability Zones. That placement addresses the mitigation by spreading the capacity provider's network interfaces across zones, which helps connectivity survive a single zone failure.

### Implementation.LAMBDA.CapacityProvider.15

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CapacityProvider`: `$.VpcConfig.SubnetIds[*]`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6)): This implementation validates at create time that `$.VpcConfig.SubnetIds[*]` contains two or more subnet entries, rejecting a single-subnet capacity provider. Multiple subnets let Lambda spread managed instances and their network interfaces across Availability Zones, helping your functions tolerate the loss of one zone.

### Implementation.LAMBDA.CapacityProvider.16

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListCapacityProviders`; check `$.CapacityProviders[*].CapacityProviderArn`, `$.CapacityProviders[*].VpcConfig.SubnetIds[*]`
2. Call `ec2:DescribeSubnets` with `$.SubnetId[*]`; check `$.subnetSet[*].subnetId`, `$.subnetSet[*].availabilityZone`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6)): This periodic check reads the subnets configured on each capacity provider in `$.CapacityProviders[*].VpcConfig.SubnetIds[*]`, then reads each subnet's Availability Zone in `$.subnetSet[*].availabilityZone`. It flags a capacity provider whose subnets resolve to a single Availability Zone, where network interfaces would lack multi-zone redundancy.

### Implementation.LAMBDA.CapacityProvider.17

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CapacityProvider`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7)): This hook checks that `$.Tags[*].Key` and `$.Tags[*].Value` include owner and purpose tags before a capacity provider is created. Requiring those tag keys and values helps you attribute ownership and business function for each capacity provider.

### Implementation.LAMBDA.CapacityProvider.18

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListCapacityProviders`; check `$.CapacityProviders[*].CapacityProviderArn`
2. Call `lambda:ListTags` with `$.Resource`; check `$.Tags{key}`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7)): This implementation reads `$.Tags{key}`, the tag set on your Lambda capacity provider. Flagging a capacity provider that omits your owner and purpose tag keys helps address the mitigation by identifying resources lacking recorded business metadata.

### Implementation.LAMBDA.CapacityProvider.19

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:CreateCapacityProvider` | `*` | `aws:RequestTag/owner` |
| `lambda:CreateCapacityProvider` | `*` | `aws:RequestTag/purpose` |
| `lambda:CreateCapacityProvider` | `*` | `aws:TagKeys` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:capacity-provider:*` | `aws:RequestTag/owner` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:capacity-provider:*` | `aws:RequestTag/purpose` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:capacity-provider:*` | `aws:TagKeys` |
| `lambda:UntagResource` | `arn:aws:lambda:*:*:capacity-provider:*` | `aws:TagKeys` |

**Implements:** [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7)): This implementation restricts capacity provider creation and tagging requests with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys. Requests that omit either required tag key are denied, helping you capture owner and purpose metadata on capacity providers you create or retag. This implementation also restricts `lambda:UntagResource` with the `aws:TagKeys` condition key; requests that remove either required tag key are denied.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.LAMBDA.CapacityProvider.20

*Guidance — Directive*

Associate your capacity provider only with security groups that restrict network access for the Lambda Managed Instances compute instances. Configure those security groups following least-privilege best practices. Restrict both inbound and outbound rules to only the sources and destinations, protocols, and ports that your functions require. Sources and destinations can be IP address ranges, referenced security groups, or prefix lists. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change. Avoid rules that permit unrestricted access from or to any IP address, particularly on administrative or sensitive ports. If you omit `SecurityGroupIds`, Lambda uses the default security group for your VPC. Specify your own purpose-built security groups instead. Because a change to the VPC configuration requires replacing the capacity provider, choose these security groups before you create it.

To govern security groups at scale, use AWS Firewall Manager security group policies across your organization in AWS Organizations. You can scope a policy by account, organizational unit, or tag. A common security group policy applies a baseline security group to in-scope resources. A content audit security group policy audits inbound and outbound rules against an allow-list or a block-list. It flags noncompliant security groups in a central dashboard, sends notifications, and can automatically remediate them. A usage audit security group policy removes unused or redundant security groups.

To verify unintended access permitted by your security groups, use Amazon VPC Network Access Analyzer. Define a Network Access Scope that describes the access you consider unintended. For example, describe any path that reaches administrative ports 22 or 3389 from an unrestricted address. Run an analysis with `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`, then review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)): This guidance recommends scoping each security group in `$.VpcConfig.SecurityGroupIds[*]` to the specific addresses, protocols, and ports your Managed Instances need. Excluding unrestricted ranges such as 0.0.0.0/0 and ::/0 from those inbound and outbound rules reduces the network exposure of your Managed Instances.

### Implementation.LAMBDA.CapacityProvider.21

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CapacityProvider`: `$.VpcConfig.SecurityGroupIds[*]`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)): This hook restricts the security groups in your capacity provider's VPC configuration (`$.VpcConfig.SecurityGroupIds[*]`) by checking their inbound rules (`$.securityGroupInfo[*].ipPermissions`) and outbound rules (`$.securityGroupInfo[*].ipPermissionsEgress`). Requests that associate groups allowing `0.0.0.0/0` or `::/0`, directly or through a prefix list, on ports beyond those you approve are denied.

### Implementation.LAMBDA.CapacityProvider.22

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListCapacityProviders`; check `$.CapacityProviders[*].CapacityProviderArn`
2. Call `lambda:GetCapacityProvider` with `$.CapacityProviderName`; check `$.CapacityProvider.VpcConfig.SecurityGroupIds[*]`
3. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
4. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)

**Scoping** (for [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8)): This check reads `$.CapacityProvider.VpcConfig.SecurityGroupIds[*]`, the security groups attached to your capacity provider, then inspects their inbound and outbound rules. Reporting the addresses, protocols, and port ranges those rules permit helps you detect unrestricted inbound sources or outbound destinations.

### Implementation.LAMBDA.CodeSigningConfig.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CodeSigningConfig`: `$.AllowedPublishers.SigningProfileVersionArns[*]`

**Implements:** [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)

**Scoping** (for [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)): This implementation checks each signing profile version ARN in the allowed publishers list at `$.AllowedPublishers.SigningProfileVersionArns[*]`. Creating or updating a code signing configuration that lists a signing profile version you have not designated as trusted is rejected before provisioning.

**Data perimeter** (for [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)): A code signing configuration can list signing profile versions owned by accounts outside your organization. Those profiles then become trusted publishers for code that your functions deploy. A hook validates `$.AllowedPublishers.SigningProfileVersionArns[*]` when the resource is created or updated. Values that reference signing profile versions outside your organization are rejected before provisioning.
- [AWS::Lambda::CodeSigningConfig resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-codesigningconfig.html)
- [AWS::Lambda::CodeSigningConfig AllowedPublishers property type](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-properties-lambda-codesigningconfig-allowedpublishers.html)
- [SigningProfileVersionArns property of AllowedPublishers](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-properties-lambda-codesigningconfig-allowedpublishers.html#cfn-lambda-codesigningconfig-allowedpublishers-signingprofileversionarns)

### Implementation.LAMBDA.CodeSigningConfig.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `lambda:CreateCodeSigningConfig` — `$.AllowedPublishers.SigningProfileVersionArns[*]`
- `lambda:UpdateCodeSigningConfig` — `$.AllowedPublishers.SigningProfileVersionArns[*]`, `$.CodeSigningConfigArn`

**Implements:** [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)

**Scoping** (for [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)): This implementation records `CreateCodeSigningConfig` and `UpdateCodeSigningConfig` calls with the `$.AllowedPublishers.SigningProfileVersionArns[*]` values they set. Reviewing those values and the calling identity helps you detect a code signing configuration that trusts a publisher outside your organization.

**Data perimeter** (for [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)): Monitor `CreateCodeSigningConfig` and `UpdateCodeSigningConfig` calls in your environment, focusing on the `$.AllowedPublishers.SigningProfileVersionArns[*]` request parameter. Each ARN names a signing profile version whose signatures Lambda accepts for functions that reference the configuration. An ARN owned by an account outside your organization extends your code-signing trust boundary. Where a recorded value is not trusted, remediate with the responsive controls of your choice.
- [AllowedPublishers: SigningProfileVersionArns request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AllowedPublishers.html#lambda-Type-AllowedPublishers-SigningProfileVersionArns)
- [CreateCodeSigningConfig API reference](https://docs.aws.amazon.com/lambda/latest/api/API_CreateCodeSigningConfig.html)
- [UpdateCodeSigningConfig API reference](https://docs.aws.amazon.com/lambda/latest/api/API_UpdateCodeSigningConfig.html)

### Implementation.LAMBDA.CodeSigningConfig.3

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::CodeSigningConfig`: `$.configuration.AllowedPublishers.SigningProfileVersionArns[*]`

**Implements:** [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)

**Scoping** (for [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1)): This implementation reads `$.configuration.AllowedPublishers.SigningProfileVersionArns[*]` for each of your code signing configurations. It flags a configuration that allows an AWS Signer signing profile version you have not identified as trusted to sign function code.

### Implementation.LAMBDA.CodeSigningConfig.4

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::CodeSigningConfig`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2)

**Scoping** (for [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2)): This implementation validates that `$.Tags[*].Key` and `$.Tags[*].Value` declare owner and purpose entries before a code signing configuration is created. Rejecting untagged definitions at deployment time helps address the mitigation by keeping ownership and business function attributable for every configuration.

### Implementation.LAMBDA.CodeSigningConfig.5

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::CodeSigningConfig`: `$.tags[*].key`, `$.tags[*].value`

**Implements:** [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2)

**Scoping** (for [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2)): This implementation reads `$.tags[*].key` for each of your code signing configurations, flagging any that lack the owner and purpose tag keys you require. That helps address the mitigation by making each configuration's accountable team and business function identifiable during review.

### Implementation.LAMBDA.CodeSigningConfig.6

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:CreateCodeSigningConfig` | `*` | `aws:RequestTag/owner` |
| `lambda:CreateCodeSigningConfig` | `*` | `aws:RequestTag/purpose` |
| `lambda:CreateCodeSigningConfig` | `*` | `aws:TagKeys` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:code-signing-config:*` | `aws:RequestTag/owner` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:code-signing-config:*` | `aws:RequestTag/purpose` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:code-signing-config:*` | `aws:TagKeys` |
| `lambda:UntagResource` | `arn:aws:lambda:*:*:code-signing-config:*` | `aws:TagKeys` |

**Implements:** [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2)

**Scoping** (for [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2)): This service control policy (SCP) restricts `lambda:CreateCodeSigningConfig` and, for code signing configurations only, `lambda:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys. Requests that do not satisfy those conditions are denied, so you can track the owner and business function of configurations you create or tag. It also restricts `lambda:UntagResource` on code signing configurations with the `aws:TagKeys` condition key, so requests that remove the owner or purpose tag are denied.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.LAMBDA.EventSourceMapping.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::EventSourceMapping`: `$.LoggingConfig.SystemLogLevel`, `$.FunctionName`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1)): This hook validates that `$.LoggingConfig.SystemLogLevel` is set, so a mapping that would poll without system logging is not created. The configured level has the Lambda event poller send system logs to CloudWatch Logs, giving you error and connection visibility.

### Implementation.LAMBDA.EventSourceMapping.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`, `$.logStreamName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `lambda:CreateEventSourceMapping` — `$.FunctionName`, `$.LoggingConfig.SystemLogLevel`
- `lambda:DeleteEventSourceMapping` — `$.UUID`
- `lambda:UpdateEventSourceMapping` — `$.LoggingConfig.SystemLogLevel`, `$.UUID`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1)): This implementation records `CreateEventSourceMapping` and `UpdateEventSourceMapping` calls that set `$.LoggingConfig.SystemLogLevel`, the system log level for your event source mapping. It also records `DeleteEventSourceMapping`, `DeleteLogGroup`, `DeleteLogStream`, and `PutRetentionPolicy`, helping you detect unexpected changes to event poller logging and its log records.

### Implementation.LAMBDA.EventSourceMapping.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListEventSourceMappings`; check `$.EventSourceMappings[*].UUID`, `$.EventSourceMappings[*].EventSourceMappingArn`, `$.EventSourceMappings[*].EventSourceArn`, `$.EventSourceMappings[*].SelfManagedEventSource.Endpoints{key}[*]`, `$.EventSourceMappings[*].ProvisionedPollerConfig.MinimumPollers`, `$.EventSourceMappings[*].LoggingConfig.SystemLogLevel`
2. Call `lambda:GetEventSourceMapping` with `$.UUID`; check `$.LoggingConfig.SystemLogLevel`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1)): This periodic check reads `$.LoggingConfig.SystemLogLevel` on each event source mapping and flags any mapping with no system log level set. Flagging that gap helps address the mitigation by identifying mappings that are not configured to send system logs to CloudWatch Logs.

### Implementation.LAMBDA.EventSourceMapping.4

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::EventSourceMapping`: `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2), [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2)): This hook requires `VPC_SUBNET` and `VPC_SECURITY_GROUP` values in `$.SourceAccessConfigurations[*].Type`, with your subnet and security group identifiers in `$.SourceAccessConfigurations[*].URI`. Provisioning requests that omit this VPC network configuration are denied, so event poller connections to your brokers can stay within your VPC.

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4)): This hook requires two or more `VPC_SUBNET` entries in `$.SourceAccessConfigurations[*].Type`, each naming one of the subnets your event poller connects through in `$.SourceAccessConfigurations[*].URI`. Create and update requests with fewer entries are denied, so you can spread your event poller network interfaces across subnets in separate Availability Zones.

### Implementation.LAMBDA.EventSourceMapping.5

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListEventSourceMappings`; check `$.EventSourceMappings[*].UUID`
2. Call `lambda:GetEventSourceMapping` with `$.UUID`; check `$.SourceAccessConfigurations[*].Type` = `VPC_SUBNET`, `$.SourceAccessConfigurations[*].Type` = `VPC_SECURITY_GROUP`, `$.SourceAccessConfigurations[*].URI`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2)): This periodic check reads `$.SourceAccessConfigurations[*].Type` and `$.SourceAccessConfigurations[*].URI` on every event source mapping in the account and Region. It flags mappings with no `VPC_SUBNET` or `VPC_SECURITY_GROUP` entry, surfacing event pollers that reach their event source without the network isolation of your VPC.

### Implementation.LAMBDA.EventSourceMapping.6

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::EventSourceMapping`: `$.KmsKeyArn`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3)): This implementation validates at create time that the `$.KmsKeyArn` template property names a customer managed key for your event source mapping's filter criteria encryption. A create that omits that key is denied, so your filter criteria do not fall back to an AWS owned key.

### Implementation.LAMBDA.EventSourceMapping.7

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListEventSourceMappings`; check `$.EventSourceMappings[*].UUID`
2. Call `lambda:GetEventSourceMapping` with `$.UUID`; check `$.KMSKeyArn`
3. Call `kms:DescribeKey` with `$.KeyId`; check `$.KeyMetadata.KeyManager` = `CUSTOMER`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3)): This rule reads `$.KMSKeyArn` on each event source mapping and checks `$.KeyMetadata.KeyManager` to confirm the referenced key is customer managed. Flagging mappings whose filter criteria are not protected by a customer managed key helps you retain control of the key lifecycle and permissions.

### Implementation.LAMBDA.EventSourceMapping.8

*Guidance — Directive*

Configure your event source mapping with two or more source access configuration entries of type VPC_SUBNET that resolve to different Availability Zones. Lambda creates elastic network interfaces in the subnets you configure on the mapping, reusing any that already exist. Supplying subnets from at least two zones places those interfaces in more than one Availability Zone, which helps connectivity to your event source survive the loss of a single zone. These subnet entries apply to self-managed Apache Kafka event sources. For an Amazon MSK, Amazon MQ, or Amazon DocumentDB source, the mapping follows the network placement of the cluster or broker you reference, so apply the same multi-zone subnet selection there.

Availability Zone membership is a property of each subnet, not of the event source mapping. A subnet resides entirely within one Availability Zone and cannot span zones. Before you supply subnet IDs, confirm they belong to distinct zones by calling the Amazon EC2 DescribeSubnets operation and reading the AvailabilityZone response field. Standardize on infrastructure-as-code review so the templates and modules that create your event source mappings supply at least two subnets from different zones.

To verify the deployed state, enumerate the network interfaces with the Amazon EC2 DescribeNetworkInterfaces operation and confirm that their subnets span more than one Availability Zone. You can also call GetEventSourceMapping, which echoes the subnet entries configured on the mapping.

To assess and track multi-AZ posture at scale, use AWS Resilience Hub. Include the application resources for your event source mapping in a Resilience Hub application and run a resiliency assessment. The assessment flags single-zone concentration against your resiliency policy.

Keep in mind that the event source mapping runs with the target function's execution role. That role needs permission to create and describe network interfaces in the subnets you select, so confirm those permissions are in place before you add a second zone.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4)): This recommendation requires the mapping's `VPC_SUBNET` entries in `$.SourceAccessConfigurations[*].URI` to name subnets from two or more Availability Zones. Because Lambda creates the mapping's network interfaces in those subnets, spanning zones helps polling survive the loss of a single zone.

### Implementation.LAMBDA.EventSourceMapping.9

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListEventSourceMappings`; check `$.EventSourceMappings[*].UUID`
2. Call `lambda:GetEventSourceMapping` with `$.UUID`; check `$.SourceAccessConfigurations[*].Type` = `VPC_SUBNET`, `$.SourceAccessConfigurations[*].URI`
3. Call `ec2:DescribeSubnets` with `$.SubnetId`; check `$.subnetSet[*].subnetId`, `$.subnetSet[*].availabilityZone`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4)): This rule flags event source mappings whose `VPC_SUBNET` entries in `$.SourceAccessConfigurations[*].URI` resolve to a single `$.subnetSet[*].availabilityZone`. Detecting single-zone event poller ENI placement helps address the mitigation by revealing where zonal redundancy is missing.

### Implementation.LAMBDA.EventSourceMapping.10

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::EventSourceMapping`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5)): This implementation checks the tags on your event source mapping, requiring `$.Tags[*].Key` entries for owner and purpose with a non-empty `$.Tags[*].Value`. Event source mappings missing this metadata can be blocked at create and update time, helping make ownership and business function identifiable.

### Implementation.LAMBDA.EventSourceMapping.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListEventSourceMappings`; check `$.EventSourceMappings[*].EventSourceMappingArn`
2. Call `lambda:ListTags` with `$.Resource`; check `$.Tags{key}`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5)): This rule enumerates your event source mappings and reads `$.Tags{key}` for each one, flagging any that omit your required owner and purpose keys. Event source mappings do not inherit tags from their target functions, making per-mapping tags the way to attribute ownership and business purpose.

### Implementation.LAMBDA.EventSourceMapping.12

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:CreateEventSourceMapping` | `*` | `aws:RequestTag/owner` |
| `lambda:CreateEventSourceMapping` | `*` | `aws:RequestTag/purpose` |
| `lambda:CreateEventSourceMapping` | `*` | `aws:TagKeys` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:event-source-mapping:*` | `aws:RequestTag/owner` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:event-source-mapping:*` | `aws:RequestTag/purpose` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:event-source-mapping:*` | `aws:TagKeys` |
| `lambda:UntagResource` | `arn:aws:lambda:*:*:event-source-mapping:*` | `aws:TagKeys` |

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5)): This service control policy (SCP) restricts the `lambda:CreateEventSourceMapping` and `lambda:TagResource` actions with the `aws:RequestTag/owner` and `aws:RequestTag/purpose` condition keys. Requests that omit either key are denied, helping you keep owner and purpose metadata on newly created and newly tagged event source mappings. The SCP also restricts `lambda:UntagResource` with the `aws:TagKeys` condition key, so requests that remove either required tag key are denied.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.LAMBDA.EventSourceMapping.13

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::EventSourceMapping`: `$.MetricsConfig.Metrics[*]`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6)): This implementation checks the event source mapping metrics configuration at `$.MetricsConfig.Metrics[*]` for the opt-in `EventCount`, `ErrorCount`, and `KafkaMetrics` groups. Creates and updates that omit these groups do not proceed, so the mapping does not operate with only default metrics.

### Implementation.LAMBDA.EventSourceMapping.14

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `lambda:CreateEventSourceMapping` — `$.FunctionName`, `$.MetricsConfig.Metrics[*]`
- `lambda:UpdateEventSourceMapping` — `$.MetricsConfig.Metrics[*]`, `$.UUID`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6)): Monitoring `UpdateEventSourceMapping` and `CreateEventSourceMapping` records each change to `$.MetricsConfig.Metrics[*]`, the opt-in metric groups on your event source mapping. This helps you detect and respond to attempts to remove detailed monitoring and reduce your visibility into event processing.

### Implementation.LAMBDA.EventSourceMapping.15

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListEventSourceMappings`; check `$.EventSourceMappings[*].UUID`
2. Call `lambda:GetEventSourceMapping` with `$.UUID`; check `$.MetricsConfig.Metrics[*]`, `$.EventSourceArn`, `$.SelfManagedEventSource.Endpoints{key}[*]`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6)): This periodic rule reads `$.MetricsConfig.Metrics[*]` on each event source mapping and flags mappings that omit the applicable `EventCount`, `ErrorCount`, and `KafkaMetrics` groups. Because these groups are opt-in, the check helps keep event, error, and offset-lag telemetry available in CloudWatch for anomaly detection.

### Implementation.LAMBDA.EventSourceMapping.16

*Guidance — Directive*

Configure the security groups associated with your event source mapping following least-privilege best practices. For a private event source, these are the `VPC_SECURITY_GROUP` entries you supply in the mapping's source access configuration. They control the network path Lambda uses to reach your event source, independently of your function's own VPC configuration. Restrict both inbound and outbound rules to only the required sources and destinations, protocols, and ports. Sources and destinations can be IP address ranges, referenced security groups, or prefix lists. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change. Avoid rules that permit unrestricted access from or to any IP address, particularly on administrative or sensitive ports.

To govern security groups at scale, use AWS Firewall Manager security group policies across an AWS Organization. You can scope each policy by account, organizational unit, or tag. A common security group policy applies a baseline group. A content audit policy audits inbound and outbound rules against an allow-list or block-list. It flags non-compliant groups in a central dashboard, notifies you, and can automatically remediate them. A usage audit policy removes unused or redundant groups.

To verify unintended access permitted by your security groups, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended. For example, match any path that reaches administrative ports 22 or 3389 from an unrestricted address. Then run an analysis with `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`, and review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7)): Restrict the inbound and outbound rules on your event source mapping's `VPC_SECURITY_GROUP` security groups to only the addresses, protocols, and ports required. Covering both directions reduces the mapping's network exposure, instead of allowing unrestricted access from or to any IP address.

### Implementation.LAMBDA.EventSourceMapping.17

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::EventSourceMapping`: `$.SourceAccessConfigurations[*].Type`, `$.SourceAccessConfigurations[*].URI`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7)): At create and update, this implementation resolves the VPC security groups your event source mapping declares in `$.SourceAccessConfigurations[*].URI` when `$.SourceAccessConfigurations[*].Type` is `VPC_SECURITY_GROUP`. Groups whose inbound rules (`$.securityGroupInfo[*].ipPermissions`) or outbound rules (`$.securityGroupInfo[*].ipPermissionsEgress`) allow unrestricted traffic fail the check, preventing the stack operation.

### Implementation.LAMBDA.EventSourceMapping.18

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListEventSourceMappings`; check `$.EventSourceMappings[*].UUID`
2. Call `lambda:GetEventSourceMapping` with `$.UUID`; check `$.SourceAccessConfigurations[*].Type` = `VPC_SECURITY_GROUP`, `$.SourceAccessConfigurations[*].URI`
3. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
4. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7)

**Scoping** (for [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7)): This AWS Config rule resolves the security groups named in `$.SourceAccessConfigurations[*].URI` for each event source mapping. It flags a group whose inbound source range `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` or outbound destination range `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` permits traffic from or to any address.

### Implementation.LAMBDA.Function.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.LoggingConfig.LogGroup`, `$.LoggingConfig.LogFormat`, `$.LoggingConfig.ApplicationLogLevel`, `$.LoggingConfig.SystemLogLevel`

**Implements:** [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1)

**Scoping** (for [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1)): This implementation validates `$.LoggingConfig.LogGroup`, `$.LoggingConfig.LogFormat`, `$.LoggingConfig.ApplicationLogLevel`, and `$.LoggingConfig.SystemLogLevel` before the function is created. Blocking templates that leave these values unset helps keep function activity visible in CloudWatch Logs for audit and error detection.

### Implementation.LAMBDA.Function.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`, `$.logStreamName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `lambda:CreateFunction` — `$.FunctionName`, `$.LoggingConfig.ApplicationLogLevel`, `$.LoggingConfig.LogFormat`, `$.LoggingConfig.LogGroup`, `$.LoggingConfig.SystemLogLevel`
- `lambda:UpdateFunctionConfiguration` — `$.FunctionName`, `$.LoggingConfig.ApplicationLogLevel`, `$.LoggingConfig.LogFormat`, `$.LoggingConfig.LogGroup`, `$.LoggingConfig.SystemLogLevel`

**Implements:** [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1)

**Scoping** (for [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1)): This implementation records the `UpdateFunctionConfiguration` and `CreateFunction` calls that set `$.LoggingConfig.LogGroup`, `$.LoggingConfig.LogFormat`, and the application and system log-level filters for your function. Watching `DeleteLogGroup`, `DeleteLogStream`, and `PutRetentionPolicy` helps address the mitigation by surfacing attempts to redirect, suppress, or erase those log records.

### Implementation.LAMBDA.Function.3

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.configuration.functionName`, `$.configuration.functionArn`

**Implements:** [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1)

**Scoping** (for [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1)): This implementation resolves each recorded function from `$.configuration.functionName`, then checks `$.LoggingConfig.LogGroup`, `$.LoggingConfig.LogFormat`, `$.LoggingConfig.ApplicationLogLevel`, and `$.LoggingConfig.SystemLogLevel`. Flagging functions with unset logging controls or an unintended log group helps preserve the audit trail and error detail your investigations need.

### Implementation.LAMBDA.Function.4

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:RequestedRegion`, `aws:SourceVpc` |
| `*` | `*` | `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService`, `lambda:SourceFunctionArn` (alternatives: `aws:PrincipalTag/dp:include:network`) |

**Implements:** [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2)

**Scoping** (for [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2)): This implementation restricts AWS API requests made with your function's execution role credentials, using the `lambda:SourceFunctionArn`, `aws:SourceIp`, and `aws:SourceVpc` condition keys. Requests from outside your expected networks are denied, while exempting AWS services that call through forward access sessions (`aws:ViaAWSService`).

**Data perimeter** (for [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2)): AWS Lambda makes some calls on your behalf from its own network, not from your VPC. These include Amazon CloudWatch Logs, Amazon EFS, and AWS X-Ray calls made with your execution role. Those calls also carry `lambda:SourceFunctionArn`. Exclude `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`, `elasticfilesystem:client*`, and `xray:PutTraceSegments` so this restriction does not deny them. If your execution role assumes another role, apply the same network restriction to that role.
- [Network perimeter SCP example for AWS Lambda execution roles](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/network_perimeter_lambda_scp.json)
- [Network perimeter guidance for service control policies](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/README.md#sidenforcenetworkperimetervpceorgid)

**Reference:** To restrict access to expected networks at scale, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `lambda:SourceFunctionArn` to apply the restriction to the role sessions whose credentials can be used outside your expected networks. Alternatively, you can use `aws:PrincipalTag/tag-key`, tagging the roles with the tag key and value your IAM policy management strategy uses. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)
- [SCP sample: network perimeter policy for AWS Lambda](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/network_perimeter_lambda_scp.json)

### Implementation.LAMBDA.Function.5

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.Role`

**Implements:** [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)

**Scoping** (for [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)): This implementation restricts role assumption through the trust policy at `$.Role.AssumeRolePolicyDocument` for the IAM execution role that `$.Role` names. Create requests are denied when that document grants role assumption to untrusted principals, so such a role is not deployed with your function.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.LAMBDA.Function.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListFunctions`; check `$.Functions[*].FunctionName`, `$.Functions[*].Role`
2. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
3. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
4. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)

**Scoping** (for [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)): This implementation reads each function's execution role from `$.Functions[*].Role`, then evaluates that role's trust policy at `$.Role.AssumeRolePolicyDocument` with IAM Access Analyzer custom policy checks. It flags trust policies that allow public role assumption, or that grant role assumption to principals you designate as untrusted.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.LAMBDA.Function.7

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `lambda.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)

**Scoping** (for [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)): This implementation guidance recommends a trust policy that restricts assumption of `$.Role`, your function's IAM execution role, to the AWS Lambda service principal `lambda.amazonaws.com`.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.LAMBDA.Function.8

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)

**Scoping** (for [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3)): This resource control policy (RCP) restricts `sts:AssumeRole` against the execution role named in the `$.Role` request parameter with the `aws:PrincipalOrgID` condition key. Requests that do not satisfy that condition are denied, so identities outside your organization cannot assume the role and obtain its permissions. The policy exempts AWS services that access the role on your behalf (`aws:PrincipalIsAWSService`), scoping those service requests with the `aws:SourceOrgID` condition key.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.LAMBDA.Function.9

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.VpcConfig.SubnetIds[*]`, `$.VpcConfig.Ipv6AllowedForDualStack`

**Implements:** [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4)

**Scoping** (for [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4)): This implementation validates each subnet ID in `$.VpcConfig.SubnetIds[*]` at create and update time by resolving the referenced subnet's route table. Placement in a subnet with a default route to an internet gateway fails the check, helping reduce your function's network exposure.

### Implementation.LAMBDA.Function.10

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListFunctions`; check `$.Functions[*].FunctionName`, `$.Functions[*].VpcConfig.SubnetIds[*]`, `$.Functions[*].VpcConfig.VpcId`
2. Call `ec2:DescribeRouteTables` with `$.Filters`; check `$.RouteTables[*].Routes[*].GatewayId`, `$.RouteTables[*].Routes[*].DestinationCidrBlock` = `0.0.0.0/0`, `$.RouteTables[*].Routes[*].DestinationIpv6CidrBlock` = `::/0`

**Implements:** [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4)

**Scoping** (for [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4)): This periodic rule reads your function's subnet attachment at `$.Functions[*].VpcConfig.SubnetIds[*]`, then resolves each subnet's routes at `$.RouteTables[*].Routes[*].GatewayId`. It flags a function whose subnet sends default traffic to an internet gateway, so you can detect placement in a public subnet.

### Implementation.LAMBDA.Function.11

*Resource policy — Preventive*

Attaches to `AWS::Lambda::Function` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:*` | `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Alias}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Version}` | `aws:PrincipalIsAWSService`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:ViaAWSService`, `aws:VpceOrgID` (alternatives: `aws:SourceVpc`, `aws:SourceVpce`, `aws:VpceAccount`, `aws:VpceOrgPaths`) |

**Implements:** [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5)

**Scoping** (for [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5)): You can attach a resource-based policy to your Lambda function, which the service evaluates on API calls to invoke, read, or modify the function. This makes the resource-based policy a natural enforcement point for restricting access to your expected network locations.

**Reference:** Confirm that resource-based policies grant access only to expected networks. As a defense-in-depth measure, use the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys where supported. These scope access to VPC endpoints owned by accounts within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys are preferred for enforcing network perimeter controls across multiple accounts because they do not require you to enumerate individual VPCs or endpoints. For finer-grained control, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys to restrict access to a specific VPC or VPC endpoint. Use them also for services that do not currently support the `aws:VpceOrgID`, `aws:VpceOrgPaths`, or `aws:VpceAccount` condition keys. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalIsAWSService` to exempt AWS services that access your resources with their service principals. Use `aws:PrincipalTag/tag-key` to exempt principals that must legitimately operate from outside your expected networks. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)

### Implementation.LAMBDA.Function.12

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::ResourcePolicy`: `$.PolicyDocument`, `$.ResourceArn`

**Implements:** [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5)

**Scoping** (for [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5)): This hook checks the function's resource-based policy document in `$.PolicyDocument` for the `aws:SourceVpc`, `aws:VpceOrgID`, and `aws:SourceIp` condition keys. CloudFormation create and update operations that omit these network restrictions fail, helping you keep access to your function within expected networks.

### Implementation.LAMBDA.Function.13

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:AddLayerVersionPermission`, `lambda:AddPermission` | `*` | `aws:PrincipalTag/dp:exclude:identity` |

**Implements:** [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

**Scoping** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): This implementation governs `lambda:AddPermission` and `lambda:AddLayerVersionPermission` with the `aws:PrincipalTag/dp:exclude:identity` condition key. As a result, principals without that tag are restricted from adding resource-based policy statements that grant access to your functions and layer versions.

**Scoping** (for [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5)): This implementation restricts `lambda:AddPermission` and `lambda:AddLayerVersionPermission` on your functions and layer versions, while exempting principals you tag for exclusion (`aws:PrincipalTag/dp:exclude:identity`). Requests from other principals are denied, so those resources are less likely to be accessed from unexpected networks.

**Data perimeter:** AWS Lambda is not among the services supported by resource control policies (RCPs). The conditions that keep your functions and layer versions accessible only from expected networks therefore live in their own resource-based policies. Restricting `lambda:AddPermission` and `lambda:AddLayerVersionPermission` helps keep that network perimeter intact. Principals you tag for exclusion remain able to make these changes.
- [AWS Lambda API Reference: AddPermission](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html)
- [AWS Lambda API Reference: AddLayerVersionPermission](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html)
- [Example SCP: restrict resource-based policy configurations](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_resource_policy_configurations_scp.json)

**Data perimeter:** AWS Lambda resource-based policies are the surface that grants other identities access to your functions and layer versions. Lambda is not among the services supported by resource control policies (RCPs). The identity perimeter on these resources therefore depends on their resource-based policies. Restricting `lambda:AddPermission` and `lambda:AddLayerVersionPermission` to select principals helps keep those policies from granting access to identities outside your organization.
- [AWS Lambda API Reference: AddPermission](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html)
- [AWS Lambda API Reference: AddLayerVersionPermission](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html)
- [Data perimeter policy example: restrict resource-based policy configurations (SCP)](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_resource_policy_configurations_scp.json)

### Implementation.LAMBDA.Function.14

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6)

**Scoping** (for [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6)): This resource-perimeter implementation governs AWS Lambda API calls with the `aws:ResourceOrgID` condition key. As a result, your identities are restricted from invoking or managing Lambda functions outside your organization, except through unauthenticated function URLs.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.LAMBDA.Function.15

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.VpcConfig.SubnetIds[*]`, `$.VpcConfig.SecurityGroupIds[*]`

**Implements:** [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)

**Scoping** (for [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)): This implementation validates at create time that `$.VpcConfig.SubnetIds[*]` and `$.VpcConfig.SecurityGroupIds[*]` declare the subnets and security groups for VPC attachment. Templates that omit that VPC configuration are denied, so a new function's outbound traffic traverses your VPC subnets under your security groups.

**Data perimeter** (for [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)): A hook can inspect templates before deployment and require VPC attachment on each `AWS::Lambda::Function` resource. VPC configuration is optional for a function, so a template can omit `$.VpcConfig.SubnetIds[*]` and `$.VpcConfig.SecurityGroupIds[*]` entirely. Requiring both at validation time routes a new function's outbound traffic through your VPC subnets, under the security groups you choose.
- [AWS::Lambda::Function CloudFormation resource reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-function.html)
- [VpcConfig property of AWS::Lambda::Function](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-function.html#cfn-lambda-function-vpcconfig)

### Implementation.LAMBDA.Function.16

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `lambda:CreateFunction` — `$.VpcConfig.SecurityGroupIds[*]`, `$.VpcConfig.SubnetIds[*]`
- `lambda:UpdateFunctionConfiguration` — `$.VpcConfig.SecurityGroupIds[*]`, `$.VpcConfig.SubnetIds[*]`

**Implements:** [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)

**Scoping** (for [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)): This implementation records `CreateFunction` and `UpdateFunctionConfiguration` calls together with the `$.VpcConfig.SubnetIds[*]` and `$.VpcConfig.SecurityGroupIds[*]` values each request carries. Surfacing the subnets and security groups requested for a function helps you detect functions created or reconfigured without VPC attachment.

**Data perimeter:** Consider monitoring `CreateFunction` and `UpdateFunctionConfiguration` calls in your environment, focusing on the `VpcConfig` request parameter. This parameter carries the subnets and security groups that attach a function to your VPC. Lambda derives the VPC from those subnet and security group values rather than from a VPC ID. An empty subnet and security group list detaches a function from its VPC. Reviewing the recorded identity behind each call helps you tell routine deployments from unexpected changes. Where a call warrants a response, remediate with the responsive controls of your choice.
- [CreateFunction API reference](https://docs.aws.amazon.com/lambda/latest/api/API_CreateFunction.html)
- [VpcConfig data type reference](https://docs.aws.amazon.com/lambda/latest/api/API_VpcConfig.html)
- [UpdateFunctionConfiguration API reference](https://docs.aws.amazon.com/lambda/latest/api/API_UpdateFunctionConfiguration.html)
- [UpdateFunctionConfiguration VpcConfig request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_UpdateFunctionConfiguration.html#lambda-UpdateFunctionConfiguration-request-VpcConfig)

### Implementation.LAMBDA.Function.17

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.configuration.vpcConfig.subnetIds[*]`, `$.configuration.vpcConfig.securityGroupIds[*]`

**Implements:** [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)

**Scoping** (for [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)): This implementation flags functions whose recorded `$.configuration.vpcConfig.subnetIds[*]` and `$.configuration.vpcConfig.securityGroupIds[*]` are empty. Attaching your function to a VPC helps address the mitigation by placing its network reachability under your own subnet and security group controls.

### Implementation.LAMBDA.Function.18

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:CreateFunction`, `lambda:UpdateFunctionConfiguration` | `*` | `aws:PrincipalTag/dp:exclude`, `lambda:VpcIds` |

**Implements:** [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)

**Scoping** (for [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)): This implementation guidance recommends applying the `lambda:VpcIds` condition key in a service control policy (SCP) to govern `lambda:CreateFunction` and `lambda:UpdateFunctionConfiguration`. These conditions help prevent principals in your organization from deploying functions without a VPC attachment, while exempting principals you tag for exclusion (`aws:PrincipalTag/dp:exclude`).

**Data perimeter** (for [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)): Restricting function deployment to a VPC you manage is a data perimeter control for AWS Lambda. Functions without a VPC attachment reach the internet over a network path your VPC does not control. Requiring the `lambda:VpcIds` condition key on function create and update calls brings that traffic into your VPC. You can then inspect it by using VPC endpoint policies and network perimeter controls.
- [Data perimeter example: service control policy restricting non-VPC deployment](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_nonvpc_deployment_scp.json)

### Implementation.LAMBDA.Function.19

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.KmsKeyArn`, `$.Code.SourceKMSKeyArn`, `$.DurableConfig.KMSKeyArn`

**Implements:** [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)

**Scoping** (for [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)): This hook restricts your function's KMS key properties (`$.KmsKeyArn`, `$.Code.SourceKMSKeyArn`, `$.DurableConfig.KMSKeyArn`) to keys that `DescribeKey` identifies as customer managed. Create and update requests that do not satisfy that condition are denied, which helps keep the key policy and lifecycle under your control.

### Implementation.LAMBDA.Function.20

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.configuration.kmskeyArn`, `$.configuration.functionName`

**Implements:** [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)

**Scoping** (for [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8)): This rule resolves `$.KeyMetadata.KeyManager` for the key recorded at `$.configuration.kmskeyArn`, the key that encrypts your function's environment variables and SnapStart snapshots. A function is flagged as noncompliant when that key is not customer managed, identifying functions whose encryption key you do not manage.

### Implementation.LAMBDA.Function.21

*Guidance — Directive*

Configure your Lambda function's VPC subnet list with two or more subnets that resolve to different Availability Zones. This recommendation applies to functions you attach to a VPC in your account. A function with no VPC configuration is already available across Availability Zones in the Region.

The Hyperplane elastic network interfaces (ENIs) that Lambda creates for your function are tied to the subnet and security group combination you supply. Supplying subnets from more than one Availability Zone places those ENIs in more than one zone. This helps your function's VPC network connectivity survive the loss of a single zone.

Subnet-to-Availability-Zone membership is a property of each subnet, not of the function. A subnet resides entirely within one Availability Zone and cannot span zones. Before you supply subnet IDs, confirm that they belong to distinct Availability Zones. The Amazon EC2 `DescribeSubnets` operation returns the `AvailabilityZone` field for each subnet. Standardize on infrastructure-as-code review so the templates and modules that create your functions always supply at least two subnets from different Availability Zones.

To verify the deployed state, check the subnet placement your function reports through `GetFunctionConfiguration` under `VpcConfig.SubnetIds`. You can also enumerate network interfaces with the Amazon EC2 `DescribeNetworkInterfaces` operation. Lambda manages the Hyperplane ENIs and they are not directly visible to you, so the function's own configuration is the more direct check. Confirm that the subnets span more than one Availability Zone.

To assess and track multi-AZ posture at scale, use AWS Resilience Hub. Include your function among an application's resources in Resilience Hub and run a resiliency assessment. The assessment flags single-Availability-Zone concentration against your resiliency policy.

- [Subnets for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html)
- [DescribeSubnets (Amazon EC2 API Reference)](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeSubnets.html)
- [What is AWS Resilience Hub?](https://docs.aws.amazon.com/resilience-hub/latest/userguide/what-is.html)

**Implements:** [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9)

**Scoping** (for [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9)): This recommendation has you supply two or more subnets from different Availability Zones in your function's VPC subnet configuration. Spreading the Hyperplane ENIs Lambda creates for your function across zones helps VPC connectivity survive the loss of a single Availability Zone.

### Implementation.LAMBDA.Function.22

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.VpcConfig.SubnetIds[*]`

**Implements:** [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9)

**Scoping** (for [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9)): This hook validates at create and update time that `$.VpcConfig.SubnetIds[*]` lists two or more subnet entries. Requiring more than one subnet entry is the precondition for placing your function's network interfaces in more than one Availability Zone.

### Implementation.LAMBDA.Function.23

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.configuration.vpcConfig.subnetIds[*]`

**Implements:** [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9)

**Scoping** (for [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9)): This implementation reads `$.configuration.vpcConfig.subnetIds[*]` on your function, then resolves each subnet's `$.subnetSet[*].availabilityZone`. It flags a VPC-connected function confined to one Availability Zone, addressing the mitigation by surfacing single-zone Hyperplane ENI placement.

### Implementation.LAMBDA.Function.24

*Resource policy — Preventive*

Attaches to `AWS::Lambda::Function` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:GetFunction`, `lambda:GetFunctionConfiguration`, `lambda:InvokeFunction`, `lambda:InvokeFunctionUrl` | `arn:aws:lambda:${Region}:${Account}:function:${FunctionName}`, `arn:aws:lambda:${Region}:${Account}:function:${FunctionName}:${Alias}`, `arn:aws:lambda:${Region}:${Account}:function:${FunctionName}:${Version}` | `aws:PrincipalAccount` |

**Implements:** [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10)

**Scoping** (for [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10)): This implementation restricts invoke and read access to your function with the `aws:PrincipalAccount` condition key. Requests from IAM principals outside the defined subset are denied, which helps protect your function from unintended access.

**Reference:** Confirm that resource-based policies grant access only to authorized identities. As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.LAMBDA.Function.25

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Permission`: `$.Principal`, `$.PrincipalOrgID`, `$.SourceAccount`, `$.SourceArn`, `$.Action`, `$.FunctionUrlAuthType`

**Implements:** [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10)

**Scoping** (for [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10)): This hook restricts create and update requests for your Lambda permissions that grant an action (`$.Action`) to a wildcard principal (`$.Principal`). These requests are denied unless the permission also sets an organization ID (`$.PrincipalOrgID`), a source account (`$.SourceAccount`), or a source ARN (`$.SourceArn`).

**Reference:** This hook calls the AWS IAM Access Analyzer `CheckNoPublicAccess` custom policy check with your permission statement as `$.policyDocument`. The check uses automated reasoning to determine whether the statement might grant public access. Requests to create or update the permission are denied unless the check returns a `$.result` of `PASS`.
- [AWS IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.LAMBDA.Function.26

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::ResourcePolicy`: `$.PolicyDocument`, `$.ResourceArn`

**Implements:** [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10)

**Scoping** (for [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10)): This implementation evaluates the `$.PolicyDocument` declared for your Lambda function and rejects a policy that can grant public access. Requiring explicit principals instead of a wildcard helps prevent your function from inadvertently becoming invocable by unintended principals.

**Reference:** This implementation recommends using Access Analyzer custom policy checks. Custom policy checks (CheckNoPublicAccess) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.LAMBDA.Function.27

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.supplementaryConfiguration.Policy`

**Implements:** [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10)

**Scoping** (for [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10)): This rule reads your function's resource-based policy from `$.supplementaryConfiguration.Policy` and marks the function noncompliant when the public-access check returns a `$.result` other than `PASS`. That verdict surfaces policies that may allow public access, helping you replace broad grants with explicit principal definitions.

**Reference:** This implementation evaluates the policy document with the IAM Access Analyzer custom policy check `CheckNoPublicAccess`. The check applies automated reasoning to the policy you pass to it. A `PASS` result means the policy does not allow public access for the resource type. A `FAIL` result means the policy might allow public access. The check also reports the policy statements behind that result, so you can review them.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.LAMBDA.Function.28

*Resource policy — Preventive*

Attaches to `AWS::Lambda::Function` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:*` | `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Alias}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Version}` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)

**Scoping** (for [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)): This implementation restricts AWS Lambda API calls against your function with the `aws:PrincipalOrgID` condition key, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). Requests that do not satisfy that condition are denied, so unauthenticated callers cannot invoke your function or read its configuration.

**Reference:** Confirm that resource-based policies grant access only to trusted identities. As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. For anonymous requests, `aws:PrincipalOrgID` and `aws:PrincipalOrgPaths` are not included in the request context and `aws:PrincipalAccount` is set to `anonymous`. Requiring these keys to match your organization or account values therefore helps prevent unauthenticated access. For requests by AWS service principals, these keys do not contain your organization or account values, so apply them only when `aws:PrincipalIsAWSService` is `false`. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.LAMBDA.Function.29

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Permission`: `$.Action`, `$.Principal`, `$.FunctionUrlAuthType`, `$.InvokedViaFunctionUrl`, `$.FunctionName`

**Implements:** [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)

**Scoping** (for [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)): This implementation validates the granted action (`$.Action`), the grantee (`$.Principal`), and the function URL auth type condition (`$.FunctionUrlAuthType`) that a permission statement declares. Stack operations that grant `lambda:InvokeFunctionUrl` publicly under the `NONE` auth type are denied, so unauthenticated callers are restricted from invoking your function.

### Implementation.LAMBDA.Function.30

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::ResourcePolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)

**Scoping** (for [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)): This implementation evaluates the `$.PolicyDocument` you declare for your function's resource-based policy and identifies documents that can grant public access. Stack operations declaring such a document are denied, so unauthenticated callers are restricted from invoking your function.

**Reference:** This implementation evaluates the declared policy document with the IAM Access Analyzer custom policy check `CheckNoPublicAccess`. This check uses automated reasoning to evaluate whether a resource-based policy can grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.LAMBDA.Function.31

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Url`: `$.AuthType`

**Implements:** [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)

**Scoping** (for [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)): This implementation checks the function URL auth type declared in `$.AuthType`, requiring `AWS_IAM` rather than `NONE` when the URL is created or updated. Rejecting a URL that declares `NONE` helps address the mitigation by preventing deployment of an endpoint that skips IAM authentication.

### Implementation.LAMBDA.Function.32

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListFunctions`; check `$.Functions[*].FunctionName`
2. Call `lambda:ListFunctionUrlConfigs` with `$.FunctionName`; check `$.FunctionUrlConfigs[*].AuthType` = `AWS_IAM`
3. Call `lambda:GetPolicy` with `$.FunctionName`; check `$.Policy`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::Lambda::Function`; check `$.result` = `PASS`

**Implements:** [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)

**Scoping** (for [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11)): This rule checks the function URL auth type recorded at `$.FunctionUrlConfigs[*].AuthType`, flagging any URL that reports `NONE` rather than `AWS_IAM`. It also submits your function's resource-based policy to a public-access check, where `$.result` reports whether that policy can grant unauthenticated invocation.

**Reference:** This Config rule evaluates your function's resource-based policy by calling the IAM Access Analyzer custom policy check `CheckNoPublicAccess`. That check uses automated reasoning to report whether the policy can grant public access to your function.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.LAMBDA.Function.33

*Guidance — Directive*

Consider using AWS Identity and Access Management Access Analyzer external access analyzers to review your functions and layer versions. These analyzers help identify resources in your accounts that are shared with identities outside your zone of trust. An external access analyzer evaluates the resource-based policies attached to your functions and layer versions. It generates a finding for each policy that grants access to a principal outside your zone of trust. Each finding names the resource, the identity that was granted access, and the permissions granted. Review each finding to determine whether the access is intended. If necessary, remediate with the responsive controls of your choice. External access findings cover only resources in the Region where the analyzer is enabled. Create an analyzer in each Region where you run functions or publish layer versions. External access is not reported for resource-based policies attached to aliases or to specific versions invoked through a qualified ARN.

- [Using AWS Identity and Access Management Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)

**Implements:** [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

**Scoping** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): This recommendation directs you to review the resource-based policies on your functions for access granted to identities outside your organization. Remediating any unintended grants helps you keep access to your functions limited to trusted identities.

### Implementation.LAMBDA.Function.34

*Resource policy — Preventive*

Attaches to `AWS::Lambda::Function` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:*` | `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Alias}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Version}` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `lambda:*` | `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Alias}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Version}` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

**Scoping** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): This implementation restricts AWS Lambda API calls against your function with the `aws:PrincipalOrgID` and `aws:SourceOrgID` condition keys, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). Requests that do not satisfy those conditions are denied, so identities outside your organization cannot invoke or modify the function.

**Reference:** Confirm that resource-based policies grant access only to trusted identities and to services acting on your behalf.

As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`.

To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys.

Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.LAMBDA.Function.35

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Permission`: `$.FunctionName`, `$.Principal`, `$.PrincipalOrgID`, `$.SourceAccount`, `$.SourceArn`

**Implements:** [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

**Scoping** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): This implementation checks `$.Principal`, `$.PrincipalOrgID`, `$.SourceAccount`, and `$.SourceArn` on the function resource-based policy statement a template declares. A statement naming an account, organization, or source that you do not permit is rejected before the permission is created.

**Data perimeter** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): A hook on this resource type can reject a permission statement before it is added to your function's policy. Validate `$.Principal` so that a template cannot grant access to identities outside your organization. Validate `$.PrincipalOrgID` so that an organization-wide grant names only your own organization.
- [AWS::Lambda::Permission resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-permission.html)
- [AWS::Lambda::Permission Principal property](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-permission.html#cfn-lambda-permission-principal)
- [AWS::Lambda::Permission PrincipalOrgID property](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-permission.html#cfn-lambda-permission-principalorgid)

**Reference:** This implementation recommends using both Access Analyzer basic policy checks and custom policy checks. Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning, but do provide actionable recommendations that help you author policies that are functional and conform to security best practices. Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions. The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource; the check fails when the resource policy grants access beyond that allowlist. Example reference policies for resource policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.LAMBDA.Function.36

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::ResourcePolicy`: `$.PolicyDocument`, `$.ResourceArn`

**Implements:** [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

**Scoping** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): This implementation validates `$.PolicyDocument`, the resource-based policy on your function, before CloudFormation attaches it. Rejecting grants that reach beyond the principals you allowlist helps limit function access to trusted identities.

**Data perimeter** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): A CloudFormation template can grant access to your function through principal and organization-identifier properties on Lambda permission resources. Values naming identities outside your organization may grant unintended cross-account access. Validating `$.PolicyDocument` at deployment time helps you review those grants before the policy is attached.
- [AWS::Lambda::Permission resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-permission.html)
- [Principal property of AWS::Lambda::Permission](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-permission.html#cfn-lambda-permission-principal)
- [PrincipalOrgID property of AWS::Lambda::Permission](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-permission.html#cfn-lambda-permission-principalorgid)
- [AWS::Lambda::LayerVersionPermission resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-layerversionpermission.html)
- [Principal property of AWS::Lambda::LayerVersionPermission](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-layerversionpermission.html#cfn-lambda-layerversionpermission-principal)
- [OrganizationId property of AWS::Lambda::LayerVersionPermission](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-layerversionpermission.html#cfn-lambda-layerversionpermission-organizationid)

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks validate your policy against IAM policy grammar and AWS best practices by calling `ValidatePolicy`. These checks do not use automated reasoning. They do provide actionable recommendations that help you author functional policies that conform to security best practices.

Custom policy checks use automated reasoning to provide a higher level of assurance. Calling `CheckNoPublicAccess` verifies that a policy does not grant public access. Calling `CheckNoNewAccess` verifies that it does not grant access beyond a reference policy you maintain. Calling `CheckAccessNotGranted` verifies that it does not grant permissions you specify.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to your function. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [Security best practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.LAMBDA.Function.37

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `lambda:AddLayerVersionPermission` — `$.OrganizationId`, `$.Principal`
- `lambda:AddPermission` — `$.Principal`, `$.PrincipalOrgID`, `$.SourceAccount`, `$.SourceArn`

**Implements:** [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

**Scoping** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): This implementation captures each `AddPermission` and `AddLayerVersionPermission` call in CloudTrail along with the requested `$.Principal`, `$.PrincipalOrgID`, `$.SourceArn`, `$.SourceAccount`, and `$.OrganizationId` values. Reviewing those grants helps you detect resource-based policy statements that extend function or layer access beyond trusted principals.

**Data perimeter:** Both `AddPermission` and `AddLayerVersionPermission` write statements into a resource-based policy that grants other identities access to your function or layer version. Monitoring these calls surfaces the principal and organization scoping that each request carries. Each event also records the identity that made the call. If necessary, remediate with the responsive controls of your choice.
- [AWS Lambda AddPermission API reference](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html)
- [AddPermission: Principal request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html#lambda-AddPermission-request-Principal)
- [AddPermission: PrincipalOrgID request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html#lambda-AddPermission-request-PrincipalOrgID)
- [AddPermission: SourceArn request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html#lambda-AddPermission-request-SourceArn)
- [AddPermission: SourceAccount request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html#lambda-AddPermission-request-SourceAccount)
- [AWS Lambda AddLayerVersionPermission API reference](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html)
- [AddLayerVersionPermission: Principal request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html#lambda-AddLayerVersionPermission-request-Principal)
- [AddLayerVersionPermission: OrganizationId request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html#lambda-AddLayerVersionPermission-request-OrganizationId)

### Implementation.LAMBDA.Function.38

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.supplementaryConfiguration.Policy`, `$.configuration.functionArn`

**Implements:** [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

**Scoping** (for [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)): This implementation inspects the function's resource-based policy recorded at `$.supplementaryConfiguration.Policy` for public access grants and for statements missing the `aws:SourceArn` or `aws:SourceAccount` condition keys. Flagging those policies helps you detect invoke access that is not limited to trusted identities.

**Reference:** This implementation recommends using both Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (ValidatePolicy) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning, but do provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (CheckNoPublicAccess, CheckNoNewAccess, CheckAccessNotGranted) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for CheckNoNewAccess allowlists the principals that may be granted access to the resource; the check fails when the resource policy grants access beyond that allowlist. Example reference policies for resource policies are available in the IAM Access Analyzer custom policy checks samples repository.

External access findings, generated by an external access analyzer, also use automated reasoning to identify resource policies that allow access from outside your zone of trust.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.LAMBDA.Function.39

*Resource policy — Preventive*

Attaches to `AWS::Lambda::Function` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:InvokeFunction`, `lambda:InvokeFunctionUrl` | `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Alias}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Version}` | `aws:SourceAccount`, `aws:SourceArn` |

**Implements:** [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)

**Scoping** (for [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)): This implementation governs `lambda:InvokeFunction` and `lambda:InvokeFunctionUrl` on your function with the `aws:SourceArn`, `aws:SourceAccount`, or `aws:SourceOrgID` condition keys. As a result, AWS services acting on your behalf are restricted from invoking your function on behalf of resources outside your organization.

**Reference:** To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.LAMBDA.Function.40

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Permission`: `$.Principal`, `$.SourceArn`, `$.SourceAccount`, `$.FunctionName`

**Implements:** [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)

**Scoping** (for [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)): This implementation checks a function permission at create time, requiring `$.SourceArn` or `$.SourceAccount` when `$.Principal` names an AWS service. Without one of those source values, other accounts could configure their own resources to invoke your function.

### Implementation.LAMBDA.Function.41

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::ResourcePolicy`: `$.PolicyDocument`, `$.ResourceArn`

**Implements:** [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)

**Scoping** (for [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)): This implementation validates `$.PolicyDocument`, the resource-based policy on your Lambda resource, at create time. It prevents grants to an AWS service principal that omit `aws:SourceArn` or `aws:SourceAccount`, scoping cross-service access to the resources and accounts you name.

**Reference:** This implementation uses IAM Access Analyzer basic policy checks. These checks validate a policy against IAM policy grammar and AWS best practices. They do not use automated reasoning. The hook calls `ValidatePolicy` with the policy document from your template. It then inspects the findings for a security warning that a statement grants an AWS service principal access without a source condition. Adding `aws:SourceArn` or `aws:SourceAccount` resolves that warning.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.LAMBDA.Function.42

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.supplementaryConfiguration.Policy`

**Implements:** [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)

**Scoping** (for [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)): This rule submits the function's recorded resource policy at `$.supplementaryConfiguration.Policy` to a policy validation check for missing source conditions. Flagging grants that omit `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` helps you find functions a service could be induced to invoke for an untrusted resource.

**Reference:** This implementation recommends using Access Analyzer basic policy checks.

Basic policy checks (ValidatePolicy) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning, but do provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.LAMBDA.Function.43

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `lambda:AddPermission` — `$.Action`, `$.FunctionName`, `$.Principal`, `$.PrincipalOrgID`
- `lambda:GetFunction` — `$.FunctionName`
- `lambda:PutResourcePolicy` — `$.Policy`, `$.ResourceArn`
- `lambda:RemovePermission` — `$.FunctionName`, `$.StatementId`

**Implements:** [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14)

**Scoping** (for [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14)): CloudTrail records `AddPermission`, `PutResourcePolicy`, and `RemovePermission` calls that rewrite your function's resource-based policy. Flagging changes to `$.Action` and `$.Policy` helps you detect attempts to broaden which principals can request the deployment package presigned URL.

**Data perimeter** (for [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14)): CloudTrail records each `GetFunction` call, which returns the Amazon S3 presigned URL for downloading a function's deployment package. Recording `$.FunctionName` with the caller's identity and network origin helps you review whether only selected principals requested that package. If a call is unexpected, you can follow up with the responsive controls of your choice.
- [AWS Lambda API Reference: GetFunction](https://docs.aws.amazon.com/lambda/latest/api/API_GetFunction.html)

### Implementation.LAMBDA.Function.44

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:GetFunction` | `*` | `aws:PrincipalTag/${TagKey}` (alternatives: `aws:PrincipalArn`) |

**Implements:** [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14)

**Scoping** (for [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14)): This implementation restricts `lambda:GetFunction` with the `aws:PrincipalTag/<key>` condition key, limiting the action to select principals. Requests from other principals are denied, so they cannot obtain the Amazon S3 presigned URL used to download the function's deployment package.

**Data perimeter** (for [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14)): The download link that `GetFunction` returns is an Amazon S3 presigned URL that AWS Lambda signs with its own service principal. Conditions that evaluate the calling principal's request context do not apply once that URL is issued. Restricting `lambda:GetFunction` to select principals is therefore the preventive lever available to you. Principals that need only the function's settings can call `GetFunctionConfiguration` instead.
- [AWS Lambda GetFunction API reference](https://docs.aws.amazon.com/lambda/latest/api/API_GetFunction.html)
- [Data perimeter example SCP: restrict service-principal presigned URLs](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_presignedURL_scp.json)

**Reference:** To restrict access to select principals, use the `aws:PrincipalArn` or `aws:PrincipalTag/tag-key` condition keys. Use `aws:PrincipalArn` to specify the Amazon Resource Names (ARNs) of the principals. Those ARNs can follow a naming convention or an IAM path that scopes the restriction to a set of principals without enumerating each one. Use `aws:PrincipalTag/tag-key` to identify those principals by a tag attached to them. Choose the condition key that matches your IAM policy management strategy.

### Implementation.LAMBDA.Function.45

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15)

**Scoping** (for [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15)): This implementation validates owner and purpose metadata in `$.Tags[*].Key` and `$.Tags[*].Value` before the function is created. Functions missing those tags are not deployed, helping keep each function traceable to an accountable owner and business purpose.

### Implementation.LAMBDA.Function.46

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.tags[*].key`, `$.tags[*].value`

**Implements:** [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15)

**Scoping** (for [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15)): This rule flags a Lambda function when the tag keys recorded at `$.tags[*].key` omit the owner and purpose keys you require. Attributing each function to an accountable owner and a business purpose helps you triage findings and route them to the right team.

### Implementation.LAMBDA.Function.47

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:CreateFunction` | `*` | `aws:RequestTag/owner` |
| `lambda:CreateFunction` | `*` | `aws:RequestTag/purpose` |
| `lambda:CreateFunction` | `*` | `aws:TagKeys` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:function:*` | `aws:RequestTag/owner` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:function:*` | `aws:RequestTag/purpose` |
| `lambda:TagResource` | `arn:aws:lambda:*:*:function:*` | `aws:TagKeys` |
| `lambda:UntagResource` | `arn:aws:lambda:*:*:function:*` | `aws:TagKeys` |

**Implements:** [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15)

**Scoping** (for [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15)): This implementation restricts `lambda:CreateFunction` and `lambda:TagResource` on your functions with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys. Requests that do not satisfy these conditions are denied, which helps each function you create or tag receive owner and purpose tags. This implementation also restricts `lambda:UntagResource` with the `aws:TagKeys` condition key, so requests that remove the owner or purpose tag key are denied.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### Implementation.LAMBDA.Function.48

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.Layers[*]`, `$.Environment.Variables{key}`

**Implements:** [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16)

**Scoping** (for [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16)): This hook requires the Lambda Insights extension layer in `$.Layers[*]` and the `AWS_LAMBDA_EXEC_WRAPPER` key in `$.Environment.Variables{key}`. Deployments that omit either setting are denied, so your functions collect detailed CloudWatch metrics.

### Implementation.LAMBDA.Function.49

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `lambda:CreateFunction` — `$.FunctionName`, `$.Layers[*]`
- `lambda:UpdateFunctionConfiguration` — `$.FunctionName`, `$.Layers[*]`

**Implements:** [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16)

**Scoping** (for [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16)): This implementation records `CreateFunction` and `UpdateFunctionConfiguration` calls, including the `$.Layers[*]` value that lists the layers attached to your function. Reviewing these events helps address the mitigation by showing when the layers that carry Lambda Insights and Application Signals were set.

### Implementation.LAMBDA.Function.50

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::Lambda::Function`: `$.configuration.layers[*].arn`, `$.configuration.functionName`

**Implements:** [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16)

**Scoping** (for [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16)): This detective rule checks `$.configuration.layers[*].arn` for the Lambda Insights and CloudWatch Application Signals layers, plus the `AWS_LAMBDA_EXEC_WRAPPER` entry in `$.Environment.Variables{key}`. It identifies functions that collect only the default CloudWatch metrics rather than the detailed metrics these settings enable.

### Implementation.LAMBDA.Function.51

*Guidance — Directive*

Configure the security groups named in your function's VPC configuration following least-privilege best practices. Restrict both inbound and outbound rules to only the sources, destinations, protocols, and ports your function requires. Sources and destinations can be IP address ranges, referenced security groups, or prefix lists. Where the peer is an AWS resource in a security group, reference that security group instead of its address range. The rule then follows the peer's network interfaces rather than addresses that can change. Avoid rules that permit unrestricted access from or to any IP address, such as 0.0.0.0/0 or ::/0. Pay particular attention to outbound rules, because a VPC-attached function's traffic is predominantly egress.

To govern security groups at scale, use AWS Firewall Manager security group policies across your organization in AWS Organizations. You can scope each policy by account, organizational unit, or tag. A common security group policy applies a baseline security group to the resources in scope. A content audit security group policy audits inbound and outbound rules against an allow-list or a block-list. It flags non-compliant security groups in a central dashboard, notifies you, and can automatically remediate them. A usage audit security group policy removes unused or redundant security groups.

To verify unintended access permitted by your security groups, use Amazon VPC Network Access Analyzer. Define a Network Access Scope for the access you consider unintended. For example, match any path that reaches administrative ports 22 or 3389 from an unrestricted address. Then run an analysis with `CreateNetworkInsightsAccessScope` and `StartNetworkInsightsAccessScopeAnalysis`, and review the findings. Network Access Analyzer reasons about the security group together with the route table, network ACL, and public IP assignment. This helps you identify unintended network access relative to your security and compliance requirements.

- [Using content audit security group policies with Firewall Manager](https://docs.aws.amazon.com/waf/latest/developerguide/security-group-policies-audit.html)
- [How to continuously audit and limit security groups with AWS Firewall Manager](https://aws.amazon.com/blogs/security/how-to-continuously-audit-and-limit-security-groups-with-aws-firewall-manager/)
- [What is Network Access Analyzer?](https://docs.aws.amazon.com/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html)

**Implements:** [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17)

**Scoping** (for [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17)): This recommendation directs you to restrict the inbound and outbound rules on your function's VPC security groups to the required addresses, protocols, and ports. Traffic to or from any other address, protocol, or port is then not permitted by those groups.

### Implementation.LAMBDA.Function.52

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::Function`: `$.VpcConfig.SecurityGroupIds[*]`

**Implements:** [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17)

**Scoping** (for [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17)): This implementation validates the groups named in `$.VpcConfig.SecurityGroupIds[*]` at deploy time, resolving `$.securityGroupInfo[*].ipPermissions` and `$.securityGroupInfo[*].ipPermissionsEgress`. Functions that would attach groups permitting unrestricted inbound or outbound traffic are not created, helping reduce your function's network exposure.

### Implementation.LAMBDA.Function.53

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListFunctions`; check `$.Functions[*].FunctionName`, `$.Functions[*].VpcConfig.SecurityGroupIds[*]`
2. Call `ec2:DescribeSecurityGroups` with `$.GroupId[*]`; check `$.securityGroupInfo[*].ipPermissions[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissions[*].fromPort`, `$.securityGroupInfo[*].ipPermissions[*].toPort`, `$.securityGroupInfo[*].ipPermissions[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissions[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissions[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissions[*].prefixListIds[*].prefixListId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipProtocol` = `-1`, `$.securityGroupInfo[*].ipPermissionsEgress[*].fromPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].toPort`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipRanges[*].cidrIp` = `0.0.0.0/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].ipv6Ranges[*].cidrIpv6` = `::/0`, `$.securityGroupInfo[*].ipPermissionsEgress[*].groups[*].groupId`, `$.securityGroupInfo[*].ipPermissionsEgress[*].prefixListIds[*].prefixListId`
3. Call `ec2:GetManagedPrefixListEntries` with `$.PrefixListId`; check `$.entrySet[*].cidr` = `0.0.0.0/0`, `$.entrySet[*].cidr` = `::/0`

**Implements:** [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17)

**Scoping** (for [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17)): This implementation resolves the security groups recorded at `$.Functions[*].VpcConfig.SecurityGroupIds[*]` and inspects their inbound and outbound rules. Flagging rules that admit `0.0.0.0/0` or `::/0` on sensitive ports or all protocols helps you detect functions with broader network exposure than they need.

### Implementation.LAMBDA.Layer.1

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:AddLayerVersionPermission`, `lambda:DeleteLayerVersion`, `lambda:GetLayerVersion`, `lambda:GetLayerVersionPolicy`, `lambda:PublishLayerVersion`, `lambda:RemoveLayerVersionPermission` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1)

**Scoping** (for [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern `lambda:GetLayerVersion`, `lambda:GetLayerVersionPolicy`, `lambda:PublishLayerVersion`, `lambda:DeleteLayerVersion`, `lambda:AddLayerVersionPermission`, and `lambda:RemoveLayerVersionPermission`. This condition helps prevent access to layers outside your organization; it takes effect at the `GetLayerVersion` request that issues a layer archive's 10-minute download link, not at the download itself.

**Data perimeter** (for [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1)): AWS services such as Amazon CloudWatch and AWS AppConfig publish Lambda extensions as layers that your functions can add. Those layers are hosted in AWS-owned accounts outside your organization. Exclude their layer ARNs from this statement so your identities can keep using them. The default resource perimeter policy example shows this exclusion. The service-owned resources reference lists the ARN patterns to exclude.
- [Default resource perimeter SCP example](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)
- [Service-owned resources reference](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_owned_resources.md)

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.LAMBDA.Layer.2

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::Lambda::LayerVersionPermission`: `$.Principal`, `$.OrganizationId`

**Implements:** [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)

**Scoping** (for [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)): At create time, this implementation validates `$.Principal`, the account granted layer usage permission, and `$.OrganizationId`, the organization that scopes a wildcard principal. Stack operations granting `lambda:GetLayerVersion` to accounts outside your organization are blocked, which helps limit who can download your layer archives.

**Scoping** (for [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2)): This implementation validates `$.Principal` and `$.OrganizationId` at create time, rejecting a layer version permission that grants `lambda:GetLayerVersion` to all AWS accounts. Requiring a named account or an organization scope helps limit which principals can read your layer archive.

**Data perimeter** (for [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)): Layer version permissions are declared in CloudFormation as `AWS::Lambda::LayerVersionPermission` resources. A hook on this resource type can reject a template whose `$.Principal` names an account outside your organization. It can also require `$.OrganizationId` to name your own organization whenever `$.Principal` is set to the wildcard value. This helps keep an unscoped wildcard grant from reaching your layer versions through a stack deployment.
- [AWS::Lambda::LayerVersionPermission resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-layerversionpermission.html)
- [Principal property of AWS::Lambda::LayerVersionPermission](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-layerversionpermission.html#cfn-lambda-layerversionpermission-principal)
- [OrganizationId property of AWS::Lambda::LayerVersionPermission](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-lambda-layerversionpermission.html#cfn-lambda-layerversionpermission-organizationid)

### Implementation.LAMBDA.Layer.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListLayers`; check `$.Layers[*].LayerName`
2. Call `lambda:ListLayerVersions` with `$.LayerName`; check `$.LayerVersions[*].Version`, `$.LayerVersions[*].LayerVersionArn`
3. Call `lambda:GetLayerVersionPolicy` with `$.LayerName`, `$.VersionNumber`; check `$.Policy`

**Implements:** [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2)

**Scoping** (for [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2)): This implementation inspects each layer version's resource-based policy at `$.Policy` and flags wildcard-principal grants of `lambda:GetLayerVersion` that carry no `aws:PrincipalOrgID` condition. Flagging those grants helps you detect layer versions readable by every AWS account rather than a defined set of principals.

### Implementation.LAMBDA.Layer.4

*Guidance — Directive*

Consider using AWS Identity and Access Management Access Analyzer external access analyzers to help identify layer versions in your accounts that are shared with untrusted identities. Also use these external access analyzers to help identify functions in your accounts that are shared with untrusted identities. Layer version permissions can grant `lambda:GetLayerVersion` outside your account. That read access returns a link to download the layer archive. Review each finding to confirm whether the sharing is intended. If necessary, remediate with the responsive controls of your choice.

- [AWS Identity and Access Management Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)

**Implements:** [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)

**Scoping** (for [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)): This recommendation directs you to identify layer versions whose resource-based policy shares `lambda:GetLayerVersion` with identities outside your organization. Remediating unintended sharing helps keep your layer content accessible only to trusted identities, including AWS services acting on your behalf.

### Implementation.LAMBDA.Layer.5

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `lambda:AddLayerVersionPermission` — `$.OrganizationId`, `$.Principal`
- `lambda:AddPermission` — `$.Principal`, `$.PrincipalOrgID`, `$.SourceAccount`, `$.SourceArn`

**Implements:** [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)

**Scoping** (for [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)): This implementation records `AddLayerVersionPermission` calls, capturing the grant recipient in the `$.Principal` request parameter and the organization scope in `$.OrganizationId`. Reviewing these events helps you detect a layer version policy change that shares your layer with identities outside your organization.

**Data perimeter:** A layer version policy can grant layer usage to a single account, to all accounts in an organization, or to all AWS accounts. The `$.Principal` request parameter records the recipient of that grant. The `$.OrganizationId` request parameter records the organization it is scoped to. Reviewing both parameters helps you identify a layer version shared with identities outside your organization. You can then remediate with the responsive controls of your choice.
- [AddLayerVersionPermission: Principal request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html#lambda-AddLayerVersionPermission-request-Principal)
- [AddLayerVersionPermission: OrganizationId request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html#lambda-AddLayerVersionPermission-request-OrganizationId)
- [AWS Lambda API Reference: AddLayerVersionPermission](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html)
- [AddPermission: Principal request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html#lambda-AddPermission-request-Principal)
- [AddPermission: PrincipalOrgID request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html#lambda-AddPermission-request-PrincipalOrgID)
- [AddPermission: SourceArn request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html#lambda-AddPermission-request-SourceArn)
- [AddPermission: SourceAccount request parameter](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html#lambda-AddPermission-request-SourceAccount)
- [AWS Lambda API Reference: AddPermission](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html)

### Implementation.LAMBDA.Layer.6

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `lambda:ListLayers`; check `$.Layers[*].LayerName`, `$.Layers[*].LayerArn`
2. Call `lambda:ListLayerVersions` with `$.LayerName`; check `$.LayerVersions[*].Version`, `$.LayerVersions[*].LayerVersionArn`
3. Call `lambda:GetLayerVersionPolicy` with `$.LayerName`, `$.VersionNumber`; check `$.Policy`
4. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`
5. Call `accessanalyzer:CheckAccessNotGranted` with `$.policyDocument`, `$.policyType` = `RESOURCE_POLICY`, `$.access[*].actions[*]` = `lambda:GetLayerVersion`; check `$.result`
6. Call `accessanalyzer:ListFindingsV2` with `$.analyzerArn`, `$.filter`; check `$.findings`

**Implements:** [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)

**Scoping** (for [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)): This rule reads each layer version's resource-based policy at `$.Policy` and identifies grants of `lambda:GetLayerVersion` to other accounts or to all accounts. Reporting those grants helps you keep layer content readable only by trusted identities.

**Reference:** Your AWS Config rule evaluates each layer version's resource-based policy with IAM Access Analyzer custom policy checks. Custom policy checks use automated reasoning to analyze the policy document.

`CheckNoNewAccess` compares that policy against a reference policy you maintain. Your reference policy allowlists the principals that may be granted access. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.

`CheckAccessNotGranted` checks whether that policy grants the actions you list.

An external access analyzer reports resource-based policies that allow access from outside your organization. Your rule reads those findings with `ListFindingsV2`.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.LAMBDA.Layer.7

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `lambda:AddLayerVersionPermission`, `lambda:AddPermission` | `*` | `aws:PrincipalTag/dp:exclude:identity` |

**Implements:** [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)

**Scoping** (for [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)): This implementation restricts `lambda:AddLayerVersionPermission` and `lambda:AddPermission`, while exempting principals tagged for exclusion from the identity perimeter with the `aws:PrincipalTag/dp:exclude:identity` condition key. Requests from other principals are denied, so they cannot grant other accounts access to your layer versions.

**Data perimeter:** AWS Lambda is not among the services that support resource control policies (RCPs). A service control policy (SCP) is therefore the preventive guardrail available for its resource-based policies. `lambda:AddLayerVersionPermission` and `lambda:AddPermission` can grant other accounts or AWS services access to a layer version or a function. Restricting these actions to select principals helps prevent grants to identities outside your organization.
- [AWS Lambda API Reference: AddLayerVersionPermission](https://docs.aws.amazon.com/lambda/latest/api/API_AddLayerVersionPermission.html)
- [AWS Lambda API Reference: AddPermission](https://docs.aws.amazon.com/lambda/latest/api/API_AddPermission.html)
- [Data perimeter policy example: restrict resource policy configurations (SCP)](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_resource_policy_configurations_scp.json)

### ManagedControl.LAMBDA.Function.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if AWS Lambda functions have the log format set to JSON for more control and better readability

**Aliases:** `CONFIG.LAMBDA.DT.5`

**Identifier:** `LAMBDA_FUNCTION_LOG_FORMAT_JSON`

**ARN:** `arn:aws:controlcatalog:::control/3gfur5zvrmdzqfyn1be5to3t9`

Checks if AWS Lambda functions have the log format set to JSON for more control and better readability. The rule is NON_COMPLIANT if configuration.loggingConfig.logFormat is not 'JSON'.

**Implements:** [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1)

### ManagedControl.LAMBDA.Function.2

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require an AWS Lambda function to be in a customer-managed Amazon Virtual Private Cloud (VPC)

**Aliases:** `CT.LAMBDA.PR.3`

**ARN:** `arn:aws:controlcatalog:::control/8jyhx8tw05jw29leqyxxzvf04`

This control checks whether an AWS Lambda function has been configured with access to resources in a customer-managed Amazon Virtual Private Cloud (VPC).

**Implements:** [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)

### ManagedControl.LAMBDA.Function.3

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if a Lambda function is allowed access to a virtual private cloud (VPC)

**Aliases:** `CONFIG.LAMBDA.DT.4`

**Identifier:** `LAMBDA_INSIDE_VPC`

**ARN:** `arn:aws:controlcatalog:::control/jy5yd51dcuq7lndh6tzq1odh`

Checks if a Lambda function is allowed access to a virtual private cloud (VPC). The rule is NON_COMPLIANT if the Lambda function is not VPC enabled.

**Implements:** [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7)

### ManagedControl.LAMBDA.Function.4

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require an AWS Lambda function URL to use AWS IAM-based authentication

**Aliases:** `CT.LAMBDA.PR.5`

**ARN:** `arn:aws:controlcatalog:::control/am05zps6qc75j09wv4b7tqja0`

This control checks whether an AWS Lambda function URL is configured to use authentication that's based on AWS IAM.

**Implements:** [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12)

### ManagedControl.LAMBDA.Function.5

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Lambda has more than 1 availability zone associated

**Aliases:** `CONFIG.LAMBDA.DT.8`

**Identifier:** `LAMBDA_VPC_MULTI_AZ_CHECK`

**ARN:** `arn:aws:controlcatalog:::control/6fbksqcnurv1uuo69nekf29ge`

Checks if Lambda has more than 1 availability zone associated. The rule is NON_COMPLIANT if only 1 availability zone is associated with the Lambda or the number of availability zones associated is less than number specified in the optional parameter.

**Implements:** [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9)

### ManagedControl.LAMBDA.Function.6

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require AWS Lambda function policies to prohibit public access

**Aliases:** `CT.LAMBDA.PR.2`

**ARN:** `arn:aws:controlcatalog:::control/9yksb96puyx3sy9rpmpvxa9np`

This control checks whether an AWS Lambda function resource-based policy prohibits public access.

**Implements:** [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)

### ManagedControl.LAMBDA.Function.7

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Detect whether the AWS Lambda function policy attached to the Lambda resource blocks public access

**Aliases:** `AWS-GR_LAMBDA_FUNCTION_PUBLIC_ACCESS_PROHIBITED`

**Identifier:** `LAMBDA_FUNCTION_PUBLIC_ACCESS_PROHIBITED`

**ARN:** `arn:aws:controlcatalog:::control/b2gzofz99eb7nsuj5g8wcimse`

Detects whether the AWS Lambda function policy attached to the Lambda resource prohibits public access. The rule is NON_COMPLIANT if the Lambda function policy allows public access.

**Implements:** [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13)

### ManagedControl.LAMBDA.Layer.1

*CloudFormation hook — Proactive*

AWS managed control from Controls governing this resource/service.

**Title:** Require an AWS Lambda layer permission to grant access to an AWS organization or specific AWS account

**Aliases:** `CT.LAMBDA.PR.4`

**ARN:** `arn:aws:controlcatalog:::control/3isxiu8l69yubgk9rtgv1vlxg`

This control checks whether an AWS Lambda layer permission has been configured to grant access to an AWS organization or to a specific AWS account only, by ensuring that public access from all AWS accounts has not been granted to a layer.

**Implements:** [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3)

---

## Appendix

### Resources

#### CapacityProvider

*COMPUTE_RESOURCE*

A Lambda capacity provider defines a pool of Amazon EC2 compute that AWS Lambda provisions and operates on your behalf. It is the foundation for running Lambda functions on managed instances rather than on Lambda's default shared fleet. You declare where that compute runs, using VPC subnets and security groups, plus the IAM role Lambda assumes to manage instances.

- IAM resource ARN: `arn:${Partition}:lambda:${Region}:${Account}:capacity-provider:${CapacityProviderName}`
- CloudFormation: `AWS::Lambda::CapacityProvider`
- 9 operations, 12 threats, 8 mitigations

#### CodeSigningConfig

*CONFIGURATION*

An AWS Lambda code signing configuration is the trust definition behind Lambda code signing. It lists the AWS Signer signing profile versions that Lambda treats as trusted publishers, and it sets the action Lambda takes when a signature check fails. A single configuration can be attached to many functions, so a central team can define trust once and apply it broadly.

- IAM resource ARN: `arn:${Partition}:lambda:${Region}:${Account}:code-signing-config:${CodeSigningConfigId}`
- CloudFormation: `AWS::Lambda::CodeSigningConfig`
- AWS Config: `AWS::Lambda::CodeSigningConfig`
- 12 operations, 2 threats, 2 mitigations

#### DurableExecution

*JOB_OR_WORKFLOW*

A Lambda durable execution is a single long-running run of a durable Lambda function. It records progress as checkpoints so work survives interruptions. One execution can span many function invocations and continue for up to a year.

- IAM resource ARN: `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Version}/durable-execution/${ExecutionName}/${ExecutionId}`
- CloudFormation: `AWS::Lambda::DurableExecution`
- 9 operations, 0 threats, 0 mitigations

#### EventSourceMapping

*INTEGRATION*

An event source mapping is a Lambda-managed poller that connects a stream or queue to a Lambda function. Lambda reads records from the source, groups them into batches, and invokes the function with each batch. This lets you consume messages from services that do not invoke functions directly, without running your own consumer fleet. Supported sources include Amazon SQS, Amazon Kinesis, Amazon DynamoDB Streams, Amazon MSK, self-managed Apache Kafka, Amazon MQ, and Amazon DocumentDB change streams.

- IAM resource ARN: `arn:${Partition}:lambda:${Region}:${Account}:event-source-mapping:${UUID}`
- CloudFormation: `AWS::Lambda::EventSourceMapping`
- 8 operations, 12 threats, 7 mitigations

#### Function

*COMPUTE_RESOURCE*

An AWS Lambda function is a unit of serverless compute that runs your code in response to events or direct requests. You package handler code as a .zip archive or a container image, and Lambda provisions and manages the underlying execution environments. Lambda handles capacity, scaling, patching, and fault tolerance, so you do not maintain servers.

- IAM resource ARNs: `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Version}`, `arn:${Partition}:lambda:${Region}:${Account}:function:${FunctionName}:${Alias}`
- CloudFormation: `AWS::Lambda::Function`, `AWS::Lambda::Version`, `AWS::Lambda::Alias`, `AWS::Lambda::Url`, `AWS::Lambda::Permission`, `AWS::Lambda::ResourcePolicy`, `AWS::Lambda::EventInvokeConfig`
- AWS Config: `AWS::Lambda::Function`
- 53 operations, 21 threats, 17 mitigations

#### Layer

*DATA_STORE*

An AWS Lambda layer is a .zip archive of supplementary code or data that functions can share. A layer typically holds library dependencies, a custom runtime, or configuration files. Moving those files out of the function deployment package keeps packages small and lets teams update shared dependencies independently of business logic.

- IAM resource ARNs: `arn:${Partition}:lambda:${Region}:${Account}:layer:${LayerName}`, `arn:${Partition}:lambda:${Region}:${Account}:layer:${LayerName}:${LayerVersion}`
- CloudFormation: `AWS::Lambda::LayerVersion`, `AWS::Lambda::LayerVersionPermission`
- 9 operations, 3 threats, 3 mitigations

### Operations

| Operation | Analyses | Resources | Mitigations |
|---|---|---|---|
| `AddLayerVersionPermission` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) | [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1), [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| `AddPermission` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13), [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14) |
| `CheckpointDurableExecution` | DOCUMENTATION | [DurableExecution](#durableexecution) |  |
| `CreateAlias` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `CreateCapacityProvider` | DOCUMENTATION | [CapacityProvider](#capacityprovider) | [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1), [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2), [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3), [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4), [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5), [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6), [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7), [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8) |
| `CreateCodeSigningConfig` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig) | [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1), [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2) |
| `CreateEventSourceMapping` | DOCUMENTATION, DATA_PERIMETER | [EventSourceMapping](#eventsourcemapping) | [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1), [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2), [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3), [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4), [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5), [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6), [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7) |
| `CreateFunction` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7), [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8), [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9), [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15), [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16), [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17), [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1), [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| `CreateFunctionUrlConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11) |
| `DeleteAlias` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `DeleteCapacityProvider` | DOCUMENTATION | [CapacityProvider](#capacityprovider) |  |
| `DeleteCodeSigningConfig` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig) |  |
| `DeleteEventSourceMapping` | DOCUMENTATION, DATA_PERIMETER | [EventSourceMapping](#eventsourcemapping) | [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1) |
| `DeleteFunction` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12) |
| `DeleteFunctionCodeSigningConfig` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig), [Function](#function) |  |
| `DeleteFunctionConcurrency` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `DeleteFunctionEventInvokeConfig` | DOCUMENTATION | [Function](#function) |  |
| `DeleteFunctionUrlConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `DeleteLayerVersion` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) | [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1) |
| `DeleteProvisionedConcurrencyConfig` | DOCUMENTATION | [Function](#function) |  |
| `DeleteResourcePolicy` | DOCUMENTATION | [Function](#function) | [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12) |
| `GetAccountSettings` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `GetAlias` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `GetCapacityProvider` | DOCUMENTATION | [CapacityProvider](#capacityprovider) | [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1), [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2), [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3), [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4), [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5), [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6), [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8) |
| `GetCodeSigningConfig` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig) | [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1) |
| `GetDurableExecution` | DOCUMENTATION | [DurableExecution](#durableexecution) | [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8) |
| `GetDurableExecutionHistory` | DOCUMENTATION | [DurableExecution](#durableexecution) | [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8) |
| `GetDurableExecutionState` | DOCUMENTATION | [DurableExecution](#durableexecution) |  |
| `GetEventSourceMapping` | DOCUMENTATION | [EventSourceMapping](#eventsourcemapping) | [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1), [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2), [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3), [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4), [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6), [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7) |
| `GetFunction` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7), [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8), [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9), [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14), [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15), [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16), [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17) |
| `GetFunctionCodeSigningConfig` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig), [Function](#function) |  |
| `GetFunctionConcurrency` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `GetFunctionConfiguration` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7), [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8), [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9), [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16), [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17) |
| `GetFunctionEventInvokeConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `GetFunctionRecursionConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `GetFunctionScalingConfig` | DOCUMENTATION | [Function](#function) |  |
| `GetFunctionUrlConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11) |
| `GetLayerVersion` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) | [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1), [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| `GetLayerVersionByArn` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1), [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| `GetLayerVersionPolicy` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) | [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| `GetPolicy` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13), [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14) |
| `GetProvisionedConcurrencyConfig` | DOCUMENTATION | [Function](#function) |  |
| `GetResourcePolicy` | DOCUMENTATION | [Function](#function) | [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13), [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14) |
| `GetRuntimeManagementConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `Invoke` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7), [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16) |
| `InvokeAsync` | DOCUMENTATION | [Function](#function) |  |
| `InvokeWithResponseStream` | DOCUMENTATION | [Function](#function) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7), [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12) |
| `ListAliases` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `ListCapacityProviders` | DOCUMENTATION | [CapacityProvider](#capacityprovider) | [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2), [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3), [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4), [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5), [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6) |
| `ListCodeSigningConfigs` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig) | [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1) |
| `ListDurableExecutionsByFunction` | DOCUMENTATION | [DurableExecution](#durableexecution) |  |
| `ListEventSourceMappings` | DOCUMENTATION, DATA_PERIMETER | [EventSourceMapping](#eventsourcemapping) | [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1), [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2), [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3), [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4), [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6), [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7) |
| `ListFunctionEventInvokeConfigs` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `ListFunctionUrlConfigs` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11) |
| `ListFunctionVersionsByCapacityProvider` | DOCUMENTATION | [CapacityProvider](#capacityprovider) |  |
| `ListFunctions` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7), [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8), [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9), [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17) |
| `ListFunctionsByCodeSigningConfig` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig) |  |
| `ListLayerVersions` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) | [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| `ListLayers` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) |  |
| `ListProvisionedConcurrencyConfigs` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `ListTags` | DOCUMENTATION, DATA_PERIMETER | [CapacityProvider](#capacityprovider), [CodeSigningConfig](#codesigningconfig), [EventSourceMapping](#eventsourcemapping), [Function](#function) | [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7), [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2), [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5), [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15) |
| `ListVersionsByFunction` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3), [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8), [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17) |
| `PublishLayerVersion` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) | [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1) |
| `PublishVersion` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `PutFunctionCodeSigningConfig` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig), [Function](#function) |  |
| `PutFunctionConcurrency` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `PutFunctionEventInvokeConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6) |
| `PutFunctionRecursionConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `PutFunctionScalingConfig` | DOCUMENTATION | [Function](#function) |  |
| `PutProvisionedConcurrencyConfig` | DOCUMENTATION | [Function](#function) |  |
| `PutResourcePolicy` | DOCUMENTATION | [Function](#function) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13), [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14) |
| `PutRuntimeManagementConfig` | DOCUMENTATION | [Function](#function) |  |
| `RemoveLayerVersionPermission` | DOCUMENTATION, DATA_PERIMETER | [Layer](#layer) | [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1), [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| `RemovePermission` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14) |
| `SendDurableExecutionCallbackFailure` | DOCUMENTATION | [DurableExecution](#durableexecution) |  |
| `SendDurableExecutionCallbackHeartbeat` | DOCUMENTATION | [DurableExecution](#durableexecution) |  |
| `SendDurableExecutionCallbackSuccess` | DOCUMENTATION | [DurableExecution](#durableexecution) |  |
| `StopDurableExecution` | DOCUMENTATION | [DurableExecution](#durableexecution) |  |
| `TagResource` | DOCUMENTATION, DATA_PERIMETER | [CapacityProvider](#capacityprovider), [CodeSigningConfig](#codesigningconfig), [EventSourceMapping](#eventsourcemapping), [Function](#function) | [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7), [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2), [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5), [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15) |
| `UntagResource` | DOCUMENTATION, DATA_PERIMETER | [CapacityProvider](#capacityprovider), [CodeSigningConfig](#codesigningconfig), [EventSourceMapping](#eventsourcemapping), [Function](#function) | [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7), [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2), [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5), [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15) |
| `UpdateAlias` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) |  |
| `UpdateCapacityProvider` | DOCUMENTATION | [CapacityProvider](#capacityprovider) | [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1), [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7) |
| `UpdateCodeSigningConfig` | DOCUMENTATION, DATA_PERIMETER | [CodeSigningConfig](#codesigningconfig) | [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1) |
| `UpdateEventSourceMapping` | DOCUMENTATION | [EventSourceMapping](#eventsourcemapping) | [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1), [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2), [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3), [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4), [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6), [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7) |
| `UpdateFunctionCode` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5), [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7), [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8), [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16) |
| `UpdateFunctionConfiguration` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.1](#mitigationlambda1), [Mitigation.LAMBDA.2](#mitigationlambda2), [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1), [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2), [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7), [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8), [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9), [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16), [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17), [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1), [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| `UpdateFunctionEventInvokeConfig` | DOCUMENTATION | [Function](#function) |  |
| `UpdateFunctionUrlConfig` | DOCUMENTATION, DATA_PERIMETER | [Function](#function) | [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11) |

Mitigations attach to an operation directly (the operation appears in the mitigation's request parameters) or through a threat whose request parameters include the operation.

### Goal Coverage (21 of 38)

| Mitigation Goal | Covered | Mitigations |
|---|---|---|
| [AI Services Opt-Out Policy](../mitigation-goals.md#ai-services-opt-out-policy) | — |  |
| [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup) | — |  |
| [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native) | — |  |
| [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery) | yes | [Mitigation.LAMBDA.CapacityProvider.1](#mitigationlambdacapacityprovider1), [Mitigation.LAMBDA.EventSourceMapping.1](#mitigationlambdaeventsourcemapping1), [Mitigation.LAMBDA.Function.1](#mitigationlambdafunction1) |
| [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2) | — |  |
| [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest) | — |  |
| [Cross-Region Replication](../mitigation-goals.md#cross-region-replication) | — |  |
| [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies) | yes | [Mitigation.LAMBDA.Function.13](#mitigationlambdafunction13) |
| [Customer Code Execution - Govern Code Content](../mitigation-goals.md#customer-code-execution---govern-code-content) | — |  |
| [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage) | — |  |
| [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection) | — |  |
| [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics) | yes | [Mitigation.LAMBDA.EventSourceMapping.6](#mitigationlambdaeventsourcemapping6), [Mitigation.LAMBDA.Function.16](#mitigationlambdafunction16) |
| [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics) | — |  |
| [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy) | — |  |
| [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature) | — |  |
| [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication) | — |  |
| [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network) | yes | [Mitigation.LAMBDA.2](#mitigationlambda2) |
| [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources) | yes | [Mitigation.LAMBDA.Function.12](#mitigationlambdafunction12), [Mitigation.LAMBDA.Layer.3](#mitigationlambdalayer3) |
| [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption) | yes | [Mitigation.LAMBDA.CapacityProvider.5](#mitigationlambdacapacityprovider5), [Mitigation.LAMBDA.EventSourceMapping.3](#mitigationlambdaeventsourcemapping3), [Mitigation.LAMBDA.Function.8](#mitigationlambdafunction8) |
| [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment) | — |  |
| [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment) | yes | [Mitigation.LAMBDA.CapacityProvider.6](#mitigationlambdacapacityprovider6), [Mitigation.LAMBDA.EventSourceMapping.4](#mitigationlambdaeventsourcemapping4), [Mitigation.LAMBDA.Function.9](#mitigationlambdafunction9) |
| [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks) | yes | [Mitigation.LAMBDA.Function.2](#mitigationlambdafunction2) |
| [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks) | yes | [Mitigation.LAMBDA.Function.5](#mitigationlambdafunction5) |
| [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement) | yes | [Mitigation.LAMBDA.CapacityProvider.3](#mitigationlambdacapacityprovider3), [Mitigation.LAMBDA.Function.4](#mitigationlambdafunction4) |
| [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets) | yes | [Mitigation.LAMBDA.CodeSigningConfig.1](#mitigationlambdacodesigningconfig1) |
| [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources) | yes | [Mitigation.LAMBDA.Function.6](#mitigationlambdafunction6), [Mitigation.LAMBDA.Layer.1](#mitigationlambdalayer1) |
| [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network) | yes | [Mitigation.LAMBDA.1](#mitigationlambda1) |
| [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition) | yes | [Mitigation.LAMBDA.Function.10](#mitigationlambdafunction10), [Mitigation.LAMBDA.Layer.2](#mitigationlambdalayer2) |
| [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention) | — |  |
| [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose) | yes | [Mitigation.LAMBDA.CapacityProvider.7](#mitigationlambdacapacityprovider7), [Mitigation.LAMBDA.CodeSigningConfig.2](#mitigationlambdacodesigningconfig2), [Mitigation.LAMBDA.EventSourceMapping.5](#mitigationlambdaeventsourcemapping5), [Mitigation.LAMBDA.Function.15](#mitigationlambdafunction15) |
| [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role) | yes | [Mitigation.LAMBDA.CapacityProvider.2](#mitigationlambdacapacityprovider2), [Mitigation.LAMBDA.Function.3](#mitigationlambdafunction3) |
| [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access) | yes | [Mitigation.LAMBDA.CapacityProvider.8](#mitigationlambdacapacityprovider8), [Mitigation.LAMBDA.EventSourceMapping.7](#mitigationlambdaeventsourcemapping7), [Mitigation.LAMBDA.Function.17](#mitigationlambdafunction17) |
| [Service Principal Presigned URL](../mitigation-goals.md#service-principal-presigned-url) | yes | [Mitigation.LAMBDA.Function.14](#mitigationlambdafunction14) |
| [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration) | — |  |
| [Turn on Block Public Access (BPA)](../mitigation-goals.md#turn-on-block-public-access-bpa) | — |  |
| [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention) | yes | [Mitigation.LAMBDA.Function.11](#mitigationlambdafunction11) |
| [Use Nitro instances](../mitigation-goals.md#use-nitro-instances) | yes | [Mitigation.LAMBDA.CapacityProvider.4](#mitigationlambdacapacityprovider4) |
| [VPC Deployment](../mitigation-goals.md#vpc-deployment) | yes | [Mitigation.LAMBDA.EventSourceMapping.2](#mitigationlambdaeventsourcemapping2), [Mitigation.LAMBDA.Function.7](#mitigationlambdafunction7) |

### Impact Rollup

| Impact | Threats |
|---|---|
| [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery) | [Threat.LAMBDA.CapacityProvider.6](#threatlambdacapacityprovider6), [Threat.LAMBDA.CapacityProvider.7](#threatlambdacapacityprovider7), [Threat.LAMBDA.CapacityProvider.9](#threatlambdacapacityprovider9) |
| [Credential Theft](../threat-impacts.md#credential-theft) | [Threat.LAMBDA.Function.15](#threatlambdafunction15) |
| [Data Collection](../threat-impacts.md#data-collection) | [Threat.LAMBDA.1](#threatlambda1), [Threat.LAMBDA.2](#threatlambda2), [Threat.LAMBDA.CapacityProvider.3](#threatlambdacapacityprovider3), [Threat.LAMBDA.CapacityProvider.8](#threatlambdacapacityprovider8), [Threat.LAMBDA.CodeSigningConfig.1](#threatlambdacodesigningconfig1), [Threat.LAMBDA.EventSourceMapping.5](#threatlambdaeventsourcemapping5), [Threat.LAMBDA.EventSourceMapping.8](#threatlambdaeventsourcemapping8), [Threat.LAMBDA.EventSourceMapping.10](#threatlambdaeventsourcemapping10), [Threat.LAMBDA.Function.5](#threatlambdafunction5), [Threat.LAMBDA.Function.8](#threatlambdafunction8), [Threat.LAMBDA.Function.11](#threatlambdafunction11), [Threat.LAMBDA.Function.12](#threatlambdafunction12), [Threat.LAMBDA.Function.13](#threatlambdafunction13), [Threat.LAMBDA.Function.14](#threatlambdafunction14), [Threat.LAMBDA.Function.15](#threatlambdafunction15), [Threat.LAMBDA.Function.16](#threatlambdafunction16), [Threat.LAMBDA.Layer.2](#threatlambdalayer2), [Threat.LAMBDA.Layer.3](#threatlambdalayer3) |
| [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Threat.LAMBDA.CapacityProvider.1](#threatlambdacapacityprovider1), [Threat.LAMBDA.EventSourceMapping.3](#threatlambdaeventsourcemapping3), [Threat.LAMBDA.Function.3](#threatlambdafunction3) |
| [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Threat.LAMBDA.1](#threatlambda1), [Threat.LAMBDA.2](#threatlambda2), [Threat.LAMBDA.CapacityProvider.11](#threatlambdacapacityprovider11), [Threat.LAMBDA.CapacityProvider.12](#threatlambdacapacityprovider12), [Threat.LAMBDA.CodeSigningConfig.1](#threatlambdacodesigningconfig1), [Threat.LAMBDA.EventSourceMapping.1](#threatlambdaeventsourcemapping1), [Threat.LAMBDA.EventSourceMapping.8](#threatlambdaeventsourcemapping8), [Threat.LAMBDA.EventSourceMapping.10](#threatlambdaeventsourcemapping10), [Threat.LAMBDA.EventSourceMapping.12](#threatlambdaeventsourcemapping12), [Threat.LAMBDA.Function.1](#threatlambdafunction1), [Threat.LAMBDA.Function.6](#threatlambdafunction6), [Threat.LAMBDA.Function.8](#threatlambdafunction8), [Threat.LAMBDA.Function.10](#threatlambdafunction10), [Threat.LAMBDA.Function.11](#threatlambdafunction11), [Threat.LAMBDA.Function.12](#threatlambdafunction12), [Threat.LAMBDA.Function.13](#threatlambdafunction13), [Threat.LAMBDA.Function.14](#threatlambdafunction14), [Threat.LAMBDA.Function.15](#threatlambdafunction15), [Threat.LAMBDA.Function.16](#threatlambdafunction16), [Threat.LAMBDA.Function.19](#threatlambdafunction19), [Threat.LAMBDA.Function.20](#threatlambdafunction20), [Threat.LAMBDA.Function.21](#threatlambdafunction21), [Threat.LAMBDA.Layer.1](#threatlambdalayer1), [Threat.LAMBDA.Layer.2](#threatlambdalayer2), [Threat.LAMBDA.Layer.3](#threatlambdalayer3) |
| [Defense Evasion](../threat-impacts.md#defense-evasion) | [Threat.LAMBDA.CapacityProvider.2](#threatlambdacapacityprovider2), [Threat.LAMBDA.CapacityProvider.10](#threatlambdacapacityprovider10), [Threat.LAMBDA.CapacityProvider.11](#threatlambdacapacityprovider11), [Threat.LAMBDA.CapacityProvider.12](#threatlambdacapacityprovider12), [Threat.LAMBDA.CodeSigningConfig.2](#threatlambdacodesigningconfig2), [Threat.LAMBDA.EventSourceMapping.1](#threatlambdaeventsourcemapping1), [Threat.LAMBDA.EventSourceMapping.2](#threatlambdaeventsourcemapping2), [Threat.LAMBDA.EventSourceMapping.4](#threatlambdaeventsourcemapping4), [Threat.LAMBDA.EventSourceMapping.6](#threatlambdaeventsourcemapping6), [Threat.LAMBDA.EventSourceMapping.9](#threatlambdaeventsourcemapping9), [Threat.LAMBDA.EventSourceMapping.11](#threatlambdaeventsourcemapping11), [Threat.LAMBDA.EventSourceMapping.12](#threatlambdaeventsourcemapping12), [Threat.LAMBDA.Function.1](#threatlambdafunction1), [Threat.LAMBDA.Function.2](#threatlambdafunction2), [Threat.LAMBDA.Function.4](#threatlambdafunction4), [Threat.LAMBDA.Function.7](#threatlambdafunction7), [Threat.LAMBDA.Function.10](#threatlambdafunction10), [Threat.LAMBDA.Function.18](#threatlambdafunction18), [Threat.LAMBDA.Function.19](#threatlambdafunction19), [Threat.LAMBDA.Function.20](#threatlambdafunction20), [Threat.LAMBDA.Function.21](#threatlambdafunction21) |
| [Endpoint Denial of Service](../threat-impacts.md#endpoint-denial-of-service) | [Threat.LAMBDA.CapacityProvider.4](#threatlambdacapacityprovider4), [Threat.LAMBDA.EventSourceMapping.7](#threatlambdaeventsourcemapping7), [Threat.LAMBDA.Function.9](#threatlambdafunction9) |
| [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Threat.LAMBDA.CapacityProvider.1](#threatlambdacapacityprovider1), [Threat.LAMBDA.Function.3](#threatlambdafunction3) |
| [Initial Access](../threat-impacts.md#initial-access) | [Threat.LAMBDA.CapacityProvider.6](#threatlambdacapacityprovider6), [Threat.LAMBDA.CapacityProvider.7](#threatlambdacapacityprovider7), [Threat.LAMBDA.CapacityProvider.9](#threatlambdacapacityprovider9), [Threat.LAMBDA.Function.6](#threatlambdafunction6), [Threat.LAMBDA.Function.10](#threatlambdafunction10), [Threat.LAMBDA.Function.12](#threatlambdafunction12), [Threat.LAMBDA.Function.13](#threatlambdafunction13), [Threat.LAMBDA.Layer.1](#threatlambdalayer1), [Threat.LAMBDA.Layer.3](#threatlambdalayer3) |
| [Lateral Movement](../threat-impacts.md#lateral-movement) | [Threat.LAMBDA.CapacityProvider.5](#threatlambdacapacityprovider5) |
| [Malicious Code Execution](../threat-impacts.md#malicious-code-execution) | [Threat.LAMBDA.CodeSigningConfig.1](#threatlambdacodesigningconfig1) |
| [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Threat.LAMBDA.CapacityProvider.9](#threatlambdacapacityprovider9), [Threat.LAMBDA.Function.17](#threatlambdafunction17) |
