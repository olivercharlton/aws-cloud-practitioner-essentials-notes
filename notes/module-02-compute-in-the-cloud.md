# Module 2 - Compute in the Cloud

## Amazon EC2 Fundamentals

Amazon Elastic Compute Cloud (**Amazon EC2**) provides resizable compute capacity as **Compute as a Service**. An EC2 instance is a virtual machine that can run an operating system and applications. Instead of purchasing, installing, and maintaining a physical server before using it, a customer can provision an instance when needed and select its compute, memory, networking, and storage characteristics.

The basic state changes have different effects:

- **Launch:** Provision a new instance from a selected configuration.
- **Stop:** Shut down an EBS-backed instance so that it can be started again. On-Demand EC2 instance usage is not charged while it is stopped, but attached Amazon EBS storage, public IPv4 addresses, pricing commitments, and other retained resources can still incur charges.
- **Terminate:** Permanently delete the instance. Attached resources are deleted or retained according to their configuration, so termination does not guarantee that every related charge ends.

Compared with buying on-premises servers, EC2 makes it faster to try a configuration, change it, and release it without owning the physical hardware. The customer retains control over the guest operating system, installed software, network settings, and storage configuration. AWS manages the underlying facilities, host hardware, and virtualization infrastructure.

An instance can often be vertically resized by stopping it, changing its instance type, and starting it again. Compatibility and service constraints must still be checked. EC2 provides the capability to resize and scale, but a workload does not scale automatically unless appropriate mechanisms, such as EC2 Auto Scaling policies, are configured.

## Virtualization, Hypervisors, and Multi-Tenancy

A **physical host** is the underlying server. A **hypervisor** is the virtualization layer that allocates host resources to virtual machines and isolates those machines from one another. Several EC2 instances can therefore use one physical host while behaving as separate computers.

AWS manages the host and hypervisor. The hypervisor isolates resources such as CPU and memory, and EC2 also provides network and storage isolation controls.

In technically precise EC2 terms, **multi-tenancy means that multiple customers can have separate EC2 instances running on shared underlying physical hardware while those instances remain isolated**. It does not mean that several users merely log in to one EC2 instance. The course assessment used imprecise wording for this concept, so these notes retain the technically correct definition.

## Launching and Using an EC2 Instance

The simplified lifecycle is:

1. **Launch:** Select the image, hardware profile, networking, storage, access, and initialization settings.
2. **Connect:** Establish an authorized management session.
3. **Use:** Install or run the required workload, monitor it, and manage its lifecycle.

### AMIs and Instance Types

An **Amazon Machine Image (AMI)** is a consistent template used to launch instances. It contains the software configuration needed to boot an instance and can define the operating system, preconfigured software, and storage mappings. Launching several instances from the same AMI helps produce a consistent starting state.

The **instance type** specifies a combination of resources and capabilities, including:

- CPU capacity
- Memory
- Networking capability
- Supported storage characteristics

The AMI answers, broadly, "what software image should boot?" The instance type answers "what hardware capacity and characteristics should it receive?"

### Connecting

- **SSH** is commonly used for command-line access to Linux instances.
- **RDP** is commonly used for graphical remote access to Windows instances.
- **AWS Systems Manager**, including Session Manager when correctly configured, provides another management and access option without relying on a directly exposed SSH or RDP endpoint.

All methods require appropriate identity permissions, instance configuration, and network or Systems Manager connectivity.

## EC2 Instance Families

The best family is determined by the workload's limiting resource or special requirement.

