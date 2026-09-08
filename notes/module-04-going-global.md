# Module 4 - Going Global

## Using AWS Global Infrastructure

AWS Global Infrastructure provides geographic and architectural layers for placing workloads appropriately. These layers can help reduce latency, meet compliance requirements, improve availability, and serve users in different locations. Infrastructure as Code complements them by making repeated deployments consistent and automatable.

Global deployment is not an automatic requirement. A single-Region design may be appropriate when it satisfies availability, recovery, compliance, performance, and cost requirements. Additional locations should address a defined need because they also add cost and operational complexity.

## Choosing an AWS Region

Four main factors guide Region selection:

| Factor | Question to answer | Why it matters |
| --- | --- | --- |
| Compliance | Where are the workload and its data permitted or required to reside? | Laws, regulations, contracts, and organizational policies may restrict location. |
| Proximity | Where are the users and dependent systems? | Shorter geographic and network distance can reduce latency and improve responsiveness. |
| Feature availability | Are all required services, features, and instance types available? | AWS services and new features can become available in Regions at different times. |
| Pricing | What will the workload cost in each suitable Region? | Service prices can vary because of local infrastructure costs, taxes, and market conditions. |

### Compliance

Regulatory, legal, contractual, or internal requirements may determine where data is stored or processed. Data residency concerns where data is located, while data sovereignty concerns the laws and governance that apply to it. Compliance can override latency or cost preferences, but not every regulation mandates a particular AWS Region. The exact requirement must be evaluated for the workload and organization.

### Proximity

Locating a workload nearer to users or connected systems can shorten its network path, reduce latency, and improve responsiveness. The physically closest Region is not automatically correct because internet routing, compliance, service availability, architecture, and cost also matter. Network measurements are more useful than distance alone.

### Feature Availability

AWS service and feature coverage differs by Region, particularly when a service or capability is new. Every required service, feature, instance type, and integration should be checked before a Region is selected.

### Pricing

Prices can differ across Regions due to factors such as land, electricity, network infrastructure, and taxes. Cost can influence the choice after mandatory compliance, capability, and performance requirements are satisfied. The full architecture matters, including data transfer and any duplicate resources, not only one service's listed price.

The order above is a useful decision sequence, not an absolute algorithm for every scenario.

## Region Isolation

An **AWS Region** is a separate geographic area and a major administrative and fault-isolation boundary. Many AWS resources are scoped to the Region in which they are created. Regions being available does not cause application resources or data to be copied between them automatically.

Cross-Region replication, movement, and failover depend on the service, architecture, configuration, and permissions. Some services are global, and some Regional services offer configured replication. An application should not assume cross-Region behavior merely because multiple Regions exist.

## Regions, Availability Zones, and Edge Locations

| Infrastructure layer | Meaning | Main purpose |
| --- | --- | --- |
| AWS Region | A geographic area containing multiple Availability Zones | Place Regional workloads according to compliance, latency, feature, cost, and resilience requirements |
| Availability Zone | An isolated failure domain within a Region, made up of one or more data centers | Provide separation for highly available designs within one Region |
| Edge location | A point of presence in AWS global edge infrastructure, separate from the Region and AZ hierarchy | Bring content delivery, DNS, or network entry closer to users |

Availability Zones in the same Region have independent infrastructure designed to reduce correlated failure, while high-bandwidth, low-latency networking connects them. This makes multiple AZs useful for Regional resilience.

**Edge locations are not smaller Regions or Availability Zones.** They support edge and networking services and do not provide the full set of general-purpose Regional workload services.

## Multi-AZ Architecture

A **Multi-AZ** architecture places redundant resources across two or more Availability Zones in one Region. It is a common high-availability pattern because an application can retain capacity in another AZ if one AZ is impaired. Depending on the design and services used, this can support business continuity and faster recovery.

Merely creating resources in multiple AZs does not create working failover. The design may also need:

- Redundant compute
- Load balancing and routing
- Health checks
- Replicated or highly available data
- Service-specific failover configuration
- Sufficient capacity in the remaining AZs

## Multi-Region Architecture

A **Multi-Region** architecture deploys application resources in more than one AWS Region. It can protect against a broader Regional disruption, support disaster recovery, provide global availability, or reduce latency for geographically distributed users.

This design normally adds duplicate-resource cost, operational complexity, data-replication work, consistency challenges, and explicit failover planning. Not every highly available workload needs multiple Regions. The required recovery objectives, business impact, compliance constraints, user distribution, and budget should justify the design.

### Multi-AZ Versus Multi-Region

