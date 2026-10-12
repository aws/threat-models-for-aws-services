# Potential Threats

The threat scenarios assessed for every service in this release. 47 entries.

### CloudWatch log tampering

CloudWatch log entries can be deleted or made inaccessible by modifying log group retention policies, deleting log groups, or deleting log streams.

### Cross-service confused deputy problem through resource-based policies

An untrusted IAM principal with access to configure an AWS service that has been granted access to your resource through a resource-based policy may be able to cause that service to write to or read from the resource, because the policy does not restrict which source account or resource triggered the service action.

### Data exfiltration through overly-permissive outbound security group rules

A threat actor who controls a resource can use overly permissive outbound security group rules that permit traffic to unrestricted destination ranges (0.0.0.0/0 or ::/0) to interact with ungoverned network destinations from the resource.

### Data exposure from missing encryption at rest

An AWS resource that supports optional encryption at rest can be created or modified without encryption enabled, leaving the data it stores in plaintext and removing the layer of protection that encryption at rest provides.

### Data in transit interception through unencrypted client connection

Traffic destined for a resource can be captured and data in transit observed by a network-adjacent actor when the resource accepts unencrypted client connections.

### Data inaccessibility through resource-based policy lockout

Modifying a resource-based policy to remove all principals’ access renders the governed resource unmanageable, effectively making the data it protects inaccessible without deleting the resource itself.

### Data loss through encryption key deletion

Deleting an encryption key (such as a KMS customer managed key or CloudHSM key) permanently renders all data encrypted with that key inaccessible, because the key material required for decryption no longer exists.

### Data loss through inaccessible encryption key

Data can be encrypted with key material that the data owner does not control, such as a cross-account KMS key, imported key material, or an external key store. If the data owner loses access to the key, the encrypted data may become permanently inaccessible.

### Data loss through resource deletion

Deleting an entire AWS resource permanently destroys all data it contains as a side effect, even when the IAM principal does not have permissions to delete the data directly.

### Defense evasion through unconfigured enhanced monitoring

An AWS resource that supports enhanced monitoring capabilities can be created or configured without these capabilities enabled, reducing the ability to detect anomalous resource behavior.

### Defense evasion through unconfigured logging

AWS resources with optional logging capabilities that have never been enabled allow principals to interact with the resource without generating activity records, creating blind spots in security monitoring from the time of resource creation.

### Direct data deletion

An IAM principal can delete data (objects, records, messages) from an AWS resource that supports data deletion operations, without deleting the AWS resource itself.

### Flawed authentication model problem

An authentication or authorization mechanism you define that contains implementation errors (such as SAML signature validation gaps, JWT algorithm confusion, or overly permissive trust policies) can be used by a threat actor to gain unauthorized access to the AWS environment.

### Identity Perimeter - Untrusted principal access from a governed VPC network

An untrusted IAM principal operating from a governed VPC network can perform AWS service API operations.

### Identity Perimeter - Untrusted principal access to my resources

An untrusted IAM principal that has been granted access to my AWS resource can use the resource.

### Identity Perimeter - Untrusted principal assumption of my IAM roles

An untrusted IAM principal that has been granted access through the trust policy of a service role my resource uses can assume the role and act with its permissions.

### Lateral movement through permissive network rules

A threat actor who controls a VPC-connected resource can move laterally through overly permissive network rules (security groups, network ACLs) to interact with other resources in the VPC.

### Limited encryption governance with AWS owned or managed KMS keys

An AWS resource encrypted with an AWS owned or AWS managed KMS key limits your ability to control the key lifecycle and customize key policies. With AWS owned keys it also limits your ability to audit key usage through CloudTrail. Full control of these capabilities is available only with customer managed keys.

### Log data destruction through logging resource deletion

Deleting a dedicated logging resource that serves as the primary store for audit trails, log entries, or monitoring data permanently destroys the log data it contains.

### Logging disablement

Logging configurations on AWS resources can be disabled through modification API operations, stopping the generation of new log entries and creating a gap in security visibility for subsequent activities.

### Monitoring disablement

Enhanced monitoring and detailed metrics configurations on AWS resources can be disabled through modification API operations, reducing the ability to detect anomalous resource behavior such as cryptomining, unusual query patterns, or resource misuse.

### Network exposure through overly-permissive inbound security group rules

A threat actor with network reachability to a resource can reach services and administrative ports that should be restricted. Overly permissive inbound security group rules (unrestricted source ranges such as 0.0.0.0/0 or ::/0, overly broad port ranges, or all protocols) enable this access.

### Network Perimeter - Credential use from unexpected network

IAM credentials obtained by an unauthorized party can be used from any network location, including infrastructure outside your control, because IAM credentials are not inherently bound to a specific network location.

### Network Perimeter - Resource access from unexpected network

An IAM principal with valid credentials can access my AWS resource from an unexpected network, because the resource does not enforce network-origin restrictions.

### Network traffic interception through AWS network-inspection features

