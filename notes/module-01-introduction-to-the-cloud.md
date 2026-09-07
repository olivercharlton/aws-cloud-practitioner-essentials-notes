# Module 1 - Introduction to the Cloud

## Client-Server Model

In the client-server model, a **client** requests information or an action and a **server** processes the request and returns a response. For example, a web browser can request a product page from a web server. The server may also retrieve data from a database before sending the completed page back to the browser.

```text
Client                    Server
  |---- request ----------->|
  |<--- response ------------|
```

Cloud computing does not replace this model. It provides flexible infrastructure and managed services on which clients, servers, databases, and other application components can run.

Early AWS services illustrate the move toward reusable, on-demand infrastructure: Amazon Simple Queue Service (SQS) began as a messaging service, Amazon Simple Storage Service (S3) added object storage, and Amazon Elastic Compute Cloud (EC2) added resizable compute capacity. SQS first appeared in beta in 2004; S3 and EC2 launched in 2006.

## What Cloud Computing Means

Cloud computing is the on-demand delivery of IT resources over a network, commonly the internet, with usage-based pricing. Resources such as compute power, storage, databases, and networking can be provisioned when needed without first buying and installing physical hardware.

Two ideas are central:

- **On-demand delivery:** Resources can be provisioned and deprovisioned when demand changes. Provisioning is much faster than a traditional hardware purchasing cycle.
- **Pay-as-you-go pricing:** Charges are generally based on consumption. This changes much infrastructure spending from a large fixed investment into a variable operating cost.

This allows a workload to start small without a large infrastructure investment. Pay-as-you-go does not mean every design is automatically inexpensive. The customer must still select suitable services, monitor usage, and remove resources that are no longer required. AWS Pricing Calculator helps estimate planned costs, AWS Cost Explorer analyzes cost and usage, and AWS Budgets can track spending or usage against defined thresholds.

## Deployment Models

| Model | Description | Typical reason to use it |
| --- | --- | --- |
| Cloud | Workloads run in a cloud provider's environment. | Rapid provisioning, flexible capacity, and access to managed services. |
| On-premises | Workloads run on infrastructure owned or operated in an organization's own facilities or chosen data center. | Existing investments, specific control requirements, or constraints that keep a workload outside the cloud. |
| Hybrid | Cloud resources and on-premises resources are connected and used together. | Gradual migration, data-location requirements, or integration with systems that remain on-premises. |

On-premises describes where and by whom the infrastructure is operated. It does not mean users can access the system only while physically inside the premises.

## Six Benefits of the AWS Cloud

1. **Replace large upfront investment with variable spending.** Consume resources when required instead of buying capacity far in advance.
2. **Benefit from provider-scale purchasing and operations.** AWS can spread infrastructure costs across many customers, supporting lower usage-based prices than many organizations could achieve alone.
3. **Avoid predicting capacity too early.** Over-provisioning wastes money on idle resources, while under-provisioning can cause poor performance or rejected demand. Cloud capacity can be adjusted as demand changes.
4. **Move faster.** Teams can create temporary environments in minutes, experiment, and deprovision them when finished instead of waiting for hardware.
5. **Reduce data center operations work.** AWS manages the underlying facilities and hardware, allowing customers to focus more effort on their applications and users.
6. **Reach users globally more quickly.** Workloads can be deployed in additional geographic areas without the customer building data centers there.

These are capabilities and economic advantages, not guarantees. AWS does not automatically make a workload scalable, highly available, or fault tolerant; those outcomes still depend on its architecture and configuration.

## AWS Global Infrastructure

### Regions and Availability Zones

An **AWS Region** is a separate geographic area in which AWS operates infrastructure. A Region contains multiple isolated **Availability Zones (AZs)**. Each AZ consists of one or more discrete data centers with redundant power, networking, and connectivity.

```text
AWS global infrastructure
└── Region
    ├── Availability Zone A
    │   └── One or more data centers
    ├── Availability Zone B
    │   └── One or more data centers
    └── Availability Zone C
        └── One or more data centers
```

The hierarchy is therefore:

```text
Region -> Availability Zone -> data center
```

AZs within a Region are physically separated but connected by high-bandwidth, low-latency, redundant networking. Regions are isolated from one another and are separated geographically.

### Choosing a Region

Placing an application closer to its users usually reduces network latency because data travels a shorter distance. User proximity is important, but it is not the only selection factor. Service availability, cost, compliance, legal requirements, and data residency can also affect the decision.

### High Availability and Fault Tolerance

- **High availability** aims to keep a system accessible with minimal interruption. It uses redundancy, health checks, and failover to reduce downtime.
- **Fault tolerance** is the stronger property of continuing to operate when a component fails, ideally without interruption noticeable to the user.

Neither property results merely from AWS having infrastructure around the world. The workload must be designed to use appropriate isolation boundaries and remove single points of failure.

### Multi-AZ and Multi-Region

| Design | What it does | Common purpose |
| --- | --- | --- |
| Multi-AZ | Distributes resources across two or more AZs in one Region. | Protects against a data center or AZ failure and is a common first resilience step for high availability. |
| Multi-Region | Distributes or replicates a workload across separate Regions. | Supports geographically distant users, Region-level resilience, disaster recovery, or regulatory requirements. |

Multi-Region architecture is more complex. It requires deliberate decisions about traffic routing, data replication, consistency, failover, recovery targets, security, and cost. AWS does not automatically copy all regional resources to another Region.

## AWS Shared Responsibility Model

AWS and the customer divide responsibility for security and compliance.

```text
Customer: security IN the cloud
  Data, identities, access, application code, configuration,
  and service-specific controls

AWS: security OF the cloud
  Facilities, physical hardware, networking, and the managed
  infrastructure that runs AWS services
```