| Family | Appropriate requirement or bottleneck | Example workloads |
| --- | --- | --- |
| General purpose | A balanced mix of compute, memory, and networking, especially when the workload profile is not yet clear. | Web servers, application servers, development and test workloads. |
| Compute optimized | Sustained CPU-intensive processing or complex calculation. | Scientific modeling, batch computation, media processing, and high-performance application servers. |
| Memory optimized | A large active working set must remain in RAM for fast processing. | In-memory databases, caching, and in-memory analytics. |
| Accelerated computing | Specialized accelerators or coprocessors are needed for work that CPUs handle less efficiently. | Graphics and rendering, floating-point processing, and some machine learning training or inference. |
| Storage optimized | High local disk throughput, low-latency I/O, or frequent reads and writes against large locally stored datasets. | Data processing systems, search workloads, and high-throughput transactional storage. |

The phrase "large dataset" is not enough to choose between memory optimized and storage optimized. The bottleneck matters:

```text
RAM or active working set          -> Memory optimized
Disk I/O or local disk throughput  -> Storage optimized
```

For example, repeatedly analyzing a working dataset that must remain in RAM points toward memory optimized. Scanning a large local dataset where disk read throughput limits performance points toward storage optimized.

## Instance Family Versus Instance Size

- The **family** determines the resource balance or specialization.
- The **size** determines how much capacity is provided within that family.
- Larger sizes generally provide more capacity and cost more.
- **Right-sizing** means selecting enough capacity to meet performance and availability requirements without paying for unnecessary resources.

An instance choice is not permanent. A workload can be moved to another size or family later when compatibility allows. Measurements and testing should guide that change.

## Ways to Interact with AWS

AWS services are ultimately accessed through APIs. The main interfaces call those APIs in different ways.

| Interface | Best suited to |
| --- | --- |
| AWS Management Console | Graphical browser-based learning, exploration, detailed resource views, billing, monitoring, and manual or test tasks. |
| AWS Command Line Interface (AWS CLI) | Terminal commands, repeatable operations, scripts, and automation that can reduce inconsistent manual configuration. |
| AWS Software Development Kits (AWS SDKs) | Programmatic calls from application code in languages such as Python, allowing software to interact directly with AWS APIs. |

The Management Console has detailed capabilities; its main distinction is visual and manual interaction rather than command-based or application-based automation.

**AWS CloudShell** is a managed, browser-based shell with the AWS CLI and other tools available. It uses the signed-in console session's credentials, subject to the user's permissions, and avoids installing the CLI locally for supported tasks.

## EC2 Launch Demonstration

The course demonstration launched a simple web server through the EC2 console:

1. Assign a descriptive instance name.
2. Choose an AMI. The demonstration used an Amazon Linux image.
3. Choose an instance type. `t2.micro` was a course example, not a universal recommendation.
4. Select or create a key pair. The public key is placed where the instance can use it for authentication, while the user must protect the private key.
5. Configure network settings and allow inbound HTTP traffic for the web server. Production rules should be limited to the traffic actually required.
6. Configure storage. The example used an 8 GB `gp3` Amazon EBS volume. The 8 GB value described storage, not RAM.
7. Provide **User Data**, a startup script used to automate initialization. The example installed and started Nginx when the instance launched.
8. Launch the instance, obtain its public IPv4 address, and enter that address in a browser to request the web page.

User Data makes initialization repeatable. It must match the selected AMI and should not contain secrets.

## EC2 Pricing Models and Capacity Options

The correct option depends on workload predictability, interruption tolerance, capacity requirements, tenancy, and licensing.

| Option | Intended use |
| --- | --- |
| On-Demand Instances | No long-term commitment. Suitable for uncertain, changing, short-term, or non-interruptible workloads and for establishing a usage baseline before making a commitment. |
| Savings Plans | A 1-year or 3-year commitment to a consistent amount of eligible compute usage, measured in spend per hour. Appropriate for predictable usage while retaining more flexibility across eligible compute than an EC2 Reserved Instance, depending on the Savings Plan type. |
| Reserved Instances | An EC2-focused billing discount based on a 1-year or 3-year commitment to specified attributes. Appropriate for steady and predictable EC2 usage. A Reserved Instance is a billing construct, not a separate running instance. |
| Spot Instances | Uses spare EC2 capacity at a potentially very large discount, but instances can be interrupted. Suitable for interruption-tolerant work such as flexible batch processing, testing, rendering, or distributed jobs with checkpointing. |
| Dedicated Hosts | An entire physical server is dedicated to one customer, with visibility and control over instance placement and host resource allocation. Useful for host-bound licensing, compliance, and specialized isolation requirements. |
| Dedicated Instances | Instances run on hardware dedicated to one customer account and isolated from other AWS customers. They provide less physical-host placement and allocation control than Dedicated Hosts. |
| EC2 Capacity Reservations | Reserves EC2 capacity for matching instances in a specific Availability Zone. Useful when the ability to launch capacity is more important than obtaining a discount. Charges apply under the reservation's billing rules even when some reserved capacity is unused. |