A threat actor with permissions to configure AWS network-inspection resources in a VPC, or to redirect a network path through an inspection appliance they control, can capture network traffic flowing to or from resources in that VPC, and read data in transit that has not been encrypted.

### Premature data deletion through retention policy modification

Reducing data retention thresholds (lifecycle policies, TTL settings, backup retention periods) causes existing data to be automatically deleted sooner than required, potentially violating compliance requirements.

### Public subnet resource exposure

An AWS resource placed in a public subnet may be directly reachable from ungoverned networks, regardless of whether the resource was intended to be publicly accessible.

### Resource Configuration - Data movement through an untrusted configured target

My IAM principal can configure my AWS resource to use a resource outside my trust boundary as a target of an operation, because the call the service makes against that target is not governed by my permissions guardrails.

### Resource missing required tags

An AWS resource created or modified without required tags escapes tag-based governance controls, including ownership attribution, cost allocation, and compliance inventory.

### Resource Perimeter - Access to untrusted resource from a governed VPC network

An IAM principal operating from a governed VPC network can interact with AWS resources outside my trust boundary, because the network does not restrict which resources can be accessed.

### Resource Perimeter - Data exfiltration to untrusted resource

My IAM principal can interact with AWS resources outside my trust boundary if IAM policies grant permissions beyond my own AWS accounts or organization.

### Service disruption from single availability zone dependency

An AWS resource deployed in a single Availability Zone becomes unreachable during an AZ-level outage, creating a single point of failure that can cause service disruption.

### Service disruption through resource deletion

Deleting an AWS resource that provides a service capability or infrastructure function causes denial of service to applications and users that depend on it.

### System capacity exhaustion

An AWS resource that provisions a data-plane network interface or endpoint in a subnet you control can be overwhelmed by request volume that exceeds its processing capacity or available network bandwidth, causing denial of service to legitimate users.

### Unauthenticated access from connected networks

An AWS resource deployed in a VPC that does not require authentication for data plane operations can be accessed by any actor with network connectivity to the resource.

### Unauthorized data access through AWS managed KMS key

Data encrypted with an AWS managed KMS key can be read by any IAM principal in the same account that holds data-plane read permissions, because the AWS managed key's key policy grants decrypt access to every same-account principal using the key through the owning service.

### Unauthorized data access through AWS owned KMS key

Data encrypted with an AWS owned KMS key can be read by any IAM principal with data-plane read permissions, because you cannot govern the key policy to restrict which principals can decrypt, and have no visibility into or ability to audit usage of the encryption key.

### Unauthorized data access through service principal presigned URL

An IAM principal with permissions to generate an Amazon S3 presigned URL signed by a service principal can use the URL to read or write data in a service-owned resource from an unexpected network, because API calls signed with service principal credentials are not subject to your IAM policies.

### Unauthorized data access through service-managed KMS grants

Data encrypted with a KMS key can be read by any IAM principal with data-plane read permissions when the service manages KMS operations in a way that does not allow the key policy to restrict access to individual callers.

### Unauthorized data access without customer managed KMS key encryption

Data at rest that is stored without customer managed KMS key based encryption can be read by any IAM principal that holds data-plane read permissions, because access is governed solely by those permissions without the additional decrypt authorization that a customer managed KMS key's key policy provides.

### Uncontrolled internet egress through public-subnet routing

A VPC-attached resource placed in a public subnet (a subnet whose route table has a route to an internet gateway) can send outbound traffic directly to the public internet, without the egress controls that private subnet placement would keep in the path, such as NAT-based inspection, egress firewalls, and traffic monitoring.

### Unencrypted instance-to-instance traffic through instance type selection

An instance type that does not provide automatic encryption of instance-to-instance traffic carries a resource's traffic between instances without that encryption layer, whether the peer is another of the resource's backing instances or another customer-operated instance that customer code on the resource's instances communicates with, which removes a layer of defense in depth and can prevent demonstrating the end-to-end encryption in transit expected by frameworks such as HIPAA, PCI DSS, and FedRAMP.

### Ungoverned AI data processing without opt-out policy

Content submitted to AWS AI services may be used for service improvement, creating ungoverned processing and storage of potentially sensitive data outside your control.

### Ungoverned network traffic from non-VPC resources

AWS resources with network communication capabilities that are deployed without VPC attachment can send and receive traffic outside the network controls that a VPC configuration would enforce, such as firewalls and traffic inspection.

### Unintended public resource access

An AWS resource configured to allow public access can be accessed by any actor who discovers it, including unauthenticated users or any authenticated AWS account holder.

### Vulnerable identity provider problem

An identity provider with security weaknesses (such as incomplete SAML signature validation, token replay susceptibility, or exposed metadata endpoints) can be used by a threat actor to gain unauthorized access to AWS resources.

### Weak authentication problem

An AWS resource that supports an inherently weak authentication mechanism (such as default credentials, HTTP basic authentication, plaintext password transmission, or trust of an unverified client-asserted identity) can have that weak mechanism used by a threat actor to gain unauthorized access to the resource.
