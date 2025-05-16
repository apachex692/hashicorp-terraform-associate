# Introduction

Terraform is a technology produced by HashiCorp and it is an Infrastructure as Code (IaC) tool that is **declarative** and **cloud agnostic.**

- **Declarative:** This means that you don’t define the steps to provision infrastructure, rather you say what you need to provision.
- **Cloud Agnostic:** This means that the tool is not biased towards one cloud platform and can work equally well across all the CSPs.

There are in total 57 questions asked in the exam. The pass percentage is 70%, which means you can afford to get 17 questions wrong tops. There is no penalty for wrong questions. The format of the questions can be multiple choice or fill-in-the-blank. The duration of the exam is around 1 hour with a seat time of 1 and a half hour. The validity of the exam is for 24 months and this certification is based on the Terraform v1.6.0. The cost of the exam is around $70.

## Infrastructure as Code

The problem with manually configuring infrastructure is that there are chances of misconfiguring a service through human error. It is also hard to manage and transfer the infrastructure across CSPs. To overcome this, IaC is used. IaC is a way of writing configuration scripts to *automate*the creation, updation and deletion of cloud infrastructure. IaC is the *blueprint* of your infrastructure and it allows you to version and share your cloud infrastructure. There are two types of IaC paradigms:

- **Declarative:** It is considered explicit. You get to define everything you want explicitly. It is considered to be verbose and hence there are zero changes of misconfiguration. It uses scripting languages such as JSON, YAML etc.
- **Imperative:** You say what you want and the rest is filled in. This type of infrastructure provisioning is considered implicit as the tool takes care of most of the configuration. It uses programming languages and SDKs written in Python, Ruby, JavaScript etc.

HCL (HashiCorp Configuration Language) is a combination of declarative and imperative paradigm, meaning it has support for declarative like features and imperative language like features like complex data structures etc.

**Difference Between IaC and Configuration Management Tools**

Infrastructure as Code (IaC) tools like Terraform and configuration management tools like Puppet and Ansible serve different purposes in managing infrastructure.

IaC tools focus on automating the creation, modification, and deletion of infrastructure resources using code. They provide a way to define infrastructure as configuration files, allowing for version control, collaboration, and repeatability. With IaC, you can define the desired state of your infrastructure and let the tool handle the provisioning and management of resources.

On the other hand, configuration management tools like Puppet and Ansible focus on maintaining the desired state of already provisioned servers. They are used to configure and manage software, packages, and services on existing infrastructure. These tools enable system administrators to ensure consistency and enforce desired configurations across multiple servers.

In summary, IaC tools focus on provisioning and managing infrastructure resources, while configuration management tools focus on configuring and maintaining the software and services running on that infrastructure.

## Infrastructure Lifecycle

It is the idea of clearly defined and distinct work phases which are used by DevOps Engineers to plan, design, build, test, deliver, maintain and retire cloud infrastructure. It has three phases:

- **Day-0:** Plan and Design
- **Day-1:** Develop and Iterate
- **Day-2:** Go Live and Maintain

Note that “day” here doesn’t mean the usual 24 hour span but is just a broad way of defining the phases of the infrastructure lifecycle. Following are the advantages of using IaC tools in infrastructure lifecycle:

1. **Reliability:** IaC makes changes that are idempotent, consistent, repeatable and predictable.
2. Manageability
3. Sensibility

**Provisioning:** Provisioning is the act of preparing a server with systems, data, software and make it ready for network operations. Using configuration management tools like Puppet, Ansible etc., we can provision a server.

**Deployment:** Deployment is the act of delivering a version of your application to run an already provisioned server. It can be performed with tools like AWS CodePipeline, Jenkins, GitHub Actions etc.

**Orchestration:** It is the act of coordinating multiple systems or services. It is a common term when working with micro-services, containers and Kubernetes.

## Configuration Drift

Configuration drift happens when the provisioned infrastructure has an unexpected configuration change due to team members adjusting the configuration options without notice, malicious actors performing changes to the infrastructure or side effects from APIs, SDKs or CLIs. There are three ways to **detect** configuration drifts in the infrastructure:

1. Using a compliance tool that can detect drifts in configuration. Tools like AWS Config, Azure Policies and GCP Security Health Analysis can do this.
2. Using the built-in support for drift detection. This is provided by tools like AWS CloudFormation Drift Detection etc.
3. Storing the expected state to a file. This can be done in Terraform and is known as a state file.

Configuration drifts can be **corrected** in the following ways:

1. A compliance tool that can remediate misconfiguration.
2. Terraform `refresh` and `plan` commands.
3. Manually correcting the configuration (not recommended).
4. Tearing down and setting-up the infrastructure again.

Configuration drifts can be **prevented** in the following ways:

1. Using **immutable infrastructure** which enforces the creation and destruction of infrastructure rather reusing it. This includes blue-green deployment strategy.

> **NOTE**
> Blue-green deployment is a strategy that involves running two identical environments (blue and green) to reduce downtime and risk during application updates by directing traffic to the new environment (green) while keeping the old environment (blue) as a fallback.

## GitOps

It is when you take your IaC configurations and version control it with a tool like `git` to introduce a formal process to review and accept changes to infrastructure and once that configuration is accepted, the change to infrastructure is triggered automatically. It usually follows a path like this:

Update Infrastructure Configuration (HCL files) → Commit Changes to Development Branch → Push to Remote → Review Changes → Merge Pull Request → Trigger GitHub Action (or other triggers) → Make Changes to Infrastructure

## HashiCorp

HashiCorp is a company specializing in open source tools that support the development and deployment of large-scale service-oriented software installations. Following are the tools and services HashiCorp provides:

- **HashiCorp Cloud:** Unified platform to access various products and services of HashiCorp. Their services are cloud agnostic. They are highly suited for multi-cloud workloads.
- **Boundary:** It provides secure access to remote systems based on trusted identity.
- **Consul:** It is a service discovery platform. It provides a fully-featured service mesh for secure service segmentation across any cloud or runtime environment and a distributed key-value storage for application configuration.
- **Nomad:** It is used for scheduling and deployment of tasks across worker nodes in a cluster.
- **Packer:** It is a tool for building virtual machine images for deployment.
- **Terraform:** It is a IaC tool that enables provisioning and adapting virtual infrastructure across all major CSPs.
- **Terraform Cloud:** It is a place to store and manage IaC configurations in the cloud and share it across the team.
- **Vagarant:** It is used to build and maintain reproducible software development environments via virtualization software (similar to containerization but at hardware level instead of OS level).
- **Vault:** It is a tool used to manage secrets, provide identity based access, encrypting application data and auditing of secrets for applications, systems and users.
- **Waypoint:** It is a modern workflow used to build, deploy and release across platforms.
