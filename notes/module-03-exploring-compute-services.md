# Module 3 - Exploring Compute Services

## Choosing a Compute Model

AWS provides several ways to run workloads. Each option offers a different balance of infrastructure control, service abstraction, operational responsibility, and workload fit. The objective is not to move every application toward the most abstract or serverless option. It is to select the compute model that satisfies the workload's technical and operational requirements.

A useful starting comparison is:

| Model | Customer focus | AWS focus | Example |
| --- | --- | --- | --- |
| Customer-managed compute | Guest operating system, software, scaling configuration, networking, and workload operation | Facilities, physical hosts, networking, and virtualization | Amazon EC2 |
| Managed service | Service-specific configuration, permissions, data, and integration | More of the service operation, scaling, and availability | ELB, Amazon SQS, Amazon SNS |
| Fully managed or serverless | Code or workload definition, configuration, permissions, data, and application security | Underlying infrastructure, server lifecycle, managed platform, scaling, and service availability | AWS Lambda |

The course describes EC2 as an **unmanaged** option relative to more abstract compute services. This does not mean AWS has no responsibility: AWS still manages the physical infrastructure and hypervisor. It means the customer manages more of the instance and guest operating system.

**Fully managed does not mean AWS manages everything.** The customer can still be responsible for:

- Application code and dependencies
- IAM permissions
- Data access and classification
- Secrets
- Service configuration
- Application security

## Shared Responsibility and Abstraction

As a service becomes more abstracted, AWS generally operates more of the technology stack and the customer manages less infrastructure. This changes the responsibility boundary rather than removing customer responsibility. The exact division remains service-specific. With EC2, the customer patches the guest operating system. With Lambda managed runtimes, AWS operates the runtime infrastructure, while the customer still selects a supported runtime, maintains function code and dependencies, and configures permissions and runtime-update behavior appropriately.

## AWS Lambda

AWS Lambda is a serverless compute service that runs code in response to events. It is an example of **Function as a Service (FaaS)**: the unit of deployment and execution is a function rather than a customer-managed server.

Key characteristics include:

- A **function** contains code and configuration for a specific piece of application logic.
- A **trigger** connects an event source to the function so relevant events can cause an invocation.
- Lambda creates and manages execution environments, scales concurrent execution in response to demand, and operates the underlying servers.
- AWS manages the service's infrastructure availability across Availability Zones. The availability of the complete application still depends on its configuration and downstream dependencies.
- The customer does not provision servers for standard Lambda functions.
- A **runtime** supplies the language-specific environment used to run the function. AWS provides managed runtimes including Python, Java, and Node.js, and custom runtimes are also possible.
- Function memory is configurable, and Lambda allocates CPU and other resources in proportion to that setting.
- Lambda integrates with many AWS services as event sources and destinations.

A single Lambda invocation can run for a maximum of **15 minutes**. This is an execution-duration limit for one invocation, not a general "15-minute request limit."

Automatic scaling and managed infrastructure do not remove service quotas, concurrency behavior, application design, security configuration, or cost considerations.

## Lambda Use Cases

Lambda is well suited to short, event-driven units of work such as:

- Processing an image after it is uploaded
- Handling application events
- Generating notifications
- Running background processing
- Processing messages from queues
- Implementing lightweight API or backend logic

A task being quick is not enough to make Lambda appropriate. The decision should ask:

- Is the task triggered by an event?
- Can it be separated cleanly from the rest of the application?
- Does it require long-running or continuous execution?
- Does it require persistent in-process state?
- Does it require low-level control of the operating system, host, or network?
- Do its latency characteristics fit an invocation-based service?

An online game provides a useful conceptual contrast. Persistent gameplay, combat, or session processing may fit continuously running compute such as EC2 or another persistent service. Discrete events such as processing an achievement, generating a notification, or applying a background update may fit Lambda. This is an architectural illustration, not a claim about how any real game is implemented.

## Lambda Versus EC2

| Amazon EC2 | AWS Lambda |
| --- | --- |
| Runs a virtual machine. | Runs function code in managed execution environments. |
| Customer manages the guest OS and installed software. | AWS manages the servers and runtime infrastructure. |
| Provides more operating system, network, and host-level configuration control. | Provides less infrastructure control and more service abstraction. |
| Suitable for persistent, long-running, stateful, or highly customized workloads. | Suitable for event-driven, short-lived, separable units of work. |
| Customer configures scaling and availability architecture. | Service scaling and infrastructure availability are managed, within configuration and quota boundaries. |