| Multi-AZ | Multi-Region |
| --- | --- |
| Uses multiple AZs in one Region | Uses multiple AWS Regions |
| Primarily protects against an AZ-level failure | Can protect against a Regional-level failure |
| Common first pattern for high availability | Used for broader disaster recovery or global requirements |
| Lower complexity and cost than a comparable Multi-Region design | Greater replication, routing, consistency, and operational complexity |

A Region already contains multiple Availability Zones. Multiple Regions are not required merely to use multiple AZs.

## High Availability, Agility, and Elasticity

These terms describe different system qualities:

- **High availability:** The system is designed to continue providing service despite component failures and to minimize downtime. Redundancy and working failover mechanisms support this goal.
- **Agility:** Teams can deploy, change, test, and adapt systems quickly.
- **Elasticity:** Capacity can adjust up or down to follow demand, often dynamically in AWS architectures.

A system can be agile without being highly available, highly available without being elastic, or elastic without having a sound failure-recovery design.

## Edge Locations and Edge Services

AWS edge locations are distributed points of presence that bring content delivery or network entry closer to users. Services using this infrastructure include Amazon CloudFront, Amazon Route 53, and AWS Global Accelerator.

### Amazon CloudFront

Amazon CloudFront is a content delivery network (CDN). It uses edge locations to deliver static and dynamic content with lower latency. Depending on the architecture, this content can include images, video, static site assets, data, and API or application responses.

CloudFront retrieves content from an **origin**, such as an Amazon S3 bucket or an HTTP server. Cacheable content can then be served from locations closer to users. Dynamic or uncached requests can still pass through CloudFront to the origin.

CloudFront improves content delivery. It is not a way to place arbitrary general-purpose compute at an edge location.

### Amazon Route 53

Amazon Route 53 is AWS's highly available Domain Name System (DNS) service. DNS records associate human-readable domain names with appropriate network destinations, which can include AWS resources or external endpoints. Route 53 helps direct users to an application and also supports domain registration and health checking.

DNS resolution is broader than converting a complete URL into an IP address. It resolves domain names according to record types and routing configuration; the URL path is handled later by the destination service.

### AWS Global Accelerator

AWS Global Accelerator is a global networking service that uses AWS edge infrastructure and the AWS global network. It provides stable network entry points and routes traffic toward suitable application endpoints based on factors such as endpoint health, client location, and configured policies. Its purpose is to improve application availability and network performance, not to cache content.

## AWS Outposts

AWS Outposts brings AWS-managed infrastructure and selected AWS services into a customer's on-premises location. It supports a consistent AWS operating model for hybrid architectures and workloads that need low-latency access to local systems or data, local processing, or particular residency arrangements.

Outposts is not an edge location or an AWS Region. It is infrastructure installed on premises and associated with an AWS Region. AWS operates and manages the Outposts infrastructure, while the customer remains responsible for site requirements and the applicable parts of the shared responsibility model.

## Infrastructure as Code

**Infrastructure as Code (IaC)** is the practice of defining infrastructure in machine-readable files or templates instead of configuring every resource manually. It supports:

- Repeatable, consistent deployments
- Automation with less manual error and configuration drift
- Source-controlled, reviewable, and traceable infrastructure definitions

IaC is a general practice. AWS CloudFormation is one AWS service that implements it.

## AWS CloudFormation

AWS CloudFormation is an AWS-native Infrastructure as Code service. A CloudFormation template defines the desired AWS resources and their properties. CloudFormation interprets that definition and makes the underlying AWS service API calls required to create, update, or delete resources in a **stack**.

CloudFormation uses a **declarative** approach: the template describes the desired end state rather than manually listing every API operation and procedural step. Templates can be stored in source control, reviewed, parameterized, and reused to create consistent environments. CloudFormation StackSets can extend controlled deployments across multiple accounts and Regions.

Repeatability does not guarantee that the result is secure, highly available, or resilient. Those outcomes depend on the resources, relationships, permissions, and failure behavior defined by the architecture.

### Console, CLI, SDK, and IaC

AWS services are ultimately operated through APIs, but the interaction model differs:

| Method | Main interaction model |
| --- | --- |
| AWS Management Console | Visual, browser-based, and mainly manual interaction |
| AWS CLI | Command-line interaction that can be scripted |
| AWS SDK | Programmatic interaction through language-specific libraries |
| IaC with CloudFormation | Declarative templates describing repeatable desired infrastructure |

CLI scripts and IaC can both automate operations, but they are not identical. A script usually specifies an ordered sequence of commands. Declarative IaC specifies the infrastructure state to create or manage, while the IaC service determines the necessary operations.

### Repeating an Environment in Another Region

Suppose Region A contains an environment and the same architecture is required in Region B. Manual recreation is slower and risks errors and configuration drift.

A reusable CloudFormation template can define the common architecture and accept Region-specific values. Deploying it in Region B improves consistency and speed while keeping the definition in source control. Required services and resource types must still be available in the target Region.

