# Mitigation Goals

The security and governance control objectives assessed for every service in this release. 38 entries.

### AI Services Opt-Out Policy

Configure an AWS AI services opt-out policy through AWS Organizations covering AWS services with AI or ML features where your content could be used for service improvement or model training.

### Automated Backups - AWS Backup

Protect a resource by assigning it to an AWS Backup backup plan so recovery points are created and centrally governed. Applies to stateful resource types that AWS Backup supports as independently protectable resources (able to be selected in a backup plan's resource assignment).

### Automated Backups - Service-Native

Enable automated, scheduled, or continuous backups through a resource's own service API so recovery points are created without manual intervention. Applies to stateful resources whose service exposes a native backup capability (such as a backup retention period, point-in-time recovery, or an automated snapshot schedule).

### CloudWatch Logging - Custom and V1 Vended Log Delivery

Enable a resource to send logs or events to an Amazon CloudWatch log group for security-relevant visibility. Applies when API operations related to the resource include parameters for configuring CloudWatch logging that provides audit trails, access monitoring, or error detection.

### CloudWatch Logging - Vended Logs Delivery V2

Enable CloudWatch logging for a resource using the CloudWatch Logs vended logs delivery V2 mechanism. Applies when API operations related to the resource do not include logging configuration parameters and the logging provides security-relevant visibility.

### Configure Optional Encryption at Rest

Enable encryption at rest where the service does not encrypt automatically and data would otherwise be stored unencrypted. Applies when encryption must be explicitly enabled through a configuration parameter during resource creation or modification.

### Cross-Region Replication

Configure native service replication features that support continuous, automated data replication to a different AWS Region. Applies to stateful resources where the service provides a managed replication feature for multi-Region resiliency.

### Cross-Service Confused Deputy Prevention - Resource-based policies

Prevent cross-service confused deputy scenarios on resources that use resource-based policies to grant access to AWS service principals. The policies should restrict which accounts or resources the service may act on behalf of.

### Customer Code Execution - Govern Code Content

Implement code governance workflows for resources that allow custom code execution. Applies when a resource runs code you provide (functions, scripts, jobs, containers, notebooks) and does not currently support running that code within your Amazon VPC.

### Default Customer Managed Key Usage

Configure default encryption settings that automatically apply customer managed keys to new resources created after the configuration is set. Acts as a safety net when no explicit encryption is specified at resource creation time.

### Deletion/Termination Protection

Enable deletion or termination protection to guard a resource from accidental or unauthorized deletion. The protection is a boolean flag that causes delete or terminate API calls to fail until explicitly disabled.

### Detailed CloudWatch Metrics

Enable service-specific advanced monitoring features that provide more granular CloudWatch metrics beyond the default. Improves security visibility by providing finer-grained data for anomaly detection and incident response.

### Detailed Non-CloudWatch Metrics

Enable service-specific monitoring features that collect metrics or monitoring data through mechanisms other than CloudWatch and keep them outside CloudWatch Metrics. Typically uses agents or service-specific performance insights to capture detailed operational and security telemetry.

### Encryption in Transit - Resource Policy

Enforce encryption in transit through resource-based policies that deny requests made over unencrypted connections. Applies when no direct resource-level HTTPS enforcement setting exists, and also provides defense-in-depth alongside direct configuration where a cleartext path remains reachable.

### Encryption in Transit - Service Feature

Enable resource-specific configurations that directly enforce the encryption of data in transit. Applies when a resource has explicit configuration options for SSL/TLS, HTTPS, or certificate-based encryption during data transmission.

### IAM Integration for Service Authentication

Use service-specific access control mechanisms that integrate with AWS IAM as an optional enhancement or alternative to non-IAM authentication methods. Applies when a service offers IAM integration alongside passwords, API keys, or other non-IAM options.

### Identity Perimeter - Only trusted identities are allowed from my network

Restrict which identities can send requests through a governed VPC network so that only identities within your zone of trust can use the network to access AWS services. Helps prevent unintended data disclosure by disallowing the use of credentials for identities outside your zone of trust when used on a governed VPC network, as part of establishing an identity perimeter on your network.

### Identity Perimeter - Only trusted identities can access my resources

Restrict which identities can access your resources so that only trusted identities within your zone of trust can access them. Helps prevent unintended cross-account or anonymous access as part of establishing an identity perimeter on your resources.

### KMS Customer Managed Key Encryption

Use customer managed keys for encryption to gain full control over key lifecycle and permissions. Applies when a resource supports direct configuration with a KMS Key ID, Key ARN, or alias for encryption.

### Multi-AZ Deployment

Deploy or store resources across multiple Availability Zones using service-specific features for high availability and data redundancy. Provides automated failover and protection against AZ-level failures.

### Multi-AZ ENI Deployment

Deploy elastic network interfaces (ENIs) across multiple Availability Zones by configuring resources with multiple subnet IDs. Provides network-level redundancy and availability distinct from service-managed Multi-AZ features.

### Network Perimeter - My identities can access resources only from expected networks

Restrict identities to accessing AWS resources only when requests originate from expected network locations. Helps prevent the use of corporate credentials from non-corporate networks as part of establishing a network perimeter on identity.

### Network Perimeter - My resources can only be accessed from expected networks

Restrict access to AWS resources so that they can be accessed only from expected network locations. Helps prevent unintended access to resources from non-corporate networks as part of establishing a network perimeter on the resource.

### Private Subnet Placement

Associate resources only with private subnets. Helps prevent direct internet routing to the resource, reducing its network exposure.

### Resource Configuration - Trusted Targets

Configure a resource only with associated resources within your organization's trust boundary. Applies when a resource accepts non-AWS resources or resources in other AWS accounts as configuration parameters and the data plane calls to those resources are not governed by your permissions guardrails.

### Resource Perimeter - My identities can access only trusted resources

Restrict which resources your identities can access so that they interact only with resources within your organization's trust boundary. Helps prevent unintended data disclosure through use of corporate credentials as part of establishing a resource perimeter on identity.

### Resource Perimeter - Only trusted resources can be accessed from my network

Restrict which resources can be accessed through a governed VPC network so that requests are sent only to resources within your organization's trust boundary. Helps prevent unintended data disclosure by blocking requests to untrusted external resources as part of establishing a resource perimeter on your network.

### Resource Policy - Enforce Explicit Principal Definition

Limit who can access a resource through its resource-based policy by requiring explicit principal definitions rather than wildcard principals. Helps prevent the resource from inadvertently being made public or accessible to unintended actors.

### Resource Policy Lockout Prevention

Prevent resource-based policy modifications that bypass the policy lockout safety check and leave a resource unmanageable (orphaned).

### Resource Tagging - Owner and Purpose

Tag resources with business-relevant metadata to support resource identification, organization, and accountability. Owner and purpose tags help track ownership and the business function each resource supports.

### Restrict IAM Role Assumption - Service or Execution Role

Allow only the intended AWS service, acting on behalf of trusted resources, to assume service or execution roles. Helps prevent an unintended principal from assuming the role, and an AWS service from assuming it on behalf of an untrusted resource or account, through a trust policy that is overly permissive.

### Restrict Security Group Access

Configure the security groups governing a resource's network traffic to follow least-privilege best practices. Both inbound and outbound rules should restrict traffic to only the required sources and destinations (IP address ranges, referenced security groups, or prefix lists), protocols, and ports. Rules should not permit unrestricted access from or to any IP address (0.0.0.0/0 or ::/0), particularly on administrative or sensitive ports. Reduces the resource's network exposure.

### Service Principal Presigned URL

Restrict the use of presigned URLs signed by AWS service principals that grant access to your data. Applies when a service creates presigned URLs using its own service principal credentials to grant access to service-owned resources.

### Trusted SSO Configuration

Associate resources that support authentication through SSO protocols (OIDC, SAML) or external providers (Amazon Cognito, AWS IAM Identity Center) only with trusted identity providers. Helps prevent configuring untrusted identity providers that could be abused to gain unintended access.

### Turn on Block Public Access (BPA)

Enable Block Public Access features that manage public accessibility for a resource without requiring additional network or IAM controls. These are dedicated, service-specific features that provide a centralized toggle to prevent public access.

### Unauthenticated Access Prevention

Prevent direct unauthenticated access to resources that can be configured to allow anonymous or public access without authentication. Applies when the resource lacks a Block Public Access feature or dedicated public/private toggle.

### Use Nitro instances

Use EC2 instance types that support automatic encryption of instance-to-instance traffic for resources that support direct configuration with EC2 instance types. Instance types on the EC2 encryption-in-transit supported list use the Nitro System hardware to automatically encrypt traffic between instances. This adds a layer of encryption in transit that requires no application changes.

### VPC Deployment

Deploy resources within a VPC when VPC deployment is optional. Applies when a resource can be created without VPC association but optionally supports it, enabling network-level isolation and access controls.