Neither service has universally more use cases or is inherently better. Workload requirements determine the fit.

## Lambda and Amazon SQS Demonstration

The demonstrated message-processing flow was:

```text
Producer
  ↓
Amazon SQS queue
  ↓
Lambda trigger
  ↓
Lambda function
  ↓
Amazon CloudWatch Logs
```

- **Amazon SQS:** Queues messages, separates the producer from the consumer, and retains messages until they are processed or reach their retention limit.
- **Trigger:** An SQS event source mapping polls the queue and invokes the Lambda function when relevant messages are available.
- **Lambda function:** Receives messages and runs the application logic that processes them.
- **Execution role:** An IAM role assumed by the Lambda function. It grants the permissions required to access AWS services. The demonstration used permissions that allowed Lambda to read from SQS.
- **Amazon CloudWatch Logs:** Records function output and logging, making it possible to verify processing and troubleshoot execution. The execution role also needs suitable logging permissions.

Lambda does not automatically receive access to other AWS services. Its execution role must grant the required permissions, preferably following least privilege.

## Container Fundamentals

A **container** is an isolated process environment that packages an application with the components it needs to run consistently. A container image can include application code, a runtime, libraries, dependencies, and configuration defaults. This portability helps address the "works on my machine" problem by keeping the application environment consistent across development, testing, and deployment.

Isolation is not an automatic security guarantee. Container images, runtime permissions, host configuration, networking, secrets, dependencies, and orchestration controls must still be secured.

### Containers Versus Virtual Machines

| Virtual machine | Container |
| --- | --- |
| Virtualizes hardware through a hypervisor. | Uses operating-system-level isolation and shares the host kernel. |
| Typically includes a complete guest operating system. | Packages the application and dependencies without a separate full guest OS. |
| Heavier and generally slower to start. | Lighter and generally faster to start. |
| Provides a separate OS boundary. | Provides process isolation within the container runtime and host OS model. |

### Container Image Versus Container

- A **container image** is an immutable, read-only package or blueprint containing the application and its dependencies.
- A **container** is a running instance created from that image, normally with a writable runtime layer.

One image can be used to start multiple containers with the same initial application environment.

## Container Orchestration

Managing a few containers manually may be possible, but it becomes difficult as the number of applications, containers, and hosts grows. An orchestrator coordinates work including:

- Starting and stopping containers
- Deploying applications
- Scaling container counts
- Monitoring health
- Replacing failed containers
- Rolling out updates
- Placing workloads across available hosts

Orchestration manages container workloads. It still requires correct application design, resource settings, permissions, networking, and observability.

## AWS Container Service Categories

AWS container services occupy distinct layers: ECR is a registry, ECS and EKS provide orchestration, and EC2 or Fargate supplies compute. The complete workflow appears under “Useful Mental Models.”

## Amazon ECR

Amazon Elastic Container Registry (**Amazon ECR**) is a managed container image registry. It stores and manages OCI-compatible container images and artifacts. Standard container tools can push images to ECR and pull images when workloads are deployed.

ECR stores images. It does not schedule, orchestrate, or run containers. It is the natural AWS-native registry choice for ECS and EKS, but compatible architectures can use other registries.

## Amazon ECS

Amazon Elastic Container Service (**Amazon ECS**) is an AWS-native managed container orchestration service. It deploys, manages, and scales containerized workloads and integrates closely with AWS identity, networking, monitoring, registry, and load-balancing services.

ECS can place container workloads on Amazon EC2 instances or AWS Fargate. ECS provides orchestration; the selected capacity option provides the compute on which containers run.

## Amazon EKS

Amazon Elastic Kubernetes Service (**Amazon EKS**) is a managed Kubernetes service. AWS manages the Kubernetes control plane, while responsibility for worker compute depends on whether the workload uses EC2, Fargate, or another supported option.

EKS is useful when Kubernetes itself is a requirement. It supports Kubernetes APIs, ecosystem tools, portable deployment definitions, custom controllers and operators, and consistency across Kubernetes environments, including supported hybrid patterns. It can run workloads on EC2 or Fargate.

## ECS Versus EKS

| Amazon ECS | Amazon EKS |
| --- | --- |
| AWS-native orchestration model. | Kubernetes-based orchestration model. |
| Generally lower operational complexity. | Generally introduces more concepts and operational complexity. |
| Strong integration with AWS services. | Combines AWS integration with Kubernetes APIs and ecosystem tooling. |
| Good fit when Kubernetes-specific capabilities are not required. | Good fit when Kubernetes portability, tooling, controllers, operators, or organizational standardization are requirements. |