The central distinctions are:

```text
Savings Plans / Reserved Instances -> Pricing commitment and discount
Capacity Reservations               -> Capacity availability in a specific AZ
Dedicated Hosts                     -> Exclusive physical host and placement control
```

Reserved Instances and Capacity Reservations must not be treated as synonyms. A Regional Reserved Instance primarily provides a billing benefit without reserving capacity. A Zonal Reserved Instance can provide capacity in its selected AZ, but EC2 Capacity Reservations are the direct capacity-first option and can be combined with eligible billing discounts.

## Scalability and Elasticity

- **Scalability** is the ability of a system to increase or decrease capacity.
- **Elasticity** is the ability to match provisioned resources to changing demand, often dynamically and automatically in an AWS architecture.

Elasticity does not replace scalability. It builds on scalable capacity. A system may support manual capacity changes and therefore be scalable without automatically adapting to demand, so it is not yet elastic.

## Vertical and Horizontal Scaling

- **Scale up, or vertical scaling:** Increase the resources of one instance, such as adding CPU or memory by selecting a larger instance size.
- **Scale out, or horizontal scaling:** Add more instances so they can perform work in parallel.

Horizontal scaling is often useful for application tiers that process many independent requests. Work can be distributed among instances, failed capacity can be replaced, and the group can grow without relying on one increasingly large server. The application must be designed so that requests and state can be handled safely across multiple instances.

## High Availability and Redundancy

High availability requires an architecture that avoids single points of failure. For an EC2 application tier, this commonly means running redundant instances across multiple Availability Zones. If one AZ has a problem, healthy instances in another AZ can continue serving the application.

Merely launching an EC2 instance does not make a workload highly available. Traffic distribution, health checks, data design, capacity, monitoring, and failover behavior must also be configured and tested.

## Amazon EC2 Auto Scaling

Amazon EC2 Auto Scaling manages the number of EC2 instances in an Auto Scaling group. It can **scale out** by adding instances and **scale in** by removing them. Scaling policies can react to performance or application metrics observed through Amazon CloudWatch. An Auto Scaling group can also replace unhealthy instances to maintain its target capacity.

The three capacity settings are:

```text
Minimum capacity = lower boundary
Desired capacity = current target
Maximum capacity = upper boundary
```

These values should follow load testing, workload behavior, availability requirements, monitoring data, service quotas, and cost constraints. They should not be guessed arbitrarily.

Auto Scaling responds to configured signals about demand and health. It does not inherently decide whether traffic is legitimate or malicious. DDoS detection, traffic filtering, access control, and other security controls are separate concerns.

## Elastic Load Balancing

Elastic Load Balancing (**ELB**) distributes incoming traffic across registered targets such as EC2 instances. It routes traffic to healthy capacity, helps prevent one instance from receiving all requests, and scales its load-balancing capacity as traffic changes. A load balancer can be internet-facing or internal.

ELB can provide a stable endpoint between application tiers. A front-end component sends requests to the load balancer rather than tracking every changing backend instance. This decouples the caller from individual targets.

The division of work is precise:

```text
EC2 Auto Scaling -> Controls how many EC2 instances exist
ELB              -> Controls where traffic is sent
```

