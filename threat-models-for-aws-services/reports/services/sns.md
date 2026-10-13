# Amazon Simple Notification Service (sns) Threat Model

[Threat Summary](#threat-summary) · [Threats](#threats) · [Mitigations](#mitigations) · [Implementations](#implementations) · [Appendix](#appendix)

## Overview

Amazon Simple Notification Service is a fully managed messaging service that delivers messages from publishers to subscribers. Publishers send messages to a topic, which acts as a logical access point and communication channel. Subscribers attached to that topic receive each message asynchronously, so producers and consumers stay decoupled.

26 threats and 21 mitigations across 8 resource types and 42 API operations. 16 of 38 mitigation goals are covered, implemented by 65 implementations.

Source service model: [sns](https://github.com/aws/api-models-aws/blob/a0767ac42e279dfc8b311564c33436f8abfd3031/models/sns/service/2010-03-31/sns-2010-03-31.json)

---

## Threat Summary

### [Cross-service confused deputy problem through resource-based policies](../potential-threats.md#cross-service-confused-deputy-problem-through-resource-based-policies)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.12](#threatsnstopic12) | Untrusted IAM principal, with access to configure an AWS service that has been granted access to an Amazon SNS topic through its resource-based policy, may be able to cause that AWS service to interact with the topic using the principal's data or context. | [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Mitigation.SNS.Topic.9](#mitigationsnstopic9) |

### [Data exposure from missing encryption at rest](../potential-threats.md#data-exposure-from-missing-encryption-at-rest)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.5](#threatsnstopic5) | IAM principal, with IAM permissions to create or modify an Amazon SNS topic, can create or modify the topic to not use encryption at rest. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.SNS.Topic.3](#mitigationsnstopic3), [Mitigation.SNS.Topic.6](#mitigationsnstopic6) |

### [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Subscription.4](#threatsnssubscription4) | Threat actor, with network adjacency to the connection path between clients and an Amazon SNS subscription that has not been configured to encrypt data in transit, can capture subscription network traffic and read data in transit that has not been encrypted. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4) |
| [Threat.SNS.Topic.14](#threatsnstopic14) | Threat actor, with network adjacency to the connection path between clients and an Amazon SNS topic that has not been configured to encrypt data in transit, can capture topic network traffic and read data in transit that has not been encrypted. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection) | [Mitigation.SNS.Topic.10](#mitigationsnstopic10) |

### [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.1](#threatsnstopic1) | IAM principal, with IAM permissions to create or modify data in an Amazon SNS topic, can encrypt the topic's stored message bodies and FIFO message archive with key material outside the AWS account owner's control. | [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Mitigation.SNS.Topic.6](#mitigationsnstopic6) |

### [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.PlatformApplication.2](#threatsnsplatformapplication2) | IAM principal, with IAM permissions to use an Amazon SNS platform application, can interact with the platform application without a record of their activities being persisted by delivery status logging to CloudWatch Logs. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1) |
| [Threat.SNS.SmsAttribute.2](#threatsnssmsattribute2) | IAM principal, with IAM permissions to use an Amazon SNS SMS attribute, can interact with the SMS attribute without a record of their activities being persisted by SMS delivery status logging to CloudWatch Logs. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1) |
| [Threat.SNS.Topic.11](#threatsnstopic11) | IAM principal, with IAM permissions to use an Amazon SNS topic, can publish messages that fan out to subscribed endpoints without a record of those deliveries being persisted by message delivery status logging to Amazon CloudWatch Logs, while their control-plane actions remain recorded by always-on AWS CloudTrail management events. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.SNS.Topic.1](#mitigationsnstopic1) |

### [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.1](#threatsns1) | Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon SNS service API calls. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.SNS.2](#mitigationsns2) |

### [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.9](#threatsnstopic9) | Untrusted IAM principal, that has been granted access to my Amazon SNS topic through its resource-based policy, can use my topic. | [Data Collection](../threat-impacts.md#data-collection), [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |

### [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.PlatformApplication.1](#threatsnsplatformapplication1) | Untrusted IAM principal, that has been granted access to my Amazon SNS platform application's IAM success feedback role and failure feedback role through their trust policies, can assume my success feedback role and failure feedback role and act with their permissions. | [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2) |
| [Threat.SNS.SmsAttribute.1](#threatsnssmsattribute1) | Untrusted IAM principal, that has been granted access to my Amazon SNS SMS attribute's delivery status IAM role through its trust policy, can assume my delivery status IAM role and act with its permissions. | [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2) |
| [Threat.SNS.Subscription.3](#threatsnssubscription3) | Untrusted IAM principal, that has been granted access to my Amazon SNS subscription's IAM subscription role through its trust policy, can assume my subscription role and act with its permissions. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1) |
| [Threat.SNS.Topic.10](#threatsnstopic10) | Untrusted IAM principal, that has been granted access to my Amazon SNS topic's IAM success feedback role and failure feedback role through their trust policies, can assume my success feedback role and failure feedback role and act with their permissions. | [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.SNS.Topic.2](#mitigationsnstopic2) |

### [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.3](#threatsnstopic3) | IAM principal, with IAM permissions to create or modify an Amazon SNS topic, can encrypt the topic with an AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle. | [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.SNS.Topic.6](#mitigationsnstopic6) |

### [Logging disablement](../potential-threats.md#logging-disablement)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.PlatformApplication.3](#threatsnsplatformapplication3) | IAM principal, with IAM permissions to modify an Amazon SNS platform application, can disable delivery status logging to CloudWatch Logs for the platform application. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1) |
| [Threat.SNS.SmsAttribute.3](#threatsnssmsattribute3) | IAM principal, with IAM permissions to modify an Amazon SNS SMS attribute, can disable SMS delivery status logging to CloudWatch Logs and daily SMS usage report delivery to Amazon S3 for the SMS attribute. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1) |
| [Threat.SNS.Topic.13](#threatsnstopic13) | IAM principal, with IAM permissions to modify an Amazon SNS topic's delivery status logging, can disable message delivery status logging to CloudWatch Logs for the topic. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.SNS.Topic.1](#mitigationsnstopic1) |

### [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.8](#threatsnstopic8) | IAM principal, with IAM permissions on my Amazon SNS topic and access to an unexpected network, can interact with my topic from an unexpected network. | [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.SNS.Topic.4](#mitigationsnstopic4) |

### [Resource Configuration - Data movement through an untrusted configured target](../potential-threats.md#resource-configuration---data-movement-through-an-untrusted-configured-target)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Subscription.1](#threatsnssubscription1) | My IAM principal, with IAM permissions to create or modify my Amazon SNS subscription, can configure my subscription to use an untrusted delivery endpoint (an Amazon SQS queue, AWS Lambda function, Amazon Data Firehose delivery stream, mobile push platform endpoint, HTTP/S URL, email address, or SMS phone number) to which Amazon SNS delivers every message published to the topic. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3) |

### [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.2](#threatsnstopic2) | IAM principal, with IAM permissions to create or modify an Amazon SNS topic, can configure the topic without required tagging information. | [Defense Evasion](../threat-impacts.md#defense-evasion) | [Mitigation.SNS.Topic.11](#mitigationsnstopic11) |

### [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.2](#threatsns2) | IAM principal, with IAM permissions on untrusted Amazon SNS resources, can interact with those untrusted resources from a governed VPC network. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.SNS.1](#mitigationsns1) |

### [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.7](#threatsnstopic7) | My IAM principal, with IAM permissions on an untrusted Amazon SNS topic, can interact with the untrusted topic, including sending data to it or receiving data from it. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.SNS.Topic.5](#mitigationsnstopic5) |

### [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Topic.6](#threatsnstopic6) | IAM principal, with unintended IAM permissions to read messages from an Amazon SNS topic where the service decrypts data under its own service identity, can read messages in the topic because the KMS key policy cannot restrict which IAM principals can decrypt the data. | [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |

### [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)

| Threat | Summary | Impacts | Mitigated by |
|---|---|---|---|
| [Threat.SNS.Subscription.2](#threatsnssubscription2) | Threat actor, with a method for enumerating public Amazon SNS subscriptions, or prior knowledge of an existing public subscription, can access a subscription that has been configured to allow public access. | [Initial Access](../threat-impacts.md#initial-access), [Service Stop](../threat-impacts.md#service-stop) | [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2) |
| [Threat.SNS.Topic.4](#threatsnstopic4) | Threat actor, with a method for enumerating public Amazon SNS topics, or prior knowledge of an existing public topic, can access a topic that has been configured to allow public access. | [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access) | [Mitigation.SNS.Topic.7](#mitigationsnstopic7) |

---

## Threats

Each scoping statement below describes the exposure the threat's mapped mitigations address. It assumes those mitigations are not applied and makes no claim about your resources.

### Threat.SNS.1

Untrusted IAM principal, operating from a governed VPC network, can use the governed VPC network to make Amazon SNS service API calls.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.SNS.2](#mitigationsns2)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access from a governed VPC network](../potential-threats.md#identity-perimeter---untrusted-principal-access-from-a-governed-vpc-network)): Amazon SNS is reachable from your VPC through an interface VPC endpoint (powered by AWS PrivateLink). An IAM principal from an account outside your organization can use that network path to call the Amazon SNS API.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:AddPermission` | `$.AWSAccountId[*]`, `$.ActionName[*]`, `$.TopicArn` |
| `sns:DeleteTopic` | `$.TopicArn` |
| `sns:Publish` | `$.Message`, `$.PhoneNumber`, `$.TargetArn`, `$.TopicArn` |
| `sns:PublishBatch` | `$.PublishBatchRequestEntries[*].Message`, `$.TopicArn` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |
| `sns:Subscribe` | `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

### Threat.SNS.2

IAM principal, with IAM permissions on untrusted Amazon SNS resources, can interact with those untrusted resources from a governed VPC network.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.SNS.1](#mitigationsns1)

**Frameworks:** MITRE ATT&CK [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Access to untrusted resource from a governed VPC network](../potential-threats.md#resource-perimeter---access-to-untrusted-resource-from-a-governed-vpc-network)): An IAM principal with permissions on those resources can call `Publish` or `Subscribe` through an interface VPC endpoint (powered by AWS PrivateLink). The `$.TopicArn` request parameter can name a topic owned by an account outside your organization. That access enables publishing messages from your VPC to that topic, or subscribing an endpoint to the messages it delivers.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:AddPermission` | `$.AWSAccountId[*]`, `$.ActionName[*]`, `$.TopicArn` |
| `sns:DeleteTopic` | `$.TopicArn` |
| `sns:Publish` | `$.Message`, `$.TargetArn`, `$.TopicArn` |
| `sns:PublishBatch` | `$.PublishBatchRequestEntries[*].Message`, `$.TopicArn` |
| `sns:RemovePermission` | `$.TopicArn` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |
| `sns:Subscribe` | `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

### Threat.SNS.PlatformApplication.1

Untrusted IAM principal, that has been granted access to my Amazon SNS platform application's IAM success feedback role and failure feedback role through their trust policies, can assume my success feedback role and failure feedback role and act with their permissions.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1548.005](https://attack.mitre.org/techniques/T1548/005/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon SNS platform application designates a success feedback role and a failure feedback role for push delivery status logging through `SuccessFeedbackRoleArn` and `FailureFeedbackRoleArn`. The roles' trust policies can allow principals outside your organization to assume the roles and act with their CloudWatch Logs permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:CreatePlatformApplication` | `$.Attributes{key}` |
| `sns:SetPlatformApplicationAttributes` | `$.Attributes{key}` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.SNS.PlatformApplication.2

IAM principal, with IAM permissions to use an Amazon SNS platform application, can interact with the platform application without a record of their activities being persisted by delivery status logging to CloudWatch Logs.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): Delivery status logging to CloudWatch Logs emits nothing for a platform application until you configure it. Until then, an IAM principal can use the platform application to send messages to mobile endpoints with no delivery status records persisted.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:Publish` | `$.Message`, `$.MessageStructure`, `$.TargetArn`, `$.TopicArn` |

### Threat.SNS.PlatformApplication.3

IAM principal, with IAM permissions to modify an Amazon SNS platform application, can disable delivery status logging to CloudWatch Logs for the platform application.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): Delivery status logging for an Amazon SNS platform application is opt-in, turned on by the feedback role ARN entries in the `$.Attributes{key}` map. An IAM principal that can call `SetPlatformApplicationAttributes` can clear those entries, returning the application to a state that emits no push delivery records; setting `SuccessFeedbackSampleRate` to 0 in the same call stops the records of successful deliveries while failure records continue.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:SetPlatformApplicationAttributes` | `$.Attributes{key}`, `$.PlatformApplicationArn` |

### Threat.SNS.SmsAttribute.1

Untrusted IAM principal, that has been granted access to my Amazon SNS SMS attribute's delivery status IAM role through its trust policy, can assume my delivery status IAM role and act with its permissions.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1548.005](https://attack.mitre.org/techniques/T1548/005/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon SNS SMS attribute designates an IAM service role through the `DeliveryStatusIAMRole` key of `$.attributes{key}`. You author that role's trust policy, so it can authorize principals outside your organization to assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:SetSMSAttributes` | `$.attributes{key}` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.SNS.SmsAttribute.2

IAM principal, with IAM permissions to use an Amazon SNS SMS attribute, can interact with the SMS attribute without a record of their activities being persisted by SMS delivery status logging to CloudWatch Logs.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): The Amazon SNS SMS attribute holds your account-level text messaging preferences for SMS messages sent through `Publish` to a phone number. While delivery status logging is unconfigured, those sends leave no per-message delivery status record in CloudWatch Logs.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:Publish` | `$.Message`, `$.MessageAttributes{key}.StringValue`, `$.PhoneNumber` |

### Threat.SNS.SmsAttribute.3

IAM principal, with IAM permissions to modify an Amazon SNS SMS attribute, can disable SMS delivery status logging to CloudWatch Logs and daily SMS usage report delivery to Amazon S3 for the SMS attribute.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with permissions to modify the SMS attribute can clear the stored `DeliveryStatusIAMRole` or `UsageReportS3Bucket` value with a `SetSMSAttributes` call. Amazon SNS can then stop writing SMS delivery status logs to CloudWatch Logs or the daily usage report to Amazon S3.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:SetSMSAttributes` | `$.attributes{key}` |

### Threat.SNS.Subscription.1

My IAM principal, with IAM permissions to create or modify my Amazon SNS subscription, can configure my subscription to use an untrusted delivery endpoint (an Amazon SQS queue, AWS Lambda function, Amazon Data Firehose delivery stream, mobile push platform endpoint, HTTP/S URL, email address, or SMS phone number) to which Amazon SNS delivers every message published to the topic.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Configuration - Data movement through an untrusted configured target](../potential-threats.md#resource-configuration---data-movement-through-an-untrusted-configured-target)): An IAM principal that creates your subscription can direct every message published to the topic to an endpoint outside your organization or outside AWS. An IAM principal that modifies the subscription can broaden the set of messages that endpoint receives.

**Data perimeter:** `Subscribe` allows you to specify an HTTP/HTTPS endpoint, an email address, or an SMS phone number as the value for the `Endpoint` and `Protocol` request parameters. Because the subsequent requests against the endpoints are not governed by IAM, they are not restricted with `aws:ResourceOrgID` implemented in a service control policy.

**Data perimeter:** `Subscribe` allows you to specify a resource, such as an SQS queue and a Lambda function, that does not belong to your organization as the value for the `Endpoint` and `Protocol` request parameters. Because the subsequent calls against the resources are performed by the service principal, they are not restricted with `aws:ResourceOrgID` implemented in a service control policy.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:ConfirmSubscription` | `$.Token`, `$.TopicArn` |
| `sns:SetSubscriptionAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.SubscriptionArn` |
| `sns:Subscribe` | `$.Attributes{key}`, `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

### Threat.SNS.Subscription.2

Threat actor, with a method for enumerating public Amazon SNS subscriptions, or prior knowledge of an existing public subscription, can access a subscription that has been configured to allow public access.

**Impacts:** [Initial Access](../threat-impacts.md#initial-access), [Service Stop](../threat-impacts.md#service-stop)

**Mitigations:** [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2)

**Frameworks:** MITRE ATT&CK [T1489](https://attack.mitre.org/techniques/T1489/)

**Scoping** (for [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)): An Amazon SNS subscription can be configured to allow access by callers without AWS credentials. A threat actor who enumerates or already knows such a subscription may then delete it and interrupt delivery to the legitimate endpoint.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:ConfirmSubscription` | `$.Token`, `$.TopicArn` |
| `sns:Unsubscribe` | `$.SubscriptionArn` |

### Threat.SNS.Subscription.3

Untrusted IAM principal, that has been granted access to my Amazon SNS subscription's IAM subscription role through its trust policy, can assume my subscription role and act with its permissions.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): Your Amazon SNS subscription using the `firehose` protocol names a subscription role in its `SubscriptionRoleArn` attribute. The trust policy you author for that role can allow principals outside your organization to assume the role and act with its permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:SetSubscriptionAttributes` | `$.AttributeName`, `$.AttributeValue` |
| `sns:Subscribe` | `$.Attributes{key}` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.SNS.Subscription.4

Threat actor, with network adjacency to the connection path between clients and an Amazon SNS subscription that has not been configured to encrypt data in transit, can capture subscription network traffic and read data in transit that has not been encrypted.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Protocols:** `HTTP`, `SMTP`, `SMS`

**Scoping** (for [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)): Amazon SNS supports cleartext HTTP as a delivery transport to a subscribed endpoint, and those notifications cross the public internet. Delivery to email and SMS endpoints uses SMTP and carrier networks, for which the subscription exposes no transport encryption setting.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:GetSubscriptionAttributes` | `$.SubscriptionArn` |
| `sns:ListSubscriptions` | *(any parameter)* |
| `sns:ListSubscriptionsByTopic` | `$.TopicArn` |
| `sns:SetSubscriptionAttributes` | `$.AttributeName`, `$.AttributeValue` |
| `sns:Subscribe` | `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

### Threat.SNS.Topic.1

IAM principal, with IAM permissions to create or modify data in an Amazon SNS topic, can encrypt the topic's stored message bodies and FIFO message archive with key material outside the AWS account owner's control.

**Impacts:** [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact), [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery)

**Mitigations:** [Mitigation.SNS.Topic.6](#mitigationsnstopic6)

**Frameworks:** MITRE ATT&CK [T1486](https://attack.mitre.org/techniques/T1486/), [T1490](https://attack.mitre.org/techniques/T1490/)

**Scoping** (for [Data loss through inaccessible encryption key](../potential-threats.md#data-loss-through-inaccessible-encryption-key)): An IAM principal with `sns:CreateTopic` or `sns:SetTopicAttributes` can set the topic's `KmsMasterKeyId` attribute to a KMS key your account does not administer. If that key becomes unavailable, the topic's stored message bodies and FIFO message archive remain encrypted and undecryptable.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

### Threat.SNS.Topic.2

IAM principal, with IAM permissions to create or modify an Amazon SNS topic, can configure the topic without required tagging information.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.SNS.Topic.11](#mitigationsnstopic11)

**Scoping** (for [Resource missing required tags](../potential-threats.md#resource-missing-required-tags)): An IAM principal with permissions to create or modify a topic can call `CreateTopic` without supplying tags, because Amazon SNS does not require them. The topic then has no tag recording its owner or purpose for ownership attribution, cost allocation, and compliance inventory.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Tags[*].Key`, `$.Tags[*].Value` |

### Threat.SNS.Topic.3

IAM principal, with IAM permissions to create or modify an Amazon SNS topic, can encrypt the topic with an AWS managed KMS key, limiting your ability to govern the key's policy and lifecycle.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.SNS.Topic.6](#mitigationsnstopic6)

**Scoping** (for [Limited encryption governance with AWS owned or managed KMS keys](../potential-threats.md#limited-encryption-governance-with-aws-owned-or-managed-kms-keys)): An IAM principal with permission to create or modify your topic can set `KmsMasterKeyId` to the AWS managed key `alias/aws/sns`. You cannot edit that key's policy, change its yearly rotation schedule, or schedule the key for deletion.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

### Threat.SNS.Topic.4

Threat actor, with a method for enumerating public Amazon SNS topics, or prior knowledge of an existing public topic, can access a topic that has been configured to allow public access.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.SNS.Topic.7](#mitigationsnstopic7)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/), [T1580](https://attack.mitre.org/techniques/T1580/)

**Scoping** (for [Unintended public resource access](../potential-threats.md#unintended-public-resource-access)): Your topic's resource-based policy can grant `sns:Subscribe` to a wildcard principal, leaving the topic reachable by principals in accounts outside your organization. A threat actor who learns the topic ARN can then subscribe an endpoint they control and receive the messages you publish.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:ConfirmSubscription` | `$.Token`, `$.TopicArn` |
| `sns:GetDataProtectionPolicy` | `$.ResourceArn` |
| `sns:GetTopicAttributes` | `$.TopicArn` |
| `sns:ListSubscriptionsByTopic` | `$.TopicArn` |
| `sns:ListTagsForResource` | `$.ResourceArn` |
| `sns:Publish` | `$.Message`, `$.TopicArn` |
| `sns:PublishBatch` | `$.PublishBatchRequestEntries[*].Message`, `$.TopicArn` |
| `sns:Subscribe` | `$.Attributes{key}`, `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

### Threat.SNS.Topic.5

IAM principal, with IAM permissions to create or modify an Amazon SNS topic, can create or modify the topic to not use encryption at rest.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.SNS.Topic.3](#mitigationsnstopic3), [Mitigation.SNS.Topic.6](#mitigationsnstopic6)

**Scoping** (for [Data exposure from missing encryption at rest](../potential-threats.md#data-exposure-from-missing-encryption-at-rest)): Amazon SNS durably stores the message bodies published to your topic. An IAM principal with permissions to create or modify the topic can leave those stored payloads without encryption at rest.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

### Threat.SNS.Topic.6

IAM principal, with unintended IAM permissions to read messages from an Amazon SNS topic where the service decrypts data under its own service identity, can read messages in the topic because the KMS key policy cannot restrict which IAM principals can decrypt the data.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Frameworks:** MITRE ATT&CK [T1048](https://attack.mitre.org/techniques/T1048/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Unauthorized data access through service-managed KMS grants](../potential-threats.md#unauthorized-data-access-through-service-managed-kms-grants)): Amazon SNS decrypts your topic's messages under its own service identity before delivery, so the key policy cannot restrict which principals receive them. A customer managed key gives you audit, lifecycle control, and availability, not a per-principal gate on reading the messages.

**Data perimeter:** `AddPermission` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Data perimeter:** `CreateTopic` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Data perimeter:** `SetTopicAttributes` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:SetSubscriptionAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.SubscriptionArn` |
| `sns:Subscribe` | `$.Attributes{key}`, `$.Endpoint`, `$.Protocol`, `$.TopicArn` |
| `sqs:ReceiveMessage` | `$.QueueUrl` |

### Threat.SNS.Topic.7

My IAM principal, with IAM permissions on an untrusted Amazon SNS topic, can interact with the untrusted topic, including sending data to it or receiving data from it.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Exfiltration](../threat-impacts.md#data-exfiltration)

**Mitigations:** [Mitigation.SNS.Topic.5](#mitigationsnstopic5)

**Frameworks:** MITRE ATT&CK [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Resource Perimeter - Data exfiltration to untrusted resource](../potential-threats.md#resource-perimeter---data-exfiltration-to-untrusted-resource)): An IAM principal in your accounts can target a topic owned by an account outside your organization in Amazon SNS API calls. With IAM permissions on that topic, the principal can send data to it or change its permissions and attributes.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:AddPermission` | `$.AWSAccountId[*]`, `$.ActionName[*]`, `$.TopicArn` |
| `sns:DeleteTopic` | `$.TopicArn` |
| `sns:Publish` | `$.Message`, `$.TopicArn` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

### Threat.SNS.Topic.8

IAM principal, with IAM permissions on my Amazon SNS topic and access to an unexpected network, can interact with my topic from an unexpected network.

**Impacts:** [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.SNS.Topic.4](#mitigationsnstopic4)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1537](https://attack.mitre.org/techniques/T1537/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Network Perimeter - Resource access from unexpected network](../potential-threats.md#network-perimeter---resource-access-from-unexpected-network)): An IAM principal with permissions on your Amazon SNS topic can publish to or reconfigure it from an unexpected network.

**Data perimeter:** `AddPermission` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Data perimeter:** `CreateTopic` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Data perimeter:** `SetTopicAttributes` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:AddPermission` | `$.AWSAccountId[*]`, `$.ActionName[*]`, `$.TopicArn` |
| `sns:DeleteTopic` | `$.TopicArn` |
| `sns:Publish` | `$.Message`, `$.TopicArn` |
| `sns:PublishBatch` | `$.PublishBatchRequestEntries[*].Message`, `$.TopicArn` |
| `sns:RemovePermission` | `$.Label`, `$.TopicArn` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

### Threat.SNS.Topic.9

Untrusted IAM principal, that has been granted access to my Amazon SNS topic through its resource-based policy, can use my topic.

**Impacts:** [Data Collection](../threat-impacts.md#data-collection), [Data Destruction](../threat-impacts.md#data-destruction), [Data Exfiltration](../threat-impacts.md#data-exfiltration), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1119](https://attack.mitre.org/techniques/T1119/), [T1199](https://attack.mitre.org/techniques/T1199/), [T1485](https://attack.mitre.org/techniques/T1485/), [T1537](https://attack.mitre.org/techniques/T1537/)

**Scoping** (for [Identity Perimeter - Untrusted principal access to my resources](../potential-threats.md#identity-perimeter---untrusted-principal-access-to-my-resources)): An Amazon SNS topic has a resource-based policy that can name principals in other AWS accounts. Depending on the actions granted, an identity outside your organization may subscribe its own endpoint, publish to your subscribers, or delete the topic.

**Data perimeter:** `AddPermission` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Data perimeter:** `CreateTopic` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Data perimeter:** `SetTopicAttributes` allows you to apply a resource-based policy to grant access to a topic. The service currently does not support resource control policies.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:AddPermission` | `$.AWSAccountId[*]`, `$.ActionName[*]`, `$.TopicArn` |
| `sns:DeleteTopic` | `$.TopicArn` |
| `sns:Publish` | `$.Message`, `$.TopicArn` |
| `sns:PublishBatch` | `$.PublishBatchRequestEntries[*].Message`, `$.TopicArn` |
| `sns:RemovePermission` | `$.Label`, `$.TopicArn` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |
| `sns:Subscribe` | `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

### Threat.SNS.Topic.10

Untrusted IAM principal, that has been granted access to my Amazon SNS topic's IAM success feedback role and failure feedback role through their trust policies, can assume my success feedback role and failure feedback role and act with their permissions.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion), [Initial Access](../threat-impacts.md#initial-access)

**Mitigations:** [Mitigation.SNS.Topic.2](#mitigationsnstopic2)

**Frameworks:** MITRE ATT&CK [T1078.004](https://attack.mitre.org/techniques/T1078/004/), [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Identity Perimeter - Untrusted principal assumption of my IAM roles](../potential-threats.md#identity-perimeter---untrusted-principal-assumption-of-my-iam-roles)): You author the trust policies of the success feedback role and failure feedback role that Amazon SNS assumes to write your topic's delivery status logs. An identity outside your organization that either role's trust policy allows to assume the role can act with its CloudWatch Logs permissions.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue` |
| `sts:AssumeRole` | `$.RoleArn` |

### Threat.SNS.Topic.11

IAM principal, with IAM permissions to use an Amazon SNS topic, can publish messages that fan out to subscribed endpoints without a record of those deliveries being persisted by message delivery status logging to Amazon CloudWatch Logs, while their control-plane actions remain recorded by always-on AWS CloudTrail management events.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.SNS.Topic.1](#mitigationsnstopic1)

**Scoping** (for [Defense evasion through unconfigured logging](../potential-threats.md#defense-evasion-through-unconfigured-logging)): When message delivery status logging is not configured on your topic, no delivery records are written to Amazon CloudWatch Logs. An IAM principal with permissions on the topic can publish messages that fan out to subscribed endpoints, leaving no record of those deliveries.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:Publish` | `$.Message`, `$.TopicArn` |
| `sns:PublishBatch` | `$.PublishBatchRequestEntries[*].Message`, `$.TopicArn` |

### Threat.SNS.Topic.12

Untrusted IAM principal, with access to configure an AWS service that has been granted access to an Amazon SNS topic through its resource-based policy, may be able to cause that AWS service to interact with the topic using the principal's data or context.

**Impacts:** [Privilege Escalation](../threat-impacts.md#privilege-escalation)

**Mitigations:** [Mitigation.SNS.Topic.9](#mitigationsnstopic9)

**Frameworks:** MITRE ATT&CK [T1548.005](https://attack.mitre.org/techniques/T1548/005/)

**Scoping** (for [Cross-service confused deputy problem through resource-based policies](../potential-threats.md#cross-service-confused-deputy-problem-through-resource-based-policies)): Your topic's resource-based policy can grant an AWS service permission to publish regardless of which account's resources the service acts on behalf of. An untrusted IAM principal who can configure that service may be able to cause it to publish their content to your topic.

**Request parameters**

| Operation | Parameters |
|---|---|
| `cloudwatch:PutMetricAlarm` | `$.AlarmActions[*]`, `$.AlarmName`, `$.InsufficientDataActions[*]`, `$.OKActions[*]` |
| `eventbridge:PutTargets` | `$.Rule`, `$.Targets[*].Arn` |
| `s3:PutBucketNotificationConfiguration` | `$.Bucket`, `$.NotificationConfiguration.TopicConfiguration[*].Topic` |
| `ses:SetIdentityNotificationTopic` | `$.Identity`, `$.SnsTopic` |
| `sns:CreateTopic` | `$.Attributes{key}` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

### Threat.SNS.Topic.13

IAM principal, with IAM permissions to modify an Amazon SNS topic's delivery status logging, can disable message delivery status logging to CloudWatch Logs for the topic.

**Impacts:** [Defense Evasion](../threat-impacts.md#defense-evasion)

**Mitigations:** [Mitigation.SNS.Topic.1](#mitigationsnstopic1)

**Frameworks:** MITRE ATT&CK [T1562.008](https://attack.mitre.org/techniques/T1562/008/)

**Scoping** (for [Logging disablement](../potential-threats.md#logging-disablement)): An IAM principal with `sns:SetTopicAttributes` permission on your topic can clear the feedback role ARN attributes that turn on delivery status logging. Clearing them stops new per-protocol delivery outcome records from reaching CloudWatch Logs; setting a protocol's `SuccessFeedbackSampleRate` attribute to 0 stops that protocol's successful delivery records while its failure records continue.

**Request parameters**

| Operation | Parameters |
|---|---|
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

### Threat.SNS.Topic.14

Threat actor, with network adjacency to the connection path between clients and an Amazon SNS topic that has not been configured to encrypt data in transit, can capture topic network traffic and read data in transit that has not been encrypted.

**Impacts:** [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery), [Data Collection](../threat-impacts.md#data-collection)

**Mitigations:** [Mitigation.SNS.Topic.10](#mitigationsnstopic10)

**Frameworks:** MITRE ATT&CK [T1040](https://attack.mitre.org/techniques/T1040/)

**Protocols:** `HTTP`

**Scoping** (for [Data in transit interception through unencrypted client connection](../potential-threats.md#data-in-transit-interception-through-unencrypted-client-connection)): A topic can be reached over `HTTP`, and does not automatically reject requests that are not encrypted. A threat actor on the network path between your clients and the topic can capture that traffic and read the data in transit.

---

## Mitigations

Each scoping statement below names the capability the mitigation goal governs on the entity and the objective of governing it. Each mitigation's implementations describe how to govern or monitor that capability.

### Mitigation.SNS.1

Amazon SNS service API calls on a governed VPC network should only be made against trusted resources or AWS service-owned resources

**Goal:** [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)

**Addresses:** [Threat.SNS.2](#threatsns2)

**Implemented by:** [Implementation.SNS.1](#implementationsns1)

**Scoping** (for [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network)): You can call the Amazon SNS API from your VPC through an interface VPC endpoint (powered by AWS PrivateLink). IAM principals in your VPC can use that path to reach Amazon SNS resources outside your organization. Restricting those calls to trusted resources or AWS service-owned resources helps keep access from your networks within your resource perimeter.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.SNS.2

Amazon SNS service API calls on a governed VPC network should only be made by trusted identities or AWS services

**Goal:** [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)

**Addresses:** [Threat.SNS.1](#threatsns1)

**Implemented by:** [Implementation.SNS.2](#implementationsns2)

**Scoping** (for [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network)): Amazon SNS can be reached from your VPC through an interface VPC endpoint (powered by AWS PrivateLink). Calls over that network path can be made by IAM principals from any account, including identities outside your organization. Restricting calls over that network path to trusted identities helps keep network access within your identity perimeter.

**Configure via**

| Operation | Parameters |
|---|---|
| `ec2:CreateVpcEndpoint` | `$.PolicyDocument` |
| `ec2:ModifyVpcEndpoint` | `$.PolicyDocument`, `$.VpcEndpointId` |

**Verify via**

| Operation | Parameters |
|---|---|
| `ec2:DescribeVpcEndpoints` | `$.VpcEndpointId` |

### Mitigation.SNS.PlatformApplication.1

An Amazon SNS platform application should be configured to send delivery status logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.SNS.PlatformApplication.2](#threatsnsplatformapplication2), [Threat.SNS.PlatformApplication.3](#threatsnsplatformapplication3)

**Implemented by:** [Implementation.SNS.PlatformApplication.1](#implementationsnsplatformapplication1), [Implementation.SNS.PlatformApplication.2](#implementationsnsplatformapplication2)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): Amazon SNS can record the delivery status of push notifications sent to your platform application's endpoints. Amazon SNS sends these per-message records to CloudWatch Logs as an audit trail of delivery attempts.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreatePlatformApplication` | `$.Attributes{key}`, `$.Name` |
| `sns:SetPlatformApplicationAttributes` | `$.Attributes{key}`, `$.PlatformApplicationArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetPlatformApplicationAttributes` | `$.PlatformApplicationArn` |
| `sns:ListPlatformApplications` | *(any parameter)* |

### Mitigation.SNS.PlatformApplication.2

An Amazon SNS platform application's success feedback role and failure feedback role should be assumed only by Amazon SNS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.SNS.PlatformApplication.1](#threatsnsplatformapplication1)

**Implemented by:** [Implementation.SNS.PlatformApplication.3](#implementationsnsplatformapplication3), [Implementation.SNS.PlatformApplication.4](#implementationsnsplatformapplication4), [Implementation.SNS.PlatformApplication.5](#implementationsnsplatformapplication5)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): An Amazon SNS platform application can reference a customer-managed IAM service role that Amazon SNS assumes to write push delivery status logs. Because you create and maintain that role, its trust policy determines who can assume it.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreatePlatformApplication` | `$.Attributes{key}`, `$.Name` |
| `sns:SetPlatformApplicationAttributes` | `$.Attributes{key}`, `$.PlatformApplicationArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetPlatformApplicationAttributes` | `$.PlatformApplicationArn` |

### Mitigation.SNS.SmsAttribute.1

An Amazon SNS SMS attribute should be configured to send SMS delivery status logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.SNS.SmsAttribute.2](#threatsnssmsattribute2), [Threat.SNS.SmsAttribute.3](#threatsnssmsattribute3)

**Implemented by:** [Implementation.SNS.SmsAttribute.1](#implementationsnssmsattribute1), [Implementation.SNS.SmsAttribute.2](#implementationsnssmsattribute2)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): Your account-level SMS settings include a delivery status logging configuration. When you set it, Amazon SNS can write SMS delivery status logs to CloudWatch Logs.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:SetSMSAttributes` | `$.attributes{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetSMSAttributes` | `$.attributes[*]` |

### Mitigation.SNS.SmsAttribute.2

An Amazon SNS SMS attribute delivery status IAM service role should be assumed only by Amazon SNS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.SNS.SmsAttribute.1](#threatsnssmsattribute1)

**Implemented by:** [Implementation.SNS.SmsAttribute.3](#implementationsnssmsattribute3), [Implementation.SNS.SmsAttribute.4](#implementationsnssmsattribute4), [Implementation.SNS.SmsAttribute.5](#implementationsnssmsattribute5)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): Your Amazon SNS SMS attribute settings designate an IAM service role through the `$.attributes{key}` request parameter. Amazon SNS assumes that role to write SMS delivery status logs.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:SetSMSAttributes` | `$.attributes{key}` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetSMSAttributes` | `$.attributes[*]` |

### Mitigation.SNS.Subscription.1

An Amazon SNS subscription IAM service role should be assumed only by Amazon SNS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.SNS.Subscription.3](#threatsnssubscription3)

**Implemented by:** [Implementation.SNS.Subscription.1](#implementationsnssubscription1), [Implementation.SNS.Subscription.2](#implementationsnssubscription2), [Implementation.SNS.Subscription.3](#implementationsnssubscription3), [Implementation.SNS.Subscription.4](#implementationsnssubscription4)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): An Amazon SNS subscription for an Amazon Data Firehose delivery stream names a customer-managed IAM service role through `$.Attributes{key}` for Amazon SNS to assume. Because you create and modify that role's trust policy, restricting who can assume it helps limit the role's use to its intended delivery purpose.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:SetSubscriptionAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.SubscriptionArn` |
| `sns:Subscribe` | `$.Attributes{key}`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetSubscriptionAttributes` | `$.SubscriptionArn` |

### Mitigation.SNS.Subscription.2

An Amazon SNS subscription should not allow unauthenticated access to its unsubscribe action

**Goal:** [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention)

**Addresses:** [Threat.SNS.Subscription.2](#threatsnssubscription2)

**Implemented by:** [Implementation.SNS.Subscription.5](#implementationsnssubscription5)

**Scoping** (for [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention)): Amazon SNS supports an unauthenticated invocation mode in which `Unsubscribe` against a subscription succeeds without AWS credentials. That authentication mode is a property of the subscription, established when the subscription is confirmed.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:ConfirmSubscription` | `$.AuthenticateOnUnsubscribe`, `$.Token`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetSubscriptionAttributes` | `$.SubscriptionArn` |
| `sns:ListSubscriptionsByTopic` | `$.TopicArn` |

### Mitigation.SNS.Subscription.3

An Amazon SNS subscription should only be configured with a trusted delivery endpoint: an Amazon SQS queue, AWS Lambda function, Amazon Data Firehose delivery stream, mobile push platform endpoint, HTTP/S URL, email address, or SMS phone number

**Goal:** [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets)

**Addresses:** [Threat.SNS.Subscription.1](#threatsnssubscription1)

**Implemented by:** [Implementation.SNS.Subscription.6](#implementationsnssubscription6), [Implementation.SNS.Subscription.7](#implementationsnssubscription7), [Implementation.SNS.Subscription.8](#implementationsnssubscription8), [Implementation.SNS.Subscription.9](#implementationsnssubscription9)

**Scoping** (for [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets)): Creating a subscription persists a delivery destination in `$.Endpoint`, which can point at a resource in any account or at an endpoint outside AWS. Restricting the destination at subscribe time helps keep messages published to your topic flowing only to trusted targets.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:Subscribe` | `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetSubscriptionAttributes` | `$.SubscriptionArn` |
| `sns:ListSubscriptions` | *(any parameter)* |
| `sns:ListSubscriptionsByTopic` | `$.TopicArn` |

### Mitigation.SNS.Subscription.4

An Amazon SNS subscription should use the HTTPS delivery protocol to require encryption of data in transit

**Goal:** [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature)

**Addresses:** [Threat.SNS.Subscription.4](#threatsnssubscription4)

**Implemented by:** [Implementation.SNS.Subscription.10](#implementationsnssubscription10), [Implementation.SNS.Subscription.11](#implementationsnssubscription11), [Implementation.SNS.Subscription.12](#implementationsnssubscription12)

**Scoping** (for [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature)): When you subscribe an endpoint, `$.Protocol` sets whether Amazon SNS delivers notifications over cleartext HTTP or encrypted HTTPS. Choosing `https` helps protect message payloads in transit, because Amazon SNS does not fall back to a cleartext connection.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:Subscribe` | `$.Endpoint`, `$.Protocol`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetSubscriptionAttributes` | `$.SubscriptionArn` |
| `sns:ListSubscriptions` | *(any parameter)* |
| `sns:ListSubscriptionsByTopic` | `$.TopicArn` |

### Mitigation.SNS.Topic.1

An Amazon SNS topic should be configured to send message delivery status logs to CloudWatch Logs

**Goal:** [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)

**Addresses:** [Threat.SNS.Topic.11](#threatsnstopic11), [Threat.SNS.Topic.13](#threatsnstopic13)

**Implemented by:** [Implementation.SNS.Topic.1](#implementationsnstopic1), [Implementation.SNS.Topic.2](#implementationsnstopic2), [Implementation.SNS.Topic.3](#implementationsnstopic3), [ManagedControl.SNS.Topic.1](#managedcontrolsnstopic1)

**Scoping** (for [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery)): Your topic can send message delivery status logs to CloudWatch Logs, recording the outcome of each message delivery to a subscribed endpoint. You enable this logging per protocol through the topic's own success and failure feedback attributes.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |

### Mitigation.SNS.Topic.2

An Amazon SNS topic's success feedback role and failure feedback role should be assumed only by Amazon SNS acting on behalf of trusted resources

**Goal:** [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)

**Addresses:** [Threat.SNS.Topic.10](#threatsnstopic10)

**Implemented by:** [Implementation.SNS.Topic.4](#implementationsnstopic4), [Implementation.SNS.Topic.5](#implementationsnstopic5), [Implementation.SNS.Topic.6](#implementationsnstopic6), [Implementation.SNS.Topic.7](#implementationsnstopic7)

**Scoping** (for [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role)): Your topic can be configured with a customer-managed IAM service role that Amazon SNS assumes to write delivery status logs to Amazon CloudWatch Logs. Because you create and maintain this role, its trust policy determines which identities and AWS services can assume it.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |

### Mitigation.SNS.Topic.3

An Amazon SNS topic should have encryption at rest enabled

**Goal:** [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest)

**Addresses:** [Threat.SNS.Topic.5](#threatsnstopic5)

**Implemented by:** [Implementation.SNS.Topic.8](#implementationsnstopic8), [Implementation.SNS.Topic.9](#implementationsnstopic9), [ManagedControl.SNS.Topic.2](#managedcontrolsnstopic2)

**Scoping** (for [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest)): Amazon SNS stores messages using disk encryption by default. Server-side encryption under an AWS KMS key is optional, applies only after you enable it, and does not cover messages published earlier.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |

### Mitigation.SNS.Topic.4

An Amazon SNS topic should be accessed only from expected networks

**Goal:** [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)

**Addresses:** [Threat.SNS.Topic.6](#threatsnstopic6), [Threat.SNS.Topic.8](#threatsnstopic8)

**Implemented by:** [Implementation.SNS.Topic.10](#implementationsnstopic10), [Implementation.SNS.Topic.11](#implementationsnstopic11), [Implementation.SNS.Topic.12](#implementationsnstopic12), [Implementation.SNS.Topic.13](#implementationsnstopic13), [Implementation.SNS.Topic.14](#implementationsnstopic14)

**Scoping** (for [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks)): You can apply a resource-based policy to your Amazon SNS topic to grant access to the topic. That policy can limit the networks that requests to your topic originate from.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |

### Mitigation.SNS.Topic.5

IAM principals should be restricted to accessing only trusted Amazon SNS topic resources

**Goal:** [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)

**Addresses:** [Threat.SNS.Topic.7](#threatsnstopic7)

**Implemented by:** [Implementation.SNS.Topic.15](#implementationsnstopic15)

**Scoping** (for [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources)): Amazon SNS API calls can specify a topic that belongs to an account outside your organization. Restricting the topics your principals can access helps keep data access within trusted accounts.

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

### Mitigation.SNS.Topic.6

An Amazon SNS topic should be encrypted with an AWS KMS customer managed key

**Goal:** [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)

**Addresses:** [Threat.SNS.Topic.1](#threatsnstopic1), [Threat.SNS.Topic.3](#threatsnstopic3), [Threat.SNS.Topic.5](#threatsnstopic5)

**Implemented by:** [Implementation.SNS.Topic.16](#implementationsnstopic16), [Implementation.SNS.Topic.17](#implementationsnstopic17), [ManagedControl.SNS.Topic.2](#managedcontrolsnstopic2)

**Scoping** (for [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption)): Amazon SNS supports server-side encryption of a topic's message bodies with a customer managed key, set through the `KmsMasterKeyId` topic attribute. A customer managed key gives you lifecycle control, an audit trail of its use, and the ability to make stored messages unavailable.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |

### Mitigation.SNS.Topic.7

An Amazon SNS topic's access policy should limit permissions to a subset of IAM principals, rather than granting access to all IAM principals

**Goal:** [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)

**Addresses:** [Threat.SNS.Topic.4](#threatsnstopic4)

**Implemented by:** [Implementation.SNS.Topic.18](#implementationsnstopic18), [Implementation.SNS.Topic.19](#implementationsnstopic19), [Implementation.SNS.Topic.20](#implementationsnstopic20), [Implementation.SNS.Topic.21](#implementationsnstopic21), [ManagedControl.SNS.Topic.3](#managedcontrolsnstopic3)

**Scoping** (for [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition)): A resource-based policy attached to your Amazon SNS topic controls which principals can publish to the topic and subscribe to it. That policy can specify a wildcard principal, which would extend those permissions to all IAM principals.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:AddPermission` | `$.AWSAccountId[*]`, `$.TopicArn` |
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:RemovePermission` | `$.Label`, `$.TopicArn` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |

### Mitigation.SNS.Topic.8

An Amazon SNS topic should be accessed only by trusted identities or AWS services acting on behalf of trusted resources

**Goal:** [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)

**Addresses:** [Threat.SNS.Topic.6](#threatsnstopic6), [Threat.SNS.Topic.9](#threatsnstopic9)

**Implemented by:** [Implementation.SNS.Topic.22](#implementationsnstopic22), [Implementation.SNS.Topic.23](#implementationsnstopic23), [Implementation.SNS.Topic.24](#implementationsnstopic24), [Implementation.SNS.Topic.25](#implementationsnstopic25), [Implementation.SNS.Topic.26](#implementationsnstopic26), [Implementation.SNS.Topic.27](#implementationsnstopic27), [Implementation.SNS.Topic.28](#implementationsnstopic28), [ManagedControl.SNS.Topic.3](#managedcontrolsnstopic3)

**Scoping** (for [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources)): An Amazon SNS topic supports a resource-based policy that can grant principals outside your organization permission to perform `sns:Publish` and `sns:Subscribe`. Restricting those grants to principals in your organization helps keep access to your topic within your identity perimeter.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:AddPermission` | `$.AWSAccountId[*]`, `$.TopicArn` |
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:RemovePermission` | `$.Label`, `$.TopicArn` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |
| `sns:ListTopics` | *(any parameter)* |

### Mitigation.SNS.Topic.9

An Amazon SNS topic should be accessed by an AWS service principal, such as the Amazon S3 service principal, only when the service principal is acting on behalf of a trusted resource, AWS account, or organization

**Goal:** [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies)

**Addresses:** [Threat.SNS.Topic.12](#threatsnstopic12)

**Implemented by:** [Implementation.SNS.Topic.29](#implementationsnstopic29), [Implementation.SNS.Topic.30](#implementationsnstopic30), [Implementation.SNS.Topic.31](#implementationsnstopic31), [Implementation.SNS.Topic.32](#implementationsnstopic32)

**Scoping** (for [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies)): An Amazon SNS topic has a resource-based policy that can grant the `sns:Publish` action to AWS services acting on your behalf. You can scope those grants so they apply only to requests made on behalf of resources and accounts you trust.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |

### Mitigation.SNS.Topic.10

An Amazon SNS topic should be governed by a policy that requires all connections to be encrypted in transit

**Goal:** [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy)

**Addresses:** [Threat.SNS.Topic.14](#threatsnstopic14)

**Implemented by:** [Implementation.SNS.Topic.11](#implementationsnstopic11), [Implementation.SNS.Topic.12](#implementationsnstopic12), [Implementation.SNS.Topic.33](#implementationsnstopic33), [Implementation.SNS.Topic.34](#implementationsnstopic34), [Implementation.SNS.Topic.35](#implementationsnstopic35)

**Scoping** (for [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy)): The Amazon SNS API endpoint that serves your topic accepts requests over both HTTP and HTTPS. Your topic does not automatically reject requests that arrive over HTTP.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Attributes{key}`, `$.Name` |
| `sns:SetTopicAttributes` | `$.AttributeName`, `$.AttributeValue`, `$.TopicArn` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:GetTopicAttributes` | `$.TopicArn` |

### Mitigation.SNS.Topic.11

An Amazon SNS topic should be tagged with information that describes the owner and purpose

**Goal:** [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)

**Addresses:** [Threat.SNS.Topic.2](#threatsnstopic2)

**Implemented by:** [Implementation.SNS.Topic.36](#implementationsnstopic36), [Implementation.SNS.Topic.37](#implementationsnstopic37), [Implementation.SNS.Topic.38](#implementationsnstopic38)

**Scoping** (for [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose)): An Amazon SNS topic is a durable resource you can tag by its ARN, at creation (`$.Tags[*].Key`, `$.Tags[*].Value`) or later through `TagResource`. Recording owner and purpose tags helps you attribute each topic to a team and a workload.

**Configure via**

| Operation | Parameters |
|---|---|
| `sns:CreateTopic` | `$.Name`, `$.Tags[*].Key`, `$.Tags[*].Value` |
| `sns:TagResource` | `$.ResourceArn`, `$.Tags[*].Key`, `$.Tags[*].Value` |

**Verify via**

| Operation | Parameters |
|---|---|
| `sns:ListTagsForResource` | `$.ResourceArn` |

---

## Implementations

### Implementation.SNS.1

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.SNS.1](#mitigationsns1)

**Scoping** (for [Mitigation.SNS.1](#mitigationsns1)): This VPC endpoint policy restricts Amazon SNS API calls with the `aws:ResourceOrgID` condition key, exempting AWS services accessing service-owned resources on your behalf (`aws:PrincipalIsAWSService`). Requests through your interface VPC endpoint (powered by AWS PrivateLink) to Amazon SNS resources outside your organization are denied.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.SNS.2

*Resource policy — Preventive*

Attaches to `AWS::EC2::VPCEndpoint` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:PrincipalIsAWSService` |
| `*` | `*` | `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |

**Implements:** [Mitigation.SNS.2](#mitigationsns2)

**Scoping** (for [Mitigation.SNS.2](#mitigationsns2)): This implementation governs Amazon SNS API calls through your VPC endpoint with the `aws:PrincipalOrgID` condition key, exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`). As a result, identities outside your organization are restricted from accessing Amazon SNS through that network.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Use `aws:PrincipalIsAWSService` to allow AWS services to use their service principals to send requests from your network. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [VPC endpoint policy sample: default data perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/vpc_endpoint_policies/default_endpoint_policy.json)

### Implementation.SNS.PlatformApplication.1

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`, `$.logStreamName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `sns:CreatePlatformApplication` — `$.Attributes{key}`, `$.Name`
- `sns:DeletePlatformApplication` — `$.PlatformApplicationArn`
- `sns:SetPlatformApplicationAttributes` — `$.Attributes{key}`, `$.PlatformApplicationArn`

**Implements:** [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1)

**Scoping** (for [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1)): This implementation records `SetPlatformApplicationAttributes` calls carrying `$.Attributes{key}`, the map that holds the delivery status logging attributes for your platform application. Monitoring that operation alongside `DeletePlatformApplication` helps you detect changes that disable this logging. The map also carries `SuccessFeedbackSampleRate`, so a change that sets it to 0 and silences successful delivery records is recorded too.

### Implementation.SNS.PlatformApplication.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:ListPlatformApplications` with `$.NextToken`; check `$.PlatformApplications[*].PlatformApplicationArn`, `$.NextToken`
2. Call `sns:GetPlatformApplicationAttributes` with `$.PlatformApplicationArn`; check `$.Attributes.FailureFeedbackRoleArn`, `$.Attributes.SuccessFeedbackRoleArn`, `$.Attributes.SuccessFeedbackSampleRate`

**Implements:** [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1)

**Scoping** (for [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1)): This periodic check reads each platform application's attributes and flags applications where `$.Attributes.FailureFeedbackRoleArn` and `$.Attributes.SuccessFeedbackRoleArn` are absent. Flagging those gaps helps you detect applications whose push delivery outcomes, including failures and provider errors, are not recorded in CloudWatch Logs. It does not flag a `$.Attributes.SuccessFeedbackSampleRate` of 0, which stops successful delivery records while the feedback roles remain configured.

### Implementation.SNS.PlatformApplication.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:ListPlatformApplications`; check `$.PlatformApplications[*].PlatformApplicationArn`
2. Call `sns:GetPlatformApplicationAttributes` with `$.PlatformApplicationArn`; check `$.Attributes{key}`
3. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
5. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2)

**Scoping** (for [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2)): This implementation reads the delivery status logging role recorded in your platform application's `$.Attributes{key}` values and evaluates that role's trust policy in `$.Role.AssumeRolePolicyDocument`. Flagging a trust policy that allows public access, or grants assumption beyond your reference policy, helps you review who can assume the role.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.PlatformApplication.4

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `sns.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2)

**Scoping** (for [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2)): This implementation guidance recommends a trust policy that restricts assumption of your platform application's delivery status logging role to the `sns.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.SNS.PlatformApplication.5

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2)

**Scoping** (for [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2)): This implementation guidance recommends applying the `aws:PrincipalOrgID` condition key in a resource control policy (RCP) to restrict `sts:AssumeRole`, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. As a result, principals outside your organization are restricted from assuming your delivery status logging roles.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.SNS.SmsAttribute.1

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName` = `sns/<region>/<account-id>/DirectPublishToPhoneNumber`, `$.logGroupName` = `sns/<region>/<account-id>/DirectPublishToPhoneNumber/Failure`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName` = `sns/<region>/<account-id>/DirectPublishToPhoneNumber`, `$.logGroupName` = `sns/<region>/<account-id>/DirectPublishToPhoneNumber/Failure`, `$.retentionInDays`
- `sns:SetSMSAttributes` — `$.attributes{key}` = `DeliveryStatusIAMRole`, `$.attributes{key}` = `DeliveryStatusSuccessSamplingRate`

**Implements:** [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1)

**Scoping** (for [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1)): This implementation records `SetSMSAttributes` calls that write the SMS delivery status logging settings in `$.attributes{key}`. Flagging those calls helps you detect changes that disable delivery status logging to CloudWatch Logs or lower the success sample rate.

### Implementation.SNS.SmsAttribute.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:GetSMSAttributes`; check `$.attributes.DeliveryStatusIAMRole`, `$.attributes.DeliveryStatusSuccessSamplingRate`

**Implements:** [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1)

**Scoping** (for [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1)): This implementation reads `$.attributes.DeliveryStatusIAMRole` and `$.attributes.DeliveryStatusSuccessSamplingRate` from your account's SMS settings. Flagging accounts without a delivery status logging role helps you detect where SMS delivery outcomes and failure reasons go unrecorded.

### Implementation.SNS.SmsAttribute.3

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:GetSMSAttributes`; check `$.attributes{key}`
2. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
3. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
4. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2)

**Scoping** (for [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2)): This implementation guidance recommends a periodic AWS Config rule that reads the `DeliveryStatusIAMRole` setting from your account's text messaging preferences in `$.attributes{key}`. Flagging that role's trust policy, `$.Role.AssumeRolePolicyDocument`, helps you detect grants that can allow public role assumption or exceed your reference policy.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.SmsAttribute.4

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `sns.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2)

**Scoping** (for [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2)): This implementation guidance recommends a trust policy that restricts assumption of your SMS delivery status role to the `sns.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.SNS.SmsAttribute.5

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2)

**Scoping** (for [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2)): This implementation guidance recommends restricting `sts:AssumeRole` with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent identities outside your organization from assuming your `DeliveryStatusIAMRole` delivery status logging role.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.SNS.Subscription.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::Subscription`: `$.SubscriptionRoleArn`

**Implements:** [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)

**Scoping** (for [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)): This implementation guidance recommends validating the trust policy of `$.SubscriptionRoleArn`, the subscription role that Amazon SNS assumes, before your subscription is created. This helps prevent the subscription from being provisioned when that trust policy can grant public access, or role assumption beyond the reference policy you maintain.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.Subscription.2

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:ListSubscriptions`; check `$.Subscriptions[*].SubscriptionArn`, `$.Subscriptions[*].Protocol`
2. Call `sns:GetSubscriptionAttributes` with `$.SubscriptionArn`; check `$.Attributes.SubscriptionRoleArn`
3. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result`
5. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result`

**Implements:** [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)

**Scoping** (for [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)): This implementation guidance recommends reading the subscription role ARN at `$.Attributes.SubscriptionRoleArn`, then submitting that role's trust policy at `$.Role.AssumeRolePolicyDocument` to IAM Access Analyzer. Flagging trust policies that may allow public role assumption, or assumption by principals other than Amazon SNS (`sns.amazonaws.com`), helps you detect unintended access.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.Subscription.3

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `sns.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)

**Scoping** (for [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)): This implementation guidance recommends a trust policy that restricts `sts:AssumeRole` on your Amazon SNS subscription role to the `sns.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.SNS.Subscription.4

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)

**Scoping** (for [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1)): This implementation guidance recommends applying the `aws:PrincipalOrgID` condition key in a resource control policy (RCP) to govern `sts:AssumeRole`, while exempting AWS services acting on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. As a result, identities outside your organization are restricted from assuming your Amazon SNS subscription IAM service role.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.SNS.Subscription.5

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:ListSubscriptions`; check `$.Subscriptions[*].SubscriptionArn`
2. Call `sns:GetSubscriptionAttributes` with `$.SubscriptionArn`; check `$.Attributes.ConfirmationWasAuthenticated`

**Implements:** [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2)

**Scoping** (for [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2)): This implementation reads `$.Attributes.ConfirmationWasAuthenticated` for each subscription that `ListSubscriptions` returns. Flagging subscriptions confirmed without authentication helps you detect endpoints that unauthenticated callers can unsubscribe.

### Implementation.SNS.Subscription.6

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::Subscription`: `$.Endpoint`, `$.Protocol`

**Implements:** [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)

**Scoping** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): This implementation validates the delivery endpoint (`$.Endpoint`) and protocol (`$.Protocol`) that a subscription declares at create time. Subscriptions naming a delivery endpoint or protocol outside the set you designate are not created.

**Data perimeter** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): The `Endpoint` and `Protocol` request parameters of the `Subscribe` action name the delivery target for every subscription. Validating both values before creation addresses the same exposure as reviewing them after the fact in CloudTrail management events.
- [Subscribe request parameters, including Endpoint and Protocol](https://docs.aws.amazon.com/sns/latest/api/API_Subscribe.html#API_Subscribe_RequestParameters)
- [Amazon SNS API Reference: Subscribe](https://docs.aws.amazon.com/sns/latest/api/API_Subscribe.html)

**Data perimeter** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): You can use a hook on `AWS::SNS::Subscription` to check the `$.Endpoint` and `$.Protocol` values a developer declares. Declarations naming a delivery target outside your organization are rejected before the subscription is provisioned.
- [AWS::SNS::Subscription resource type reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-subscription.html)
- [Endpoint property of AWS::SNS::Subscription](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-subscription.html#cfn-sns-subscription-endpoint)
- [Protocol property of AWS::SNS::Subscription](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-subscription.html#cfn-sns-subscription-protocol)

### Implementation.SNS.Subscription.7

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `sns:Subscribe` — `$.Endpoint`, `$.Protocol`

**Implements:** [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)

**Scoping** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): This implementation records the `$.Endpoint` and `$.Protocol` values passed to `Subscribe`, together with the principal that made the call. Flagging endpoints outside your organization helps you detect message fanout to a delivery target you do not control.

**Data perimeter** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): Monitoring `Subscribe` calls across your environment gives you a record of every delivery target requested for your topics. Review the `$.Endpoint` and `$.Protocol` values carried in those events. You can then apply the responsive controls of your choice when a target resolves outside your organization.
- [Subscribe request parameters: Endpoint and Protocol](https://docs.aws.amazon.com/sns/latest/api/API_Subscribe.html#API_Subscribe_RequestParameters)
- [Amazon SNS API Reference: Subscribe](https://docs.aws.amazon.com/sns/latest/api/API_Subscribe.html)

### Implementation.SNS.Subscription.8

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:ListSubscriptions`; check `$.Subscriptions[*].SubscriptionArn`, `$.Subscriptions[*].TopicArn`, `$.Subscriptions[*].Protocol`, `$.Subscriptions[*].Endpoint`, `$.Subscriptions[*].Owner`
2. Call `organizations:ListAccounts`; check `$.Accounts[*].Id`

**Implements:** [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)

**Scoping** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): This implementation enumerates your subscriptions and reads `$.Subscriptions[*].Protocol` and `$.Subscriptions[*].Endpoint`, the delivery protocol and target recorded on each subscription. Flagging a subscription endpoint ARN whose account is absent from your organization's account list (`$.Accounts[*].Id`) helps you detect delivery to endpoints outside your organization.

### Implementation.SNS.Subscription.9

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:Subscribe` | `*` | `aws:PrincipalTag/dp:exclude:resource`, `sns:Endpoint`, `sns:Protocol` |

**Implements:** [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)

**Scoping** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): This service control policy (SCP) restricts `sns:Subscribe` with the `sns:Endpoint` and `sns:Protocol` condition keys, which carry the `$.Endpoint` and `$.Protocol` request parameters. `Subscribe` requests naming an endpoint you have not designated as trusted are denied, so principals in your organization cannot create those subscriptions.

**Data perimeter** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): Amazon SNS exposes `sns:Endpoint` and `sns:Protocol` as request condition keys on `sns:Subscribe`. Pairing them lets you designate which protocols and delivery targets are trusted for your topics. The linked example policy shows this statement for email subscriptions. You can extend the same shape to the `sqs`, `lambda`, `firehose`, `application`, `http`, `https`, and `sms` protocols. `aws:PrincipalTag/dp:exclude:resource` exempts the principals you designate to configure destinations outside your organization, following the tagging convention of the data perimeter policy examples.
- [Example SCP: restrict untrusted endpoints](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_untrusted_endpoints_scp.json)

**Data perimeter** (for [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3)): Subscriptions declared in a CloudFormation template carry their destination in the `Endpoint` and `Protocol` properties of `AWS::SNS::Subscription`. CloudFormation provisions them by calling `Subscribe`. The same condition keys therefore apply to stack deployments. The endpoint value's format depends on the protocol, so evaluate both properties together when you define your trusted set.
- [AWS::SNS::Subscription resource reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-subscription.html)
- [AWS::SNS::Subscription Endpoint property](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-subscription.html#cfn-sns-subscription-endpoint)
- [AWS::SNS::Subscription Protocol property](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-subscription.html#cfn-sns-subscription-protocol)

### Implementation.SNS.Subscription.10

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::Subscription`: `$.Protocol`, `$.Endpoint`

**Implements:** [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4)

**Scoping** (for [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4)): This implementation validates `$.Protocol` and `$.Endpoint` at create time, requiring the `https` delivery protocol instead of `http`. Requiring HTTPS delivery helps protect message payloads in transit, because messages sent to an `http` endpoint travel unencrypted and can be observed or altered.

### Implementation.SNS.Subscription.11

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:ListSubscriptions` with `$.NextToken`; check `$.Subscriptions[*].SubscriptionArn`, `$.Subscriptions[*].TopicArn`, `$.Subscriptions[*].Protocol`, `$.Subscriptions[*].Endpoint`
2. Call `sns:GetSubscriptionAttributes` with `$.SubscriptionArn`; check `$.Attributes{key}`

**Implements:** [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4)

**Scoping** (for [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4)): This rule reads the delivery protocol in `$.Subscriptions[*].Protocol` and, for the `http` and `https` subscriptions it governs, the endpoint URL in `$.Subscriptions[*].Endpoint`. Flagging subscriptions that deliver over `http` helps you detect notification traffic that travels to its endpoint without TLS protection.

### Implementation.SNS.Subscription.12

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:Subscribe` | `*` | `sns:Protocol` |

**Implements:** [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4)

**Scoping** (for [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4)): This implementation restricts `sns:Subscribe` with the `sns:Protocol` condition key, so requests using the cleartext `http` delivery protocol are denied. HTTP endpoint subscriptions must instead use the `https` protocol, while the other delivery protocols are unaffected.

### Implementation.SNS.Topic.1

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::Topic`: `$.DeliveryStatusLogging[*].Protocol`, `$.DeliveryStatusLogging[*].SuccessFeedbackRoleArn`, `$.DeliveryStatusLogging[*].SuccessFeedbackSampleRate`, `$.DeliveryStatusLogging[*].FailureFeedbackRoleArn`

**Implements:** [Mitigation.SNS.Topic.1](#mitigationsnstopic1)

**Scoping** (for [Mitigation.SNS.Topic.1](#mitigationsnstopic1)): This implementation checks that a topic declares a feedback role in `$.DeliveryStatusLogging[*].FailureFeedbackRoleArn` or `$.DeliveryStatusLogging[*].SuccessFeedbackRoleArn`, so you can review message delivery status in CloudWatch Logs. A declared failure feedback role authorizes Amazon SNS to publish records for failed deliveries. A declared success feedback role authorizes records for successful deliveries, at the success sample rate you set in `$.DeliveryStatusLogging[*].SuccessFeedbackSampleRate`.

### Implementation.SNS.Topic.2

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `cloudwatch-logs:DeleteLogGroup` — `$.logGroupName`
- `cloudwatch-logs:DeleteLogStream` — `$.logGroupName`, `$.logStreamName`
- `cloudwatch-logs:PutRetentionPolicy` — `$.logGroupName`, `$.retentionInDays`
- `sns:CreateTopic` — `$.Attributes{key}`, `$.Name`
- `sns:SetTopicAttributes` — `$.AttributeName`, `$.AttributeValue`, `$.TopicArn`

**Implements:** [Mitigation.SNS.Topic.1](#mitigationsnstopic1)

**Scoping** (for [Mitigation.SNS.Topic.1](#mitigationsnstopic1)): This implementation records `SetTopicAttributes` calls, where `$.AttributeName` and `$.AttributeValue` identify the delivery status logging attribute set on your topic. It also records `CreateTopic`, so you can detect topics created without this logging and changes that turn it off. Calls that set a protocol's `SuccessFeedbackSampleRate` attribute to 0, silencing its successful delivery records, are recorded the same way.

### Implementation.SNS.Topic.3

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::SNS::Topic`: `$.configuration.HTTPSuccessFeedbackRoleArn`, `$.configuration.HTTPFailureFeedbackRoleArn`, `$.configuration.HTTPSuccessFeedbackSampleRate`, `$.configuration.FirehoseSuccessFeedbackRoleArn`, `$.configuration.FirehoseFailureFeedbackRoleArn`, `$.configuration.FirehoseSuccessFeedbackSampleRate`, `$.configuration.LambdaSuccessFeedbackRoleArn`, `$.configuration.LambdaFailureFeedbackRoleArn`, `$.configuration.LambdaSuccessFeedbackSampleRate`, `$.configuration.SQSSuccessFeedbackRoleArn`, `$.configuration.SQSFailureFeedbackRoleArn`, `$.configuration.SQSSuccessFeedbackSampleRate`, `$.configuration.ApplicationSuccessFeedbackRoleArn`, `$.configuration.ApplicationFailureFeedbackRoleArn`, `$.configuration.ApplicationSuccessFeedbackSampleRate`

**Implements:** [Mitigation.SNS.Topic.1](#mitigationsnstopic1)

**Scoping** (for [Mitigation.SNS.Topic.1](#mitigationsnstopic1)): This implementation inspects the delivery status logging attributes on your topic, such as `$.configuration.LambdaFailureFeedbackRoleArn` and `$.configuration.SQSSuccessFeedbackRoleArn`. Flagging topics that have no feedback role configured helps you find where message delivery outcomes are not being recorded in CloudWatch Logs. It does not flag a success sample rate of 0, such as `$.configuration.LambdaSuccessFeedbackSampleRate`, which stops that protocol's successful delivery records while the feedback roles remain configured.

### Implementation.SNS.Topic.4

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::Topic`: `$.DeliveryStatusLogging[*].SuccessFeedbackRoleArn`, `$.DeliveryStatusLogging[*].FailureFeedbackRoleArn`

**Implements:** [Mitigation.SNS.Topic.2](#mitigationsnstopic2)

**Scoping** (for [Mitigation.SNS.Topic.2](#mitigationsnstopic2)): This CloudFormation Hook helps limit assumption of the delivery status logging roles in `$.DeliveryStatusLogging[*].SuccessFeedbackRoleArn` and `$.DeliveryStatusLogging[*].FailureFeedbackRoleArn` to Amazon SNS (`sns.amazonaws.com`). The topic create or update fails when a role's trust policy (`$.Role.AssumeRolePolicyDocument`) might allow public assumption or grant assumption beyond your reference policy.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.Topic.5

*AWS Config rule — Detective*

A periodic rule that evaluates via the API operation chain below.

1. Call `sns:ListTopics`; check `$.Topics[*].TopicArn`
2. Call `sns:GetTopicAttributes` with `$.TopicArn`; check `$.Attributes{key}`
3. Call `iam:GetRole` with `$.RoleName`; check `$.Role.AssumeRolePolicyDocument`
4. Call `accessanalyzer:CheckNoPublicAccess` with `$.policyDocument`, `$.resourceType` = `AWS::IAM::AssumeRolePolicyDocument`; check `$.result` = `PASS`
5. Call `accessanalyzer:CheckNoNewAccess` with `$.newPolicyDocument`, `$.existingPolicyDocument`, `$.policyType` = `RESOURCE_POLICY`; check `$.result` = `PASS`

**Implements:** [Mitigation.SNS.Topic.2](#mitigationsnstopic2)

**Scoping** (for [Mitigation.SNS.Topic.2](#mitigationsnstopic2)): This implementation reads the delivery status logging role references in `$.Attributes{key}` and each role's trust policy in `$.Role.AssumeRolePolicyDocument`. Flagging trust policies that permit public role assumption, or assumption beyond the reference policy you supply, helps you detect over-permissive roles.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access or grant new access beyond a reference policy you maintain.

The reference policy for `CheckNoNewAccess` allowlists the principals this role's trust policy may grant assumption to. The check fails when the trust policy grants role assumption beyond that allowlist. Example reference policies for role trust policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.Topic.6

*IAM role trust policy — Preventive*

Governs the IAM role trust policy statements below (the role is the resource).

| Principal | Actions | Condition keys |
|---|---|---|
| `sns.amazonaws.com` | `sts:AssumeRole` |  |

**Implements:** [Mitigation.SNS.Topic.2](#mitigationsnstopic2)

**Scoping** (for [Mitigation.SNS.Topic.2](#mitigationsnstopic2)): This implementation guidance recommends a trust policy that restricts assumption of your topic's delivery status logging role to the `sns.amazonaws.com` service principal.

**Reference:** Confirm that IAM role trust policies grant access only to the AWS service principals you intend. Review the `Principal` element of each policy and verify it names only the service principals your workload requires.

### Implementation.SNS.Topic.7

*Resource control policy (RCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sts:AssumeRole` | `*` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.SNS.Topic.2](#mitigationsnstopic2)

**Scoping** (for [Mitigation.SNS.Topic.2](#mitigationsnstopic2)): This implementation guidance recommends restricting `sts:AssumeRole` on your Amazon SNS topic IAM service role with the `aws:PrincipalOrgID` condition key in a resource control policy (RCP), while exempting AWS services that access the role on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. These conditions help prevent principals outside your organization from assuming the role.

**Reference:** To restrict access to trusted identities, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`. As a defense-in-depth measure, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys to restrict service access to your roles. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)
- [RCP sample: identity perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/identity_perimeter_rcp.json)

### Implementation.SNS.Topic.8

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::Topic`: `$.KmsMasterKeyId`

**Implements:** [Mitigation.SNS.Topic.3](#mitigationsnstopic3)

**Scoping** (for [Mitigation.SNS.Topic.3](#mitigationsnstopic3)): This implementation validates that `$.KmsMasterKeyId` carries a key identifier before a topic is created or updated. Failing templates that omit it helps address the mitigation, because message bodies are not protected by KMS server-side encryption until you set a key.

### Implementation.SNS.Topic.9

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::SNS::Topic`: `$.configuration.KmsMasterKeyId`

**Implements:** [Mitigation.SNS.Topic.3](#mitigationsnstopic3)

**Scoping** (for [Mitigation.SNS.Topic.3](#mitigationsnstopic3)): This implementation flags an Amazon SNS topic whose `$.configuration.KmsMasterKeyId` is not set, because that key identifier is what turns on server-side encryption for the topic. Surfacing those topics helps you detect where message bodies lack AWS KMS server-side encryption under a key you select.

### Implementation.SNS.Topic.10

*Resource policy — Preventive*

Attaches to `AWS::SNS::Topic` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:*` | `arn:${Partition}:sns:${Region}:${Account}:${TopicName}` | `aws:PrincipalIsAWSService`, `aws:PrincipalTag/dp:exclude:network`, `aws:SourceIp`, `aws:SourceVpc` (alternatives: `aws:SourceVpce`), `aws:ViaAWSService` |
| `sns:*` | `arn:${Partition}:sns:${Region}:${Account}:${TopicName}` | `aws:RequestedRegion`, `aws:SourceVpc` |

**Implements:** [Mitigation.SNS.Topic.4](#mitigationsnstopic4)

**Scoping** (for [Mitigation.SNS.Topic.4](#mitigationsnstopic4)): This implementation restricts Amazon SNS API calls against your topic with the `aws:SourceIp` and `aws:SourceVpc` condition keys. Requests from unexpected networks are denied, while AWS services that access your topic on your behalf are exempt (`aws:PrincipalIsAWSService`, `aws:ViaAWSService`).

**Reference:** To restrict access to expected networks, use the `aws:SourceVpc` or `aws:SourceVpce` condition keys. These scope access to a specific VPC or VPC endpoint. Because VPC IDs are unique only within a Region, pair `aws:SourceVpc` with `aws:RequestedRegion`. Use `aws:SourceIp` with your public IP ranges for requests that do not traverse a VPC endpoint. Use `aws:ViaAWSService` to exempt AWS services that call with your credentials through forward access sessions (FAS). Use `aws:PrincipalIsAWSService` to exempt AWS services that access your resources with their service principals. Use `aws:PrincipalTag/tag-key` to exempt principals that must legitimately operate from outside your expected networks. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a network perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-network)

### Implementation.SNS.Topic.11

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::TopicInlinePolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.10](#mitigationsnstopic10)

**Scoping** (for [Mitigation.SNS.Topic.10](#mitigationsnstopic10)): This implementation validates `$.PolicyDocument` at create and update time, confirming the topic's access policy carries the `aws:SecureTransport` condition key. Covering the update path helps address the mitigation, because a later stack update could otherwise drop that condition and reopen a cleartext path.

**Scoping** (for [Mitigation.SNS.Topic.4](#mitigationsnstopic4)): This implementation validates `$.PolicyDocument`, the access policy attached to your topic, at create and update time. Requiring conditions such as `aws:SourceVpc` and `aws:SourceVpce` helps address the mitigation by preventing policies that permit unexpected networks.

**Data perimeter** (for [Mitigation.SNS.Topic.4](#mitigationsnstopic4)): Access to your topic can be granted through a companion policy resource type instead of the topic resource itself. This implementation targets that path, validating `$.PolicyDocument` before the policy is attached to your topic. You can also require the standard network perimeter statement in these policy documents.
- [AWS::SNS::TopicInlinePolicy (CloudFormation Template Reference)](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicinlinepolicy.html)
- [AWS::SNS::TopicInlinePolicy (CloudFormation User Guide)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-topicinlinepolicy.html)
- [PolicyDocument property of AWS::SNS::TopicPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-topicpolicy.html#cfn-sns-topicpolicy-policydocument)
- [Standard network perimeter statement for Amazon SNS topic policies](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/service_specific_controls/sns_topic_policy.json#L33)

### Implementation.SNS.Topic.12

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::TopicPolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.10](#mitigationsnstopic10)

**Scoping** (for [Mitigation.SNS.Topic.10](#mitigationsnstopic10)): This implementation checks that the `$.PolicyDocument` applied to your topic conditions access on the `aws:SecureTransport` condition key. Rejecting templates that omit that condition helps prevent a topic policy from permitting cleartext HTTP requests to your topic.

**Scoping** (for [Mitigation.SNS.Topic.4](#mitigationsnstopic4)): This implementation validates `$.PolicyDocument` at create and update time for network conditions such as `aws:SourceVpc` and `aws:SourceVpce`. Rejecting a topic policy that grants access from unexpected networks helps you establish a network perimeter on the topics the policy attaches to.

**Data perimeter** (for [Mitigation.SNS.Topic.4](#mitigationsnstopic4)): A hook on this resource type can reject a `$.PolicyDocument` that grants topic permissions to unexpected networks. The Amazon SNS topic policy is reachable through two CloudFormation resource types, so cover `AWS::SNS::TopicInlinePolicy` as well. You can also require the standard network perimeter statement in these policy documents.
- [AWS::SNS::TopicPolicy resource (CloudFormation Template Reference)](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicpolicy.html)
- [PolicyDocument property of AWS::SNS::TopicPolicy (Template Reference)](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicpolicy.html#cfn-sns-topicpolicy-policydocument)
- [Standard network perimeter statement for Amazon SNS topic policies](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/service_specific_controls/sns_topic_policy.json#L33)
- [AWS::SNS::TopicInlinePolicy resource (CloudFormation Template Reference)](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicinlinepolicy.html)
- [AWS::SNS::TopicPolicy resource (CloudFormation User Guide)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-topicpolicy.html)
- [PolicyDocument property of AWS::SNS::TopicPolicy (User Guide)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-topicpolicy.html#cfn-sns-topicpolicy-policydocument)
- [AWS::SNS::TopicInlinePolicy resource (CloudFormation User Guide)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-sns-topicinlinepolicy.html)

### Implementation.SNS.Topic.13

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `sns:CreateTopic` — `$.Attributes{key}`
- `sns:SetTopicAttributes` — `$.AttributeName`, `$.AttributeValue`

**Implements:** [Mitigation.SNS.Topic.4](#mitigationsnstopic4)

**Scoping** (for [Mitigation.SNS.Topic.4](#mitigationsnstopic4)): This implementation records `CreateTopic` and `SetTopicAttributes` calls that carry the topic access policy in `$.Attributes{key}`, `$.AttributeName`, and `$.AttributeValue`. Surfacing those writes helps you detect a topic policy created or rewritten without your expected-network conditions.

**Data perimeter:** Monitoring these calls gives you a detective layer over the topic policy that defines your network perimeter. Review the `userIdentity` field on each event to see which identities changed the policy. When a write drops your expected-network conditions, you can remediate by adding the standard network perimeter statement to the resource-based policy.
- [CreateTopic request parameters: Attributes and the Policy attribute](https://docs.aws.amazon.com/sns/latest/api/API_CreateTopic.html#API_CreateTopic_RequestParameters)
- [Amazon SNS API Reference: CreateTopic](https://docs.aws.amazon.com/sns/latest/api/API_CreateTopic.html)
- [SetTopicAttributes request parameters: AttributeName and AttributeValue](https://docs.aws.amazon.com/sns/latest/api/API_SetTopicAttributes.html#API_SetTopicAttributes_RequestParameters)
- [Amazon SNS API Reference: SetTopicAttributes](https://docs.aws.amazon.com/sns/latest/api/API_SetTopicAttributes.html)
- [Network perimeter statement in the Amazon SNS topic policy example](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/service_specific_controls/sns_topic_policy.json#L33)

### Implementation.SNS.Topic.14

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:AddPermission` | `*` | `aws:PrincipalTag/dp:exclude:network` |

**Implements:** [Mitigation.SNS.Topic.4](#mitigationsnstopic4)

**Scoping** (for [Mitigation.SNS.Topic.4](#mitigationsnstopic4)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:exclude:network` condition key in a service control policy (SCP) to restrict `sns:AddPermission` to principals you designate. This condition helps prevent other principals in your organization from adding grants to your topic's resource-based policy that may allow access from unexpected networks.

**Data perimeter:** This service control policy (SCP) restricts which principals can call `sns:AddPermission`. That action adds grants to the resource-based policy on your topic that carries your network conditions. The network perimeter objective is that your resources can be accessed only from expected networks. You express those expected networks with network conditions in the resource-based policy on your topic. Resource control policies do not yet cover Amazon SNS, so that resource-based policy is where those conditions live. A grant added without your network conditions may allow access from unexpected networks. `sns:SetTopicAttributes` (its `Policy` attribute) and `sns:RemovePermission` also change the topic policy and are not governed by this SCP, which governs only the action that adds grants. A principal permitted those actions can still replace or trim the policy, so the network conditions in the topic policy itself remain the control that applies to every caller.
- [AddPermission - Amazon SNS API Reference](https://docs.aws.amazon.com/sns/latest/api/API_AddPermission.html)
- [Example SCP: Restrict resource-based policy configurations](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_resource_policy_configurations_scp.json)

### Implementation.SNS.Topic.15

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `*` | `*` | `aws:ResourceOrgID` (alternatives: `aws:ResourceAccount`, `aws:ResourceOrgPaths`) |

**Implements:** [Mitigation.SNS.Topic.5](#mitigationsnstopic5)

**Scoping** (for [Mitigation.SNS.Topic.5](#mitigationsnstopic5)): This implementation guidance recommends applying the `aws:ResourceOrgID` condition key in a service control policy (SCP) to govern Amazon SNS API calls against topics. For those calls, your identities are restricted from accessing topics outside your organization.

**Reference:** To restrict access to trusted resources, use the `aws:ResourceOrgID`, `aws:ResourceOrgPaths`, or `aws:ResourceAccount` condition keys. These scope access to resources that belong to a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing a resource perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-resource)
- [SCP sample: resource perimeter policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/resource_perimeter_scp.json)

### Implementation.SNS.Topic.16

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::Topic`: `$.KmsMasterKeyId`

**Implements:** [Mitigation.SNS.Topic.6](#mitigationsnstopic6)

**Scoping** (for [Mitigation.SNS.Topic.6](#mitigationsnstopic6)): This implementation validates `$.KmsMasterKeyId`, the topic's server-side encryption setting, at create and update time. Requests naming no key, or the AWS managed key, are denied, so a customer managed key gives you lifecycle control and an audit trail.

### Implementation.SNS.Topic.17

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::SNS::Topic`: `$.configuration.KmsMasterKeyId`

**Implements:** [Mitigation.SNS.Topic.6](#mitigationsnstopic6)

**Scoping** (for [Mitigation.SNS.Topic.6](#mitigationsnstopic6)): This implementation reads `$.configuration.KmsMasterKeyId` on your topic and resolves that key reference through `$.KeyMetadata.KeyManager` to confirm the key is customer managed. Flagging topics bound to the AWS managed key helps you detect where you lack control over key policy and rotation.

### Implementation.SNS.Topic.18

*Resource policy — Preventive*

Attaches to `AWS::SNS::Topic` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:AddPermission`, `sns:DeleteTopic`, `sns:GetDataProtectionPolicy`, `sns:GetTopicAttributes`, `sns:ListSubscriptionsByTopic`, `sns:ListTagsForResource`, `sns:Publish`, `sns:PutDataProtectionPolicy`, `sns:RemovePermission`, `sns:SetTopicAttributes`, `sns:Subscribe` | `arn:${Partition}:sns:${Region}:${Account}:${TopicName}` | `aws:PrincipalAccount` |

**Implements:** [Mitigation.SNS.Topic.7](#mitigationsnstopic7)

**Scoping** (for [Mitigation.SNS.Topic.7](#mitigationsnstopic7)): This implementation restricts Amazon SNS API calls against your topic with the `aws:PrincipalAccount` condition key. Requests that do not satisfy that condition are denied, so principals outside the AWS accounts you name cannot access your topic.

**Reference:** Confirm that resource-based policies grant access only to authorized identities. As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.SNS.Topic.19

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::TopicInlinePolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.SNS.Topic.7](#mitigationsnstopic7)

**Scoping** (for [Mitigation.SNS.Topic.7](#mitigationsnstopic7)): This hook validates `$.PolicyDocument`, the access policy you declare for your Amazon SNS topic, before CloudFormation applies it. Rejecting a policy that might allow public access helps prevent that policy from making your topic public.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.SNS.Topic.20

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::TopicPolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.SNS.Topic.7](#mitigationsnstopic7)

**Scoping** (for [Mitigation.SNS.Topic.7](#mitigationsnstopic7)): An IAM Access Analyzer custom policy check validates the topic access policy declared in `$.PolicyDocument` for public access. A policy that grants access to all principals fails the check, and the stack operation is denied.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.SNS.Topic.21

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::SNS::Topic`: `$.configuration.Policy`

**Implements:** [Mitigation.SNS.Topic.7](#mitigationsnstopic7)

**Scoping** (for [Mitigation.SNS.Topic.7](#mitigationsnstopic7)): This implementation evaluates the resource-based policy on your topic, recorded at `$.configuration.Policy`, through an IAM Access Analyzer public-access check. A `FAIL` result at `$.result` marks the topic noncompliant, helping you detect a policy that might allow access to any principal.

**Reference:** This implementation recommends using IAM Access Analyzer custom policy checks.

Custom policy checks (`CheckNoPublicAccess`) use automated reasoning to provide a higher level of assurance, provably verifying that a policy does not grant public access.
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)

### Implementation.SNS.Topic.22

*Guidance — Directive*

Consider using AWS Identity and Access Management Access Analyzer external access analyzers to help identify topics in your accounts that are shared with identities outside your organization. If necessary, remediate with the responsive controls of your choice. For example, you can add the standard identity perimeter statement to the resource-based policy.

- [Using AWS Identity and Access Management Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [Identity perimeter statement for an Amazon SNS topic policy](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/service_specific_controls/sns_topic_policy.json#L5)

**Implements:** [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Scoping** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): This recommendation directs you to review the resource-based policy on each topic for access granted to identities outside your organization. Remediating unintended access helps keep your topics accessible only to trusted identities, or to AWS services acting on behalf of trusted resources.

### Implementation.SNS.Topic.23

*Resource policy — Preventive*

Attaches to `AWS::SNS::Topic` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:*` | `arn:${Partition}:sns:${Region}:${Account}:${TopicName}` | `aws:PrincipalIsAWSService`, `aws:PrincipalOrgID` (alternatives: `aws:PrincipalAccount`, `aws:PrincipalOrgPaths`) |
| `sns:*` | `arn:${Partition}:sns:${Region}:${Account}:${TopicName}` | `aws:PrincipalIsAWSService`, `aws:SourceAccount`, `aws:SourceOrgID` (alternatives: `aws:SourceOrgPaths`) |

**Implements:** [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Scoping** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): This implementation governs Amazon SNS API calls against your topic with the `aws:PrincipalOrgID` condition key, while exempting AWS services that access it on your behalf (`aws:PrincipalIsAWSService`) and scoping those service requests with `aws:SourceOrgID`. As a result, identities outside your organization are restricted from accessing your topic.

**Reference:** Confirm that resource-based policies grant access only to trusted identities and to services acting on your behalf.

As a defense-in-depth measure, use the `aws:PrincipalOrgID`, `aws:PrincipalOrgPaths`, or `aws:PrincipalAccount` condition keys to restrict access to trusted identities. These scope access to IAM principals within a particular organization, specific organizational units, or a specific AWS account, respectively. These keys do not contain your organization or account values for requests by AWS service principals, so apply them only when `aws:PrincipalIsAWSService` is `false`.

To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Apply them only when `aws:PrincipalIsAWSService` is `true`. Also require that `aws:SourceAccount` is present in the request context, because not every service integration populates these keys.

Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.SNS.Topic.24

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::TopicInlinePolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Scoping** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): Before provisioning, this CloudFormation Hook checks `$.PolicyDocument`, your Amazon SNS topic's resource-based policy, for public access and access beyond your reference policy. It flags AWS service grants without source conditions, helping restrict your topic to trusted identities and AWS services acting on behalf of trusted resources.

**Data perimeter** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): A hook here helps prevent your developers from specifying a `$.PolicyDocument`, the topic's resource-based policy, that grants permissions to identities outside your organization. You can also require the standard identity perimeter statement in your topic resource-based policies. Covering both `AWS::SNS::TopicPolicy` and `AWS::SNS::TopicInlinePolicy` addresses every CloudFormation path for granting access to your topic.
- [AWS::SNS::TopicPolicy CloudFormation resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicpolicy.html)
- [PolicyDocument property of AWS::SNS::TopicPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicpolicy.html#cfn-sns-topicpolicy-policydocument)
- [Identity perimeter statement for Amazon SNS topic policies](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/service_specific_controls/sns_topic_policy.json#L5)
- [AWS::SNS::TopicInlinePolicy CloudFormation resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicinlinepolicy.html)

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.Topic.25

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::TopicPolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Scoping** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): This implementation validates the resource-based policy your template declares in `$.PolicyDocument` before the topic policy is provisioned. Confirming the policy grants no public access, no permissions you list, and no access to AWS services without source conditions helps address the mitigation.

**Data perimeter** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): Two CloudFormation resource types set an Amazon SNS topic's access policy through `$.PolicyDocument`. Governing both covers every template path that grants access to your topic. Validating each policy document at create and update time helps prevent a template from granting topic permissions to identities outside your organization. You can also require the standard identity perimeter statement in the topic policy your template declares.
- [AWS::SNS::TopicPolicy resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicpolicy.html)
- [AWS::SNS::TopicPolicy PolicyDocument property](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicpolicy.html#cfn-sns-topicpolicy-policydocument)
- [AWS::SNS::TopicInlinePolicy resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-sns-topicinlinepolicy.html)
- [Identity perimeter statement for Amazon SNS topic policies](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/service_specific_controls/sns_topic_policy.json#L5)

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.Topic.26

*CloudTrail management event — Detective*

Monitors the API calls below in CloudTrail.

- `sns:AddPermission` — `$.AWSAccountId[*]`, `$.TopicArn`
- `sns:CreateTopic` — `$.Attributes{key}` = `Policy`
- `sns:SetTopicAttributes` — `$.AttributeName` = `Policy`, `$.AttributeValue`, `$.TopicArn`

**Implements:** [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Scoping** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): This implementation monitors `AddPermission`, `CreateTopic`, and `SetTopicAttributes`, recording `$.AWSAccountId[*]` and the `Policy` attribute values those calls set on your topic. Surfacing those grants helps you detect access extended to identities outside your organization and review it against your identity perimeter.

**Data perimeter:** Monitoring these calls surfaces changes to the resource-based policy on your topic. `AddPermission` carries the account IDs being granted access to the topic. `CreateTopic` and `SetTopicAttributes` carry the `Policy` attribute value. Each event also records the identity that made the call. When a call extends access to identities outside your organization, remediate with the responsive controls of your choice. For example, you can add the standard identity perimeter statement to your topic's resource-based policy. That statement uses the `aws:PrincipalOrgID` condition key, while exempting AWS services that access your topic on your behalf (`aws:PrincipalIsAWSService`).
- [AddPermission request parameters (AWSAccountId.member.N)](https://docs.aws.amazon.com/sns/latest/api/API_AddPermission.html#API_AddPermission_RequestParameters)
- [Amazon SNS API Reference: AddPermission](https://docs.aws.amazon.com/sns/latest/api/API_AddPermission.html)
- [CreateTopic request parameters (Attributes and the Policy attribute)](https://docs.aws.amazon.com/sns/latest/api/API_CreateTopic.html#API_CreateTopic_RequestParameters)
- [Amazon SNS API Reference: CreateTopic](https://docs.aws.amazon.com/sns/latest/api/API_CreateTopic.html)
- [SetTopicAttributes request parameters (AttributeName and AttributeValue)](https://docs.aws.amazon.com/sns/latest/api/API_SetTopicAttributes.html#API_SetTopicAttributes_RequestParameters)
- [Amazon SNS API Reference: SetTopicAttributes](https://docs.aws.amazon.com/sns/latest/api/API_SetTopicAttributes.html)
- [Identity perimeter statement for Amazon SNS topic policies (data perimeter policy examples)](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/resource_control_policies/service_specific_controls/sns_topic_policy.json#L5)

### Implementation.SNS.Topic.27

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::SNS::Topic`: `$.configuration.Policy`

**Implements:** [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Scoping** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): This rule reads `$.configuration.Policy`, the resource-based policy on your topic, and checks it for public access. Flagging grants to AWS services that omit the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` condition keys helps you detect overly permissive access.

**Reference:** This implementation recommends using both IAM Access Analyzer basic policy checks and custom policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.

Custom policy checks (`CheckNoPublicAccess`, `CheckNoNewAccess`, `CheckAccessNotGranted`) use automated reasoning to provide a higher level of assurance. They provably verify that a policy does not grant public access, grant new access beyond a reference policy you maintain, or grant specified permissions.

The reference policy for `CheckNoNewAccess` allowlists the principals that may be granted access to the resource. The check fails when the resource-based policy grants access beyond that allowlist. Example reference policies for resource-based policies are available in the IAM Access Analyzer custom policy checks samples repository.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Access Analyzer custom policy checks](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-custom-policy-checks.html)
- [IAM Access Analyzer custom policy checks samples](https://github.com/aws-samples/iam-access-analyzer-custom-policy-check-samples)

### Implementation.SNS.Topic.28

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:AddPermission` | `*` | `aws:PrincipalTag/dp:exclude:identity` |

**Implements:** [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

**Scoping** (for [Mitigation.SNS.Topic.8](#mitigationsnstopic8)): This implementation guidance recommends applying the `aws:PrincipalTag/dp:exclude:identity` condition key in a service control policy (SCP) to govern `sns:AddPermission` for principals in your organization. This condition helps prevent principals you have not designated from granting identities outside your organization access to your topics through `AddPermission`.

**Data perimeter:** `sns:AddPermission` adds a statement to a topic's access control policy that grants access to the AWS account IDs named in the request. Amazon SNS is not covered by resource control policies (RCPs), so a service control policy (SCP) is the scalable place to govern these grants. Restricting this action to the principals you select helps prevent grants that reach identities outside your organization. `sns:SetTopicAttributes` (its `Policy` attribute) and `sns:RemovePermission` also change the topic policy. They are not governed by this SCP, which governs only the action that adds grants. A principal permitted those actions can still replace or trim the policy. So the identity conditions in the topic policy itself remain the control that applies to every caller, including principals outside your organization.
- [AddPermission – Amazon SNS API Reference](https://docs.aws.amazon.com/sns/latest/api/API_AddPermission.html)
- [Example SCP: Restrict resource-based policy configurations](https://github.com/aws-samples/data-perimeter-policy-examples/blob/main/service_control_policies/service_specific_controls/restrict_resource_policy_configurations_scp.json)

### Implementation.SNS.Topic.29

*Resource policy — Preventive*

Attaches to `AWS::SNS::Topic` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:Publish` | `arn:${Partition}:sns:${Region}:${Account}:${TopicName}` | `aws:SourceAccount`, `aws:SourceArn` |

**Implements:** [Mitigation.SNS.Topic.9](#mitigationsnstopic9)

**Scoping** (for [Mitigation.SNS.Topic.9](#mitigationsnstopic9)): This implementation restricts `sns:Publish` on your topic with the `aws:SourceArn` and `aws:SourceAccount` condition keys. Requests that do not satisfy those conditions are denied, so AWS services publish only on behalf of trusted resources and AWS accounts.

**Reference:** To restrict service access to your resources, use the `aws:SourceOrgID`, `aws:SourceOrgPaths`, `aws:SourceAccount`, or `aws:SourceArn` condition keys. These scope access performed on behalf of resources within a particular organization, specific organizational units, a specific AWS account, or a particular resource, respectively. Choose the condition key that matches the granularity of the restriction you want to enforce.
- [Establishing an identity perimeter](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_data-perimeters.html#access_policies_data-perimeters-identity)

### Implementation.SNS.Topic.30

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::TopicInlinePolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.SNS.Topic.9](#mitigationsnstopic9)

**Scoping** (for [Mitigation.SNS.Topic.9](#mitigationsnstopic9)): Before deployment, this CloudFormation hook validates `$.PolicyDocument`, your Amazon SNS topic's resource-based policy, to help prevent the cross-service confused deputy problem. The hook fails evaluation when `ValidatePolicy` reports that the policy grants an AWS service principal access without `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths`.

**Reference:** This implementation recommends using IAM Access Analyzer basic policy checks. Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.SNS.Topic.31

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::TopicPolicy`: `$.PolicyDocument`

**Implements:** [Mitigation.SNS.Topic.9](#mitigationsnstopic9)

**Scoping** (for [Mitigation.SNS.Topic.9](#mitigationsnstopic9)): This implementation restricts the resource-based policy on your topic, carried in `$.PolicyDocument`, with the `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, and `aws:SourceOrgPaths` condition keys. Requests that do not satisfy those conditions are denied, helping prevent confused deputy access to your topic.

**Reference:** This implementation recommends using IAM Access Analyzer basic policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.SNS.Topic.32

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::SNS::Topic`: `$.configuration.Policy`

**Implements:** [Mitigation.SNS.Topic.9](#mitigationsnstopic9)

**Scoping** (for [Mitigation.SNS.Topic.9](#mitigationsnstopic9)): This implementation flags your topic's resource-based policy, recorded at `$.configuration.Policy`, when it grants an AWS service principal access without `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths`. Surfacing those grants helps you detect where a service principal may act on behalf of unintended resources.

**Reference:** This implementation recommends using IAM Access Analyzer basic policy checks.

Basic policy checks (`ValidatePolicy`) validate your policy against IAM policy grammar and AWS best practices. These checks do not use automated reasoning. They provide actionable recommendations that help you author policies that are functional and conform to security best practices.
- [IAM policy grammar](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)
- [AWS best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Implementation.SNS.Topic.33

*Resource policy — Preventive*

Attaches to `AWS::SNS::Topic` and governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:*` | `arn:${Partition}:sns:${Region}:${Account}:${TopicName}` | `aws:SecureTransport` |

**Implements:** [Mitigation.SNS.Topic.10](#mitigationsnstopic10)

**Scoping** (for [Mitigation.SNS.Topic.10](#mitigationsnstopic10)): This implementation restricts Amazon SNS API calls against your topic with the `aws:SecureTransport` condition key. Requests that are not sent over TLS are denied, so publishers and subscribers cannot reach your topic over an unencrypted connection.

### Implementation.SNS.Topic.34

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::SNS::Topic`: `$.configuration.Policy`

**Implements:** [Mitigation.SNS.Topic.10](#mitigationsnstopic10)

**Scoping** (for [Mitigation.SNS.Topic.10](#mitigationsnstopic10)): This implementation checks the topic access policy recorded at `$.configuration.Policy` for a statement that requires encrypted connections through the `aws:SecureTransport` condition key. Flagging topics without that statement helps you detect where published message data can travel in cleartext.

### Implementation.SNS.Topic.35

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:*` | `*` | `aws:SecureTransport` |

**Implements:** [Mitigation.SNS.Topic.10](#mitigationsnstopic10)

**Scoping** (for [Mitigation.SNS.Topic.10](#mitigationsnstopic10)): This implementation restricts Amazon SNS API calls made by principals in your organization with the `aws:SecureTransport` condition key. Requests that are not sent over TLS are denied, so those principals cannot publish to or configure your topic over a cleartext HTTP connection.

### Implementation.SNS.Topic.36

*CloudFormation hook — Proactive*

Validates template properties before provisioning.

- `AWS::SNS::Topic`: `$.Tags[*].Key`, `$.Tags[*].Value`

**Implements:** [Mitigation.SNS.Topic.11](#mitigationsnstopic11)

**Scoping** (for [Mitigation.SNS.Topic.11](#mitigationsnstopic11)): This implementation validates at create time that `$.Tags[*].Key` and `$.Tags[*].Value` carry your required owner and purpose tags on a topic. Rejecting templates that omit them helps address the mitigation by keeping ownership and business function attributable for every topic you deploy.

### Implementation.SNS.Topic.37

*AWS Config rule — Detective*

A change-triggered rule that evaluates configuration item properties on change.

- `AWS::SNS::Topic`: `$.tags[*].key`, `$.tags[*].value`

**Implements:** [Mitigation.SNS.Topic.11](#mitigationsnstopic11)

**Scoping** (for [Mitigation.SNS.Topic.11](#mitigationsnstopic11)): This implementation flags Amazon SNS topics whose recorded tags at `$.tags[*].key` and `$.tags[*].value` omit your required owner and purpose keys. Surfacing those topics helps you attribute ownership and business function to each topic.

### Implementation.SNS.Topic.38

*Service control policy (SCP) — Preventive*

Governs the IAM permissions below.

| Actions | Resources | Condition keys |
|---|---|---|
| `sns:CreateTopic` | `*` | `aws:RequestTag/owner` |
| `sns:CreateTopic` | `*` | `aws:RequestTag/purpose` |
| `sns:CreateTopic` | `*` | `aws:TagKeys` |
| `sns:TagResource` | `arn:aws:sns:*:*:*` | `aws:RequestTag/owner` |
| `sns:TagResource` | `arn:aws:sns:*:*:*` | `aws:RequestTag/purpose` |
| `sns:TagResource` | `arn:aws:sns:*:*:*` | `aws:TagKeys` |
| `sns:UntagResource` | `arn:aws:sns:*:*:*` | `aws:TagKeys` |

**Implements:** [Mitigation.SNS.Topic.11](#mitigationsnstopic11)

**Scoping** (for [Mitigation.SNS.Topic.11](#mitigationsnstopic11)): This implementation restricts `sns:CreateTopic` and `sns:TagResource` with the `aws:RequestTag/owner`, `aws:RequestTag/purpose`, and `aws:TagKeys` condition keys, and `sns:UntagResource` with `aws:TagKeys`. Requests that omit the required tag keys in `$.Tags[*].Key` or remove them are denied, so a topic carries an owner and a business purpose.

**Reference:** Use `aws:RequestTag/owner` and `aws:RequestTag/purpose` so that a request omitting either one is denied. Use `aws:TagKeys` to require each key in its exact case because the tag-key portion of `aws:RequestTag/tag-key` is not case-sensitive. Also use `aws:TagKeys` on the untag action so that a request removing either key is denied.

### ManagedControl.SNS.Topic.1

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if Amazon Simple Notification Service (SNS) logging is enabled for the delivery status of notification messages sent to a topic for the endpoints

**Aliases:** `CONFIG.SNS.DT.1`

**Identifier:** `SNS_TOPIC_MESSAGE_DELIVERY_NOTIFICATION_ENABLED`

**ARN:** `arn:aws:controlcatalog:::control/6c3hptw3ztk6xuafbxlkl2wys`

Checks if Amazon Simple Notification Service (SNS) logging is enabled for the delivery status of notification messages sent to a topic for the endpoints. The rule is NON_COMPLIANT if the delivery status notification for messages is not enabled.

**Implements:** [Mitigation.SNS.Topic.1](#mitigationsnstopic1)

### ManagedControl.SNS.Topic.2

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if SNS topics are encrypted with AWS Key Management Service (AWS KMS)

**Aliases:** `CONFIG.SNS.DT.2`

**Identifier:** `SNS_ENCRYPTED_KMS`

**ARN:** `arn:aws:controlcatalog:::control/bn8ygh3zsy33drdeojmu09a4z`

Checks if SNS topics are encrypted with AWS Key Management Service (AWS KMS). The rule is NON_COMPLIANT if an SNS topic is not encrypted with AWS KMS. Optionally, specify the key ARNs, the alias ARNs, the alias name, or the key IDs for the rule to check.

**Implements:** [Mitigation.SNS.Topic.3](#mitigationsnstopic3), [Mitigation.SNS.Topic.6](#mitigationsnstopic6)

### ManagedControl.SNS.Topic.3

*AWS Config rule — Detective*

AWS managed control from Controls governing this resource/service.

**Title:** Checks if the SNS topic access policy allows public access

**Aliases:** `CONFIG.SNS.DT.3`

**Identifier:** `SNS_TOPIC_NO_PUBLIC_ACCESS`

**ARN:** `arn:aws:controlcatalog:::control/8r4xc9qth87n5ga1wrp2ipzm1`

Checks if the SNS topic access policy allows public access. The rule is NON_COMPLIANT if the SNS topic access policy allows public access.

**Implements:** [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8)

---

## Appendix

### Resources

#### OriginationNumber

*NETWORK*

An origination number is a dedicated phone number that identifies your account as the sender of an SMS message. Recipients see this number on their handsets instead of a shared or automatically selected sender. Common types include long codes, 10-digit long codes, toll-free numbers, and short codes.

- 1 operations, 0 threats, 0 mitigations

#### PhoneNumber

*CONFIGURATION*

An Amazon SNS phone number represents a mobile destination that receives text messages sent from your AWS account. Amazon SNS tracks whether each destination has opted out of receiving SMS messages from the account.

- 3 operations, 0 threats, 0 mitigations

#### PlatformApplication

*INTEGRATION*

An Amazon SNS platform application registers a mobile app with a third-party push notification service so Amazon SNS can deliver notifications on your behalf. You create one platform application for each app and push provider, such as Apple Push Notification service or Firebase Cloud Messaging. You supply the provider credentials that Amazon SNS uses to authenticate when it connects to that provider.

- 10 operations, 3 threats, 2 mitigations

#### SmsAttribute

*CONFIGURATION*

Amazon SNS SMS attributes are the account-level default settings that govern how text messages are sent from an AWS Region. They let you cap monthly SMS spending, choose whether deliveries are optimized for cost or for reliability, and set the sender identity that recipients see on their device.

- 2 operations, 3 threats, 2 mitigations

#### SmsSandboxAccountStatus

*CONFIGURATION*

The Amazon SNS SMS sandbox account status reports whether an AWS account is still confined to the SMS sandbox in a particular AWS Region. It is expressed as a simple in-sandbox indicator for the calling account, so applications and operators can tell which SMS sending rules currently apply before attempting a delivery.

- 1 operations, 0 threats, 0 mitigations

#### SmsSandboxPhoneNumber

*ACCESS_CONTROL*

An Amazon SNS SMS sandbox phone number is a record of a destination mobile number that your account has proven it can reach. Accounts that begin sending SMS through Amazon SNS start in a sandbox, a restricted testing environment designed to limit fraud and abuse and to protect your reputation as a sender. While the account remains in the sandbox, SNS delivers SMS only to numbers held in this verified list.

- 4 operations, 0 threats, 0 mitigations

#### Subscription

*INTEGRATION*

An Amazon SNS subscription connects a topic to a single endpoint that should receive the messages published to that topic. Endpoints can be Amazon SQS queues, AWS Lambda functions, or Amazon Data Firehose delivery streams. They can also be HTTP and HTTPS URLs, email addresses, phone numbers, or mobile push devices. Each subscription is owned separately from the topic, so one topic can fan out to many independent consumers.

- CloudFormation: `AWS::SNS::Subscription`
- 7 operations, 4 threats, 4 mitigations

#### Topic

*INTEGRATION*

An Amazon SNS topic is a named communication channel that decouples message publishers from message subscribers. A publisher sends a message once to the topic. SNS then replicates that message and pushes a copy to every subscribed endpoint.

- IAM resource ARN: `arn:${Partition}:sns:${Region}:${Account}:${TopicName}`
- CloudFormation: `AWS::SNS::Topic`, `AWS::SNS::TopicPolicy`, `AWS::SNS::TopicInlinePolicy`
- AWS Config: `AWS::SNS::Topic`
- 14 operations, 14 threats, 11 mitigations

### Operations

| Operation | Analyses | Resources | Mitigations |
|---|---|---|---|
| `AddPermission` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.1](#mitigationsns1), [Mitigation.SNS.2](#mitigationsns2), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.5](#mitigationsnstopic5), [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| `CheckIfPhoneNumberIsOptedOut` | DOCUMENTATION, DATA_PERIMETER | [PhoneNumber](#phonenumber) |  |
| `ConfirmSubscription` | DOCUMENTATION | [Subscription](#subscription) | [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2), [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3), [Mitigation.SNS.Topic.7](#mitigationsnstopic7) |
| `CreatePlatformApplication` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) | [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1), [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2) |
| `CreatePlatformEndpoint` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) |  |
| `CreateSMSSandboxPhoneNumber` | DOCUMENTATION | [SmsSandboxPhoneNumber](#smssandboxphonenumber) |  |
| `CreateTopic` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.Topic.1](#mitigationsnstopic1), [Mitigation.SNS.Topic.2](#mitigationsnstopic2), [Mitigation.SNS.Topic.3](#mitigationsnstopic3), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.6](#mitigationsnstopic6), [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8), [Mitigation.SNS.Topic.9](#mitigationsnstopic9), [Mitigation.SNS.Topic.10](#mitigationsnstopic10), [Mitigation.SNS.Topic.11](#mitigationsnstopic11) |
| `DeleteEndpoint` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) |  |
| `DeletePlatformApplication` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) |  |
| `DeleteSMSSandboxPhoneNumber` | DOCUMENTATION | [SmsSandboxPhoneNumber](#smssandboxphonenumber) |  |
| `DeleteTopic` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.1](#mitigationsns1), [Mitigation.SNS.2](#mitigationsns2), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.5](#mitigationsnstopic5), [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| `GetDataProtectionPolicy` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.Topic.7](#mitigationsnstopic7) |
| `GetEndpointAttributes` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) |  |
| `GetPlatformApplicationAttributes` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) | [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1), [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2) |
| `GetSMSAttributes` | DOCUMENTATION, DATA_PERIMETER | [SmsAttribute](#smsattribute) | [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1), [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2) |
| `GetSMSSandboxAccountStatus` | DOCUMENTATION, DATA_PERIMETER | [SmsSandboxAccountStatus](#smssandboxaccountstatus) |  |
| `GetSubscriptionAttributes` | DOCUMENTATION, DATA_PERIMETER | [Subscription](#subscription) | [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1), [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2), [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3), [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4) |
| `GetTopicAttributes` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.Topic.1](#mitigationsnstopic1), [Mitigation.SNS.Topic.2](#mitigationsnstopic2), [Mitigation.SNS.Topic.3](#mitigationsnstopic3), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.6](#mitigationsnstopic6), [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8), [Mitigation.SNS.Topic.9](#mitigationsnstopic9), [Mitigation.SNS.Topic.10](#mitigationsnstopic10) |
| `ListEndpointsByPlatformApplication` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) |  |
| `ListOriginationNumbers` | DOCUMENTATION, DATA_PERIMETER | [OriginationNumber](#originationnumber) |  |
| `ListPhoneNumbersOptedOut` | DOCUMENTATION, DATA_PERIMETER | [PhoneNumber](#phonenumber) |  |
| `ListPlatformApplications` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) | [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1) |
| `ListSMSSandboxPhoneNumbers` | DOCUMENTATION, DATA_PERIMETER | [SmsSandboxPhoneNumber](#smssandboxphonenumber) |  |
| `ListSubscriptions` | DOCUMENTATION, DATA_PERIMETER | [Subscription](#subscription) | [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3), [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4) |
| `ListSubscriptionsByTopic` | DOCUMENTATION, DATA_PERIMETER | [Subscription](#subscription) | [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2), [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3), [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4), [Mitigation.SNS.Topic.7](#mitigationsnstopic7) |
| `ListTagsForResource` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.11](#mitigationsnstopic11) |
| `ListTopics` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| `OptInPhoneNumber` | DOCUMENTATION, DATA_PERIMETER | [PhoneNumber](#phonenumber) |  |
| `Publish` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.1](#mitigationsns1), [Mitigation.SNS.2](#mitigationsns2), [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1), [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1), [Mitigation.SNS.Topic.1](#mitigationsnstopic1), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.5](#mitigationsnstopic5), [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| `PublishBatch` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.1](#mitigationsns1), [Mitigation.SNS.2](#mitigationsns2), [Mitigation.SNS.Topic.1](#mitigationsnstopic1), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| `PutDataProtectionPolicy` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) |  |
| `RemovePermission` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.1](#mitigationsns1), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| `SetEndpointAttributes` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) |  |
| `SetPlatformApplicationAttributes` | DOCUMENTATION, DATA_PERIMETER | [PlatformApplication](#platformapplication) | [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1), [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2) |
| `SetSMSAttributes` | DOCUMENTATION, DATA_PERIMETER | [SmsAttribute](#smsattribute) | [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1), [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2) |
| `SetSubscriptionAttributes` | DOCUMENTATION, DATA_PERIMETER | [Subscription](#subscription) | [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1), [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3), [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| `SetTopicAttributes` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.1](#mitigationsns1), [Mitigation.SNS.2](#mitigationsns2), [Mitigation.SNS.Topic.1](#mitigationsnstopic1), [Mitigation.SNS.Topic.2](#mitigationsnstopic2), [Mitigation.SNS.Topic.3](#mitigationsnstopic3), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.5](#mitigationsnstopic5), [Mitigation.SNS.Topic.6](#mitigationsnstopic6), [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8), [Mitigation.SNS.Topic.9](#mitigationsnstopic9), [Mitigation.SNS.Topic.10](#mitigationsnstopic10) |
| `Subscribe` | DOCUMENTATION, DATA_PERIMETER | [Subscription](#subscription) | [Mitigation.SNS.1](#mitigationsns1), [Mitigation.SNS.2](#mitigationsns2), [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1), [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3), [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4), [Mitigation.SNS.Topic.4](#mitigationsnstopic4), [Mitigation.SNS.Topic.7](#mitigationsnstopic7), [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| `TagResource` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) | [Mitigation.SNS.Topic.11](#mitigationsnstopic11) |
| `Unsubscribe` | DOCUMENTATION | [Subscription](#subscription) | [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2) |
| `UntagResource` | DOCUMENTATION, DATA_PERIMETER | [Topic](#topic) |  |
| `VerifySMSSandboxPhoneNumber` | DOCUMENTATION | [SmsSandboxPhoneNumber](#smssandboxphonenumber) |  |

Mitigations attach to an operation directly (the operation appears in the mitigation's request parameters) or through a threat whose request parameters include the operation.

### Goal Coverage (16 of 38)

| Mitigation Goal | Covered | Mitigations |
|---|---|---|
| [AI Services Opt-Out Policy](../mitigation-goals.md#ai-services-opt-out-policy) | — |  |
| [Automated Backups - AWS Backup](../mitigation-goals.md#automated-backups---aws-backup) | — |  |
| [Automated Backups - Service-Native](../mitigation-goals.md#automated-backups---service-native) | — |  |
| [CloudWatch Logging - Custom and V1 Vended Log Delivery](../mitigation-goals.md#cloudwatch-logging---custom-and-v1-vended-log-delivery) | yes | [Mitigation.SNS.PlatformApplication.1](#mitigationsnsplatformapplication1), [Mitigation.SNS.SmsAttribute.1](#mitigationsnssmsattribute1), [Mitigation.SNS.Topic.1](#mitigationsnstopic1) |
| [CloudWatch Logging - Vended Logs Delivery V2](../mitigation-goals.md#cloudwatch-logging---vended-logs-delivery-v2) | — |  |
| [Configure Optional Encryption at Rest](../mitigation-goals.md#configure-optional-encryption-at-rest) | yes | [Mitigation.SNS.Topic.3](#mitigationsnstopic3) |
| [Cross-Region Replication](../mitigation-goals.md#cross-region-replication) | — |  |
| [Cross-Service Confused Deputy Prevention - Resource-based policies](../mitigation-goals.md#cross-service-confused-deputy-prevention---resource-based-policies) | yes | [Mitigation.SNS.Topic.9](#mitigationsnstopic9) |
| [Customer Code Execution - Govern Code Content](../mitigation-goals.md#customer-code-execution---govern-code-content) | — |  |
| [Default Customer Managed Key Usage](../mitigation-goals.md#default-customer-managed-key-usage) | — |  |
| [Deletion/Termination Protection](../mitigation-goals.md#deletiontermination-protection) | — |  |
| [Detailed CloudWatch Metrics](../mitigation-goals.md#detailed-cloudwatch-metrics) | — |  |
| [Detailed Non-CloudWatch Metrics](../mitigation-goals.md#detailed-non-cloudwatch-metrics) | — |  |
| [Encryption in Transit - Resource Policy](../mitigation-goals.md#encryption-in-transit---resource-policy) | yes | [Mitigation.SNS.Topic.10](#mitigationsnstopic10) |
| [Encryption in Transit - Service Feature](../mitigation-goals.md#encryption-in-transit---service-feature) | yes | [Mitigation.SNS.Subscription.4](#mitigationsnssubscription4) |
| [IAM Integration for Service Authentication](../mitigation-goals.md#iam-integration-for-service-authentication) | — |  |
| [Identity Perimeter - Only trusted identities are allowed from my network](../mitigation-goals.md#identity-perimeter---only-trusted-identities-are-allowed-from-my-network) | yes | [Mitigation.SNS.2](#mitigationsns2) |
| [Identity Perimeter - Only trusted identities can access my resources](../mitigation-goals.md#identity-perimeter---only-trusted-identities-can-access-my-resources) | yes | [Mitigation.SNS.Topic.8](#mitigationsnstopic8) |
| [KMS Customer Managed Key Encryption](../mitigation-goals.md#kms-customer-managed-key-encryption) | yes | [Mitigation.SNS.Topic.6](#mitigationsnstopic6) |
| [Multi-AZ Deployment](../mitigation-goals.md#multi-az-deployment) | — |  |
| [Multi-AZ ENI Deployment](../mitigation-goals.md#multi-az-eni-deployment) | — |  |
| [Network Perimeter - My identities can access resources only from expected networks](../mitigation-goals.md#network-perimeter---my-identities-can-access-resources-only-from-expected-networks) | — |  |
| [Network Perimeter - My resources can only be accessed from expected networks](../mitigation-goals.md#network-perimeter---my-resources-can-only-be-accessed-from-expected-networks) | yes | [Mitigation.SNS.Topic.4](#mitigationsnstopic4) |
| [Private Subnet Placement](../mitigation-goals.md#private-subnet-placement) | — |  |
| [Resource Configuration - Trusted Targets](../mitigation-goals.md#resource-configuration---trusted-targets) | yes | [Mitigation.SNS.Subscription.3](#mitigationsnssubscription3) |
| [Resource Perimeter - My identities can access only trusted resources](../mitigation-goals.md#resource-perimeter---my-identities-can-access-only-trusted-resources) | yes | [Mitigation.SNS.Topic.5](#mitigationsnstopic5) |
| [Resource Perimeter - Only trusted resources can be accessed from my network](../mitigation-goals.md#resource-perimeter---only-trusted-resources-can-be-accessed-from-my-network) | yes | [Mitigation.SNS.1](#mitigationsns1) |
| [Resource Policy - Enforce Explicit Principal Definition](../mitigation-goals.md#resource-policy---enforce-explicit-principal-definition) | yes | [Mitigation.SNS.Topic.7](#mitigationsnstopic7) |
| [Resource Policy Lockout Prevention](../mitigation-goals.md#resource-policy-lockout-prevention) | — |  |
| [Resource Tagging - Owner and Purpose](../mitigation-goals.md#resource-tagging---owner-and-purpose) | yes | [Mitigation.SNS.Topic.11](#mitigationsnstopic11) |
| [Restrict IAM Role Assumption - Service or Execution Role](../mitigation-goals.md#restrict-iam-role-assumption---service-or-execution-role) | yes | [Mitigation.SNS.PlatformApplication.2](#mitigationsnsplatformapplication2), [Mitigation.SNS.SmsAttribute.2](#mitigationsnssmsattribute2), [Mitigation.SNS.Subscription.1](#mitigationsnssubscription1), [Mitigation.SNS.Topic.2](#mitigationsnstopic2) |
| [Restrict Security Group Access](../mitigation-goals.md#restrict-security-group-access) | — |  |
| [Service Principal Presigned URL](../mitigation-goals.md#service-principal-presigned-url) | — |  |
| [Trusted SSO Configuration](../mitigation-goals.md#trusted-sso-configuration) | — |  |
| [Turn on Block Public Access (BPA)](../mitigation-goals.md#turn-on-block-public-access-bpa) | — |  |
| [Unauthenticated Access Prevention](../mitigation-goals.md#unauthenticated-access-prevention) | yes | [Mitigation.SNS.Subscription.2](#mitigationsnssubscription2) |
| [Use Nitro instances](../mitigation-goals.md#use-nitro-instances) | — |  |
| [VPC Deployment](../mitigation-goals.md#vpc-deployment) | — |  |

### Impact Rollup

| Impact | Threats |
|---|---|
| [Account and Resource Discovery](../threat-impacts.md#account-and-resource-discovery) | [Threat.SNS.Subscription.4](#threatsnssubscription4), [Threat.SNS.Topic.4](#threatsnstopic4), [Threat.SNS.Topic.14](#threatsnstopic14) |
| [Data Collection](../threat-impacts.md#data-collection) | [Threat.SNS.1](#threatsns1), [Threat.SNS.2](#threatsns2), [Threat.SNS.Subscription.1](#threatsnssubscription1), [Threat.SNS.Subscription.3](#threatsnssubscription3), [Threat.SNS.Subscription.4](#threatsnssubscription4), [Threat.SNS.Topic.3](#threatsnstopic3), [Threat.SNS.Topic.4](#threatsnstopic4), [Threat.SNS.Topic.5](#threatsnstopic5), [Threat.SNS.Topic.6](#threatsnstopic6), [Threat.SNS.Topic.7](#threatsnstopic7), [Threat.SNS.Topic.9](#threatsnstopic9), [Threat.SNS.Topic.14](#threatsnstopic14) |
| [Data Destruction](../threat-impacts.md#data-destruction) | [Threat.SNS.Topic.9](#threatsnstopic9) |
| [Data Encrypted for Impact](../threat-impacts.md#data-encrypted-for-impact) | [Threat.SNS.Topic.1](#threatsnstopic1) |
| [Data Exfiltration](../threat-impacts.md#data-exfiltration) | [Threat.SNS.1](#threatsns1), [Threat.SNS.2](#threatsns2), [Threat.SNS.Subscription.1](#threatsnssubscription1), [Threat.SNS.Subscription.3](#threatsnssubscription3), [Threat.SNS.Topic.4](#threatsnstopic4), [Threat.SNS.Topic.5](#threatsnstopic5), [Threat.SNS.Topic.6](#threatsnstopic6), [Threat.SNS.Topic.7](#threatsnstopic7), [Threat.SNS.Topic.8](#threatsnstopic8), [Threat.SNS.Topic.9](#threatsnstopic9) |
| [Defense Evasion](../threat-impacts.md#defense-evasion) | [Threat.SNS.PlatformApplication.1](#threatsnsplatformapplication1), [Threat.SNS.PlatformApplication.2](#threatsnsplatformapplication2), [Threat.SNS.PlatformApplication.3](#threatsnsplatformapplication3), [Threat.SNS.SmsAttribute.1](#threatsnssmsattribute1), [Threat.SNS.SmsAttribute.2](#threatsnssmsattribute2), [Threat.SNS.SmsAttribute.3](#threatsnssmsattribute3), [Threat.SNS.Topic.2](#threatsnstopic2), [Threat.SNS.Topic.3](#threatsnstopic3), [Threat.SNS.Topic.8](#threatsnstopic8), [Threat.SNS.Topic.10](#threatsnstopic10), [Threat.SNS.Topic.11](#threatsnstopic11), [Threat.SNS.Topic.13](#threatsnstopic13) |
| [Inhibit System Recovery](../threat-impacts.md#inhibit-system-recovery) | [Threat.SNS.Topic.1](#threatsnstopic1) |
| [Initial Access](../threat-impacts.md#initial-access) | [Threat.SNS.PlatformApplication.1](#threatsnsplatformapplication1), [Threat.SNS.SmsAttribute.1](#threatsnssmsattribute1), [Threat.SNS.Subscription.2](#threatsnssubscription2), [Threat.SNS.Subscription.3](#threatsnssubscription3), [Threat.SNS.Topic.4](#threatsnstopic4), [Threat.SNS.Topic.8](#threatsnstopic8), [Threat.SNS.Topic.9](#threatsnstopic9), [Threat.SNS.Topic.10](#threatsnstopic10) |
| [Privilege Escalation](../threat-impacts.md#privilege-escalation) | [Threat.SNS.Topic.12](#threatsnstopic12) |
| [Service Stop](../threat-impacts.md#service-stop) | [Threat.SNS.Subscription.2](#threatsnssubscription2) |