Both services can support large-scale workloads. ECS is not only for small systems, and EKS is not only for large systems. EKS is not a higher level to "strive" toward. The deciding factor is whether Kubernetes capabilities justify the additional complexity.

Migration from ECS to EKS is possible, but it is not a simple switch. Their workload abstractions, deployment definitions, operational workflows, configuration, and tooling differ, so migration requires planning and changes.

## AWS Fargate

AWS Fargate is a serverless compute engine for containers that works with both ECS and EKS. AWS provisions and manages the underlying servers. The customer focuses on the task or pod definition, container configuration, requested CPU and memory, networking, permissions, and application behavior.

Fargate removes the need to provision and manage EC2 hosts for containers, but it is not always the best compute choice. EC2 may be preferable when a workload needs:

- More control over hosts
- Custom AMIs
- Specialized hardware
- Specific host networking or system configuration
- Capabilities not supported by the relevant Fargate mode
- A cost and operational model that favors directly managed capacity

The trade-off is reduced server management with Fargate versus greater infrastructure control and responsibility with EC2.

## ECR and Fargate

ECR and Fargate are not alternatives. A typical architecture can store an image in ECR, use ECS or EKS to orchestrate it, and use Fargate to supply compute. ECR is convenient and well integrated, but it is not conceptually mandatory when another compatible registry meets the requirements.

## AWS Elastic Beanstalk

AWS Elastic Beanstalk simplifies the deployment and management of web applications. The customer supplies application code and configuration, and Beanstalk provisions and coordinates supporting resources. Depending on the environment, these can include EC2 instances, load balancing, Auto Scaling, and health monitoring.

Beanstalk is useful for web applications, APIs, and application backends when the customer wants managed deployment and lifecycle coordination without manually connecting every infrastructure component. The underlying resources remain visible and configurable. Elastic Beanstalk is not a serverless compute service.

## AWS Batch

AWS Batch is a managed batch computing service that schedules jobs, provisions or selects compute capacity, and scales resources for batch workloads. It is designed for non-interactive, compute-heavy work that can be queued and often processed in parallel.

Example workloads include:

- Scientific simulations
- Financial risk analysis
- Media transcoding
- Data processing
- Genomics
- Large-scale parallel jobs

The assessment mental model is:

```text
Large-scale parallel jobs
+ No real-time interaction
+ Automatic scheduling and scaling
= AWS Batch
```

## Amazon Lightsail

Amazon Lightsail is a simplified cloud and virtual private server platform. It provides bundled access to virtual servers, storage, managed databases, networking, and related hosting features with predictable plan pricing and a simpler AWS experience.

It is a practical fit for blogs, basic websites, low-traffic applications, development and testing environments, small business workloads, and straightforward hosting where the broader flexibility of building directly with EC2 services is unnecessary.

## Elastic Beanstalk Versus Lightsail

| AWS Elastic Beanstalk | Amazon Lightsail |
| --- | --- |
| Focuses on application deployment and lifecycle management. | Focuses on simplified, packaged hosting. |
| Provisions AWS infrastructure behind an application. | Provides a more opinionated set of virtual server, storage, database, and networking options. |
| Useful when managed application deployment and scaling are required. | Useful for straightforward workloads where lower complexity and predictable plans are priorities. |

## AWS Outposts

AWS Outposts is a managed hybrid cloud solution that extends AWS-managed infrastructure, supported services, APIs, and tools into an on-premises location. It provides a more consistent AWS operating experience across an AWS Region and local facilities.

Outposts can support:

- Low-latency processing close to local systems or data
- Local data processing or residency requirements
- Regulatory and compliance constraints
- Hybrid architectures
- Integration or modernization of workloads that must remain local

Outposts does not simply leave an existing data center unchanged. AWS-owned and AWS-managed infrastructure is installed at the customer location, while the customer still provides and manages site requirements such as space, power, networking, and connectivity.

## Useful Mental Models

```text
EC2
  ↓
Managed services
  ↓
Serverless / fully managed

More AWS operational responsibility
Less customer infrastructure management
```

```text
Event
  ↓
Lambda trigger
  ↓
Lambda function
  ↓
Result or downstream service
```

```text
Container image
  ↓
ECR
  ↓
ECS or EKS
  ↓
EC2 or Fargate

ECR         = Store
ECS / EKS   = Orchestrate
EC2/Fargate = Run
```