## Useful Mental Models

```text
AWS Region
├─ Availability Zone
├─ Availability Zone
└─ Availability Zone

Separate from this hierarchy:

Edge locations
└─ CloudFront / Route 53 / Global Accelerator

On premises
└─ AWS Outposts
```

```text
Region selection sequence

Compliance
  ↓
Proximity
  ↓
Feature availability
  ↓
Pricing

Useful decision sequence, not an absolute algorithm
```

```text
Multi-AZ     = Multiple AZs in one Region
Multi-Region = Multiple AWS Regions
```

```text
Origin
  ↓
CloudFront
  ↓
Edge location
  ↓
User
```

```text
CloudFormation template
  ↓
CloudFormation
  ↓
AWS APIs
  ↓
AWS resources
```

## Things I Initially Misunderstood

1. **Edge locations are not smaller AWS Regions or Availability Zones.** They belong to the separate AWS global edge infrastructure.
2. **A Region contains multiple Availability Zones.** Each AZ is an isolated failure domain made up of one or more data centers.
3. **Multi-AZ means redundancy across AZs within one Region.** It is a Regional high-availability pattern.
4. **Multi-Region means deploying across separate Regions.** It addresses broader geographic, recovery, or global-user requirements.
5. **Multiple Regions are not required merely to obtain multiple Availability Zones.** Multiple AZs already exist within a Region.
6. **Simply deploying into multiple AZs or Regions does not create working failover.** Location diversity is only part of the design.
7. **Failover depends on architecture and configuration.** Routing, health checks, replicated data, redundant capacity, and service-specific behavior must work together.
8. **CloudFormation is an Infrastructure as Code tool, not the definition of IaC itself.** IaC is the broader practice of defining and managing infrastructure as code.
9. **CLI scripts and IaC are related automation approaches but are not identical.** Scripts usually describe command sequences; declarative IaC describes desired infrastructure state.
10. **CloudFormation improves repeatability and consistency but does not automatically create high availability or resilience.** The template must define an architecture that provides those qualities.
11. **Region selection is not based on proximity alone.** Compliance, feature availability, and pricing also matter.
12. **Compliance requirements can override latency or cost preferences.** Mandatory constraints come before optimization preferences.
13. **Edge locations are separate from the Region and AZ hierarchy.** They provide points of presence for edge and networking services rather than full Regional workload environments.

## Assessment

**Result: 6/6**

The six questions tested:

- The relationship between Regions, Availability Zones, and edge locations
- Infrastructure as Code
- AWS CloudFormation
- Edge location benefits
- Region-selection criteria
- Multi-AZ and Multi-Region benefits

The assessment reinforced that Regions contain multiple Availability Zones, while edge locations are separate points of presence used for services such as low-latency content delivery. CloudFormation enables consistent and repeatable infrastructure deployment, and Region selection considers compliance, proximity, feature availability, and pricing.

Multi-AZ and Multi-Region designs can improve availability and fault tolerance when the architecture includes effective redundancy, replication, routing, health evaluation, and failover. Multiple Regions can also reduce latency for geographically distributed users. The assessment wording should be matched to the exact infrastructure boundary or benefit being described.

## What I Can Explain Now

- [x] Select an AWS Region using compliance, proximity, feature availability, and pricing.
- [x] Explain why compliance can constrain Region selection.
- [x] Distinguish a Region, an Availability Zone, and an edge location.
- [x] Compare Multi-AZ and Multi-Region architectures.
- [x] Separate high availability, agility, and elasticity.
- [x] Explain Amazon CloudFront as a CDN using edge locations.
- [x] Explain Amazon Route 53 and DNS at a foundational level.
- [x] Explain AWS Global Accelerator at a foundational level.
- [x] Place AWS Outposts within a hybrid infrastructure design.
- [x] Define Infrastructure as Code and its benefits.
- [x] Explain CloudFormation templates, stacks, and declarative infrastructure.
- [x] Compare CloudFormation with manual deployment.
- [x] Distinguish IaC from CLI scripting.
- [x] Trace CloudFormation from a template through AWS APIs to resources.
- [x] Explain why repeated infrastructure deployment benefits from automation.

## References

- [AWS Regions and Availability Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
- [AWS fault isolation boundaries: points of presence](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/points-of-presence.html)
- [Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [Amazon Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)
- [AWS Global Accelerator](https://docs.aws.amazon.com/global-accelerator/latest/dg/what-is-global-accelerator.html)
- [AWS Outposts](https://docs.aws.amazon.com/outposts/latest/server-userguide/what-is-outposts.html)
- [How AWS CloudFormation works](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-overview.html)
- [Working with CloudFormation templates](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-guide.html)
- [CloudFormation best practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