ELB does not create EC2 instances. Together, EC2 Auto Scaling adds or removes capacity, while ELB distributes traffic across the available healthy capacity.

### ELB Routing Concepts

The course introduced several general routing concepts:

- **Round Robin:** Rotate requests across available targets.
- **Least Connections:** Prefer the target with the fewest active connections.
- **IP Hash:** Use address information to choose a target consistently.
- **Least Response Time:** Prefer a target expected to respond more quickly.

These are revision-level routing concepts, not a claim that every AWS load balancer type exposes or uses all four algorithms. Actual behavior depends on the load balancer type, protocol, target configuration, and supported routing settings.

## Messaging and Queuing

In a **tightly coupled** architecture, one component may depend directly on another component being available and responding immediately. A slowdown or failure can propagate through the call chain and cause a cascading failure.

In a **loosely coupled** architecture, components communicate through a boundary such as a queue or event service. A **message queue** buffers messages until a consumer can process them. The **payload** is the useful data carried in a message.

Buffering allows producers and consumers to operate at different rates. If a consumer is temporarily unavailable, the producer may continue placing work in the queue instead of failing immediately. Loose coupling improves independence and failure isolation, but resilience still depends on message retention, retries, idempotency, monitoring, capacity, and failure handling.

## Amazon SQS

Amazon Simple Queue Service (**Amazon SQS**) is a managed message queue service for reliable asynchronous communication.

Producers place messages in a queue. Consumers retrieve and process them, then delete successfully processed messages. Messages can remain queued while a consumer is temporarily unavailable, allowing components to be decoupled in time and scaled independently.

## Amazon SNS

Amazon Simple Notification Service (**Amazon SNS**) uses a publish/subscribe model:

```text
Publisher -> SNS topic -> Subscriber
                       -> Subscriber
                       -> Subscriber
```

A publisher sends a message to a **topic**. SNS pushes the message to the topic's subscribed endpoints, supporting immediate notification and fan-out to multiple recipients. Subscribers can include email, SMS, mobile push, HTTP/S endpoints, Amazon SQS queues, and other application or AWS service endpoints.

The main distinction is:

```text
SQS -> Queue and buffer for asynchronous processing
SNS -> Publish/subscribe notification and fan-out
```

SNS and SQS can also be combined, such as subscribing several SQS queues to one SNS topic so each consuming system receives its own buffered copy.

## Amazon EventBridge

Amazon EventBridge is a serverless event-routing service for event-driven architectures. It can receive events from AWS services, custom applications, and supported third-party sources. Rules filter or match events and route them to selected targets such as Lambda functions, SQS queues, SNS topics, or other services. EventBridge is included here as supporting material rather than an exhaustive service study.

## Monoliths and Microservices

- In a **monolithic architecture**, components are packaged or operated as a more tightly integrated unit. A change, resource constraint, or failure can therefore affect a wider part of the application.
- In a **microservices architecture**, components are designed as separate services with defined interfaces. Loose coupling can support independent deployment, scaling, and failure isolation.

Microservices do not automatically guarantee reliability or resilience. They introduce distributed-system concerns such as network failures, data consistency, observability, retries, and operational complexity.

## Useful Mental Models

```text
Physical host
└── Hypervisor
    ├── EC2 instance
    ├── EC2 instance
    └── EC2 instance
```

```text
Traffic
└── ELB
    ├── EC2 instance
    ├── EC2 instance
    └── EC2 instance

EC2 Auto Scaling
└── Adds or removes instances
```

```text
Producer -> SQS queue -> Consumer
```

## Things I Initially Misunderstood