## Things I Initially Misunderstood

1. **Fully managed does not mean the customer has no responsibilities.** Code, permissions, data, secrets, configuration, and application security can remain with the customer.
2. **The goal is not to move every workload from EC2 to fully managed or serverless services.** The correct abstraction depends on workload requirements.
3. **Lambda is not simply better than EC2.** Lambda fits event-driven function execution; EC2 provides persistent compute and more infrastructure control.
4. **A short task is not automatically a Lambda use case.** Persistent state, continuous processing, latency behavior, event boundaries, and control requirements matter.
5. **Quick individual events do not automatically make continuous online-game combat or session logic a good Lambda fit.** Persistent gameplay processing and discrete background events have different requirements.
6. **Lambda's 15-minute limit applies to one invocation.** It is not a general limit on every request associated with an application.
7. **Lambda still needs correct IAM permissions to access services such as SQS.** An execution role grants only the actions allowed by its policies.
8. **ECS and EKS are orchestration services; EC2 and Fargate are compute options.** They occupy different layers of a container architecture.
9. **ECR stores container images; it does not run containers.** Running requires compute and normally an orchestration layer.
10. **ECR and Fargate are not alternatives.** One stores images and the other supplies compute for running containers.
11. **ECS is not just for small workloads, and EKS is not just for large workloads.** Both can operate at large scale.
12. **EKS is not automatically better than ECS.** Kubernetes should be chosen when its ecosystem, APIs, portability, or tooling is required.
13. **Using EKS without a Kubernetes requirement can add unnecessary complexity.** Service choice should follow requirements rather than prestige.
14. **Fargate is not always the best compute choice.** EC2 can be better when host-level control, specialized hardware, custom system configuration, or a different cost model is required.
15. **ECR is a natural AWS-native registry choice but is not conceptually mandatory for every container architecture.** Other compatible registries can exist.

## Assessment

**Result: 8/8**

The eight questions tested:

- Customer responsibility with Lambda
- Lambda event-driven use cases
- Container orchestration
- Container portability and dependency consistency
- Combining ECR, EKS, and Fargate
- AWS Batch
- Amazon Lightsail
- The serverless service model

The assessment reinforced that application code remains the customer's responsibility with Lambda and that image uploads triggering processing are a strong Lambda pattern. Orchestration points to ECS or EKS, not ECR; packaging dependencies explains container consistency; a Kubernetes requirement points toward EKS; and avoiding server management for containers points toward Fargate.

For service selection, large parallel non-interactive jobs point toward AWS Batch, simple low-traffic hosting points toward Lightsail, and code-focused workloads with no server management point toward serverless compute. The wording should be matched to the workload's direct requirement rather than to a general preference for greater abstraction.

## What I Can Explain Now

- [x] Distinguish customer-managed, managed, and serverless compute models.
- [x] Compare EC2 and Lambda without treating either as universally better.
- [x] Identify when Lambda is and is not appropriate.
- [x] Explain events, triggers, functions, runtimes, and the per-invocation duration limit.
- [x] Explain Lambda execution roles and service permissions.
- [x] Trace an SQS message through a Lambda trigger, function, and CloudWatch Logs.
- [x] Define containers and compare them with virtual machines.
- [x] Distinguish a container image from a running container.
- [x] Explain why container orchestration is needed.
- [x] Separate the roles of ECR, ECS, EKS, EC2, and Fargate.
- [x] Compare ECS and EKS based on requirements rather than workload size or prestige.
- [x] Compare EC2 and Fargate as container compute options.
- [x] Explain the roles of Elastic Beanstalk, AWS Batch, Lightsail, and Outposts.
- [x] Select a compute service by considering duration, interaction model, control, portability, orchestration, location, and operational responsibility.

## References

- [AWS Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [AWS Lambda runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
- [Using Lambda with Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
- [Managing permissions in AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-permissions.html)
- [Choosing an AWS container service](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/choosing-aws-container-service.html)
- [Amazon Elastic Container Service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [Amazon Elastic Kubernetes Service](https://docs.aws.amazon.com/eks/latest/userguide/)
- [Amazon Elastic Container Registry](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
- [AWS Elastic Beanstalk](https://docs.aws.amazon.com/elastic-beanstalk/)
- [AWS Batch](https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html)
- [Amazon Lightsail](https://docs.aws.amazon.com/lightsail/latest/userguide/what-is-amazon-lightsail.html)
- [AWS Outposts](https://docs.aws.amazon.com/outposts/latest/userguide/what-is-outposts.html)