### AWS Responsibilities

AWS protects and operates the infrastructure underlying AWS services. This includes:

- Physical security of data centers
- Hardware and the AWS global network
- The virtualization layer and host infrastructure
- Managed service components that the customer cannot administer

### Customer Responsibilities

The customer secures their workload and use of AWS services. Depending on the service, this can include:

- Classifying and protecting data
- Managing identities, credentials, and permissions
- Configuring network access and security controls
- Securing application code and dependencies
- Patching guest operating systems where the customer controls them
- Selecting, configuring, and monitoring encryption
- Logging, monitoring, backup, recovery, and compliance configuration

### The Boundary Depends on the Service

The responsibility boundary moves according to how much of the technology stack AWS manages.

- With **Amazon EC2**, AWS manages physical facilities, hardware, networking, and the virtualization layer. The customer manages the guest operating system, patches, installed software, data, permissions, and network controls such as security group rules.
- With a more abstracted managed service such as **Amazon S3**, AWS also operates the service platform. The customer still controls their data, access policies, encryption choices, and resource configuration.

Using a managed service can reduce operational work, but it does not transfer ownership of the customer's data or access decisions to AWS.

### Client-Side Encryption

Client-side encryption means encrypting data before sending it to AWS. AWS receives ciphertext rather than the original plaintext. The customer must manage the encryption process and protect the relevant keys. This differs from server-side encryption, where an AWS service encrypts the data after receiving it. The available controls and the division of configuration and key-management duties vary by service and encryption option.

## How Infrastructure and Shared Responsibility Work Together

AWS Global Infrastructure supplies geographic Regions and isolated AZs that can support low latency and resilient designs. The Shared Responsibility Model defines who must secure and configure the layers built on that infrastructure.

AWS maintaining separate AZs does not by itself make a customer's application highly available. The customer must deploy resources across AZs, configure traffic distribution and failover, protect data, and test recovery. AWS secures the facilities and underlying systems; the customer uses those capabilities to create a secure and resilient workload.

## Ecommerce Global Expansion Example

Consider an ecommerce company that begins with customers in the United Kingdom and later expands internationally.

1. It selects a Region based on user proximity, service availability, cost, and legal or data-residency requirements.
2. Within that Region, it deploys application components across multiple AZs so that a single-AZ failure does not take down the storefront.
3. As customers appear in distant markets, it evaluates additional Regions to reduce latency and meet recovery or regulatory needs.
4. It designs traffic routing, data replication, inventory consistency, payment processing, and failover. These do not happen merely because another Region is available.
5. AWS secures the physical facilities, hardware, network, and managed service infrastructure. The company secures customer records, application code, identities, permissions, service configurations, and encryption choices.
6. Sensitive data can be encrypted by the application before upload when client-side encryption is required. The company remains responsible for protecting the associated keys.

This example connects the two concepts: global infrastructure provides locations and isolation boundaries, while shared responsibility requires the company to architect and secure how its workload uses them.

## Things I Initially Misunderstood

1. **On-premises does not mean systems can only be accessed from inside the physical premises.** Remote users can access on-premises systems through appropriately configured networks; the term describes where the infrastructure runs.
2. **I initially reversed Regions and Availability Zones.** A Region contains multiple AZs, and an AZ contains one or more data centers.
3. **High availability is not simply about having many locations around the world.** It comes from an architecture that uses redundancy and failover to minimize interruption.
4. **Fault tolerance means the system continues operating despite component failures.** Redundant components and automatic failover must be designed into the workload.
5. **AWS does not automatically scale every workload by default.** Scaling requires a service that supports it and appropriate customer configuration, such as policies, limits, and health checks.
6. **AWS does not automatically make an application highly available.** The customer must distribute the workload, remove single points of failure, and configure failover.
7. **Client-side encryption means encrypting data before sending it to AWS.** The client controls the encryption step and must manage the required keys securely.
8. **AWS provides infrastructure and services, but the customer still has to architect and configure the solution correctly.** AWS secures the cloud platform; it does not make every customer workload secure, resilient, or cost-efficient automatically.

## Assessment

**Result: 8/8**

The assessment tested the client-server model, the meaning and benefits of cloud computing, deployment models, AWS infrastructure hierarchy, availability and fault tolerance, Region selection, and the division of duties under shared responsibility.

The main exam technique was to read scenario wording precisely. Several options may describe real AWS benefits or responsibilities, but the best answer is the one that most directly matches the benefit, architecture property, or responsibility described in the scenario.

I also learned not to confuse reduced operational overhead with solving a capacity problem, to read the Region and AZ hierarchy in the correct direction, and to identify client-side encryption from the fact that encryption happens before data is sent to AWS.

## What I Can Explain Now

- [x] Describe the client-server request and response pattern.
- [x] Define cloud computing, on-demand resource delivery, and pay-as-you-go pricing.
- [x] Distinguish cloud, on-premises, and hybrid deployment models.
- [x] Summarize the six benefits of using the AWS Cloud.
- [x] Explain the Region, Availability Zone, and data center hierarchy.
- [x] Explain how user proximity can affect latency and Region selection.
- [x] Distinguish high availability from fault tolerance.
- [x] Compare Multi-AZ and Multi-Region designs.
- [x] Separate AWS and customer duties under the Shared Responsibility Model.
- [x] Explain why responsibility changes according to the service used.
- [x] Define client-side encryption.
- [x] Apply global infrastructure and shared responsibility concepts to an ecommerce expansion scenario.

## References

- [AWS Regions and Availability Zones](https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions-availability-zones.html)
- [AWS Shared Responsibility Model](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/shared-responsibility.html)
- [Overview of Amazon Web Services](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/)