1. **An EC2 instance is the virtual machine itself.** It is not a resource applied to a separate EC2 "service" object.
2. **EC2 support for scaling does not automatically scale a workload.** Scaling mechanisms and policies must be configured.
3. **Stopping an EC2 instance stops its On-Demand instance usage charges, not every related charge.** Attached storage, pricing commitments, and other retained AWS resources can continue to incur costs.
4. **Memory optimized versus storage optimized depends on the bottleneck, not the phrase "large dataset."** An active working set constrained by RAM points to memory optimized; local disk I/O constrained by throughput or latency points to storage optimized.
5. **The 8 GB value in the launch demonstration was storage, not RAM.** It described the size of the `gp3` EBS volume.
6. **Savings Plans and Reserved Instances are both commitment-based discounts, but they are not identical.** They differ in what is committed and how flexibly the discount applies.
7. **Reserved Instances should not be confused with EC2 Capacity Reservations.** The former is primarily a pricing commitment; the latter directly reserves launch capacity in an AZ.
8. **Elasticity does not replace scalability.** Elasticity builds on scalable capacity and adapts it to changing demand.
9. **EC2 Auto Scaling changes instance count; ELB distributes traffic.** They solve related but different problems.
10. **ELB does not create EC2 instances.** An Auto Scaling group or another provisioning mechanism supplies the targets.
11. **EC2 multi-tenancy means separate customer instances can share underlying physical hardware while remaining isolated.** It does not mean multiple users simply share one EC2 instance. The assessment wording was imprecise, but the technically correct definition should be retained.
12. **Loose coupling improves failure isolation but does not automatically make an application resilient.** The complete design must handle retries, duplicates, capacity, monitoring, and failed processing.

## Assessment

**Result: 14/14**

The 14 questions tested:

- Loose coupling
- Accelerated computing instances
- Savings Plans and Spot Instances
- On-Demand pricing
- Amazon EC2 Auto Scaling
- Amazon SNS
- General purpose instances
- Compute optimized instances
- AWS Management Console
- EC2 multi-tenancy
- Reserved Instances
- AMIs
- EC2 Auto Scaling versus ELB
- On-demand cloud resource provisioning

The main learning points were to identify the actual workload bottleneck instead of matching generic phrases such as "large dataset," distinguish Auto Scaling from ELB, and separate pricing commitments from interruption tolerance. Assessment wording must be read carefully, particularly the imprecise multi-tenancy question. These notes keep the technically correct explanation even when a question uses simplified wording.

## What I Can Explain Now

- [x] Explain what an EC2 instance is and how its lifecycle affects compute and related charges.
- [x] Describe virtualization, hypervisor isolation, and EC2 multi-tenancy accurately.
- [x] Select an AMI, instance family, and size based on workload requirements.
- [x] Distinguish memory, storage, compute, accelerated, and balanced workload needs.
- [x] Compare the Management Console, AWS CLI, AWS SDKs, CloudShell, and common instance access methods.
- [x] Explain the configuration choices in a basic EC2 web-server launch.
- [x] Match EC2 pricing and capacity options to predictability, interruption, isolation, licensing, and availability needs.
- [x] Distinguish scalability, elasticity, vertical scaling, and horizontal scaling.
- [x] Explain how Multi-AZ redundancy, EC2 Auto Scaling, CloudWatch, and ELB contribute different capabilities.
- [x] Distinguish Auto Scaling capacity management from ELB traffic distribution.
- [x] Explain tight coupling, loose coupling, message buffering, and cascading failures.
- [x] Compare SQS queues, SNS topics, and EventBridge routing at an introductory level.
- [x] Contrast monolithic and microservices architectures without treating either as an automatic reliability guarantee.

## References

- [Amazon EC2 instance state changes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)
- [Amazon EC2 instance type specifications](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-type-specifications.html)
- [Amazon EC2 billing and purchasing options](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)
- [Amazon EC2 infrastructure security](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/infrastructure-security.html)
- [Amazon EC2 Auto Scaling capacity limits](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-capacity-limits.html)
- [Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Amazon SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [Amazon EventBridge event buses](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-bus.html)
- [AWS CloudShell](https://docs.aws.amazon.com/cloudshell/latest/userguide/working-with-aws-cli.html)
