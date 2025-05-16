# Terraform

It is an open source and cloud agnostic IaC tool that is used to provision infrastructure. It uses HCL (HashiCorp Configuration Language), which is a declarative language. Following are the notable features of Terraform:

- Installable Modules
- Plan and Predict Changes
- Dependency Graphing
- State Management
- Provision Infrastructure in Familiar Languages
- 1000+ Providers with Terraform Registry

Terraform Cloud is a SaaS offering from HashiCorp that lets users manage their infrastructure configurations across teams and organizations.

## Terraform Lifecycle

Following are the phases of the Terraform lifecycle:

1. `init`
2. `plan`
3. `validate` (and `fmt`)
   1. Ensures syntax correctness and internal consistency of Terraform configurations. Validates resource definitions, modules, and configurations locally. Commonly used in development or CI/CD pipelines to catch issues before deployment.
   2. Does not interact with remote backends or check actual infrastructure state. Cannot validate input variable types unless explicitly provided. Does not verify external factors like API credentials or resource availability.
4. `apply`
5. `destroy`

**Change Management**

Change management is a standard way to apply, change and resolve conflicts brought by every change to the application. Change automation is an automatic way of creating consistent, systematic and predictable way of managing change request via controls and policies.

Terraform uses **Execution Plans** and **Resource Graphs** to apply and review complex *change sets*. A change set is a collection of commits that represent changes made to a versioning repository. IaC tools use change sets which allows users to see what has been changed by whom over time.

**Execution Plan**

An execution plan is a manual review of what will be added, modified or deleted before you apply those changes to the infrastructure. This is carried out with the `terraform plan` command. If the infrastructure already exists, Terraform reads the state files to create execution plans, else it builds the entire infrastructure.

**Dependency Graph**

Terraform builds a dependency graph from the configurations and reads this graph to generate plans, refresh state and more. A dependency graph is a directed graph which is used to represent inter-dependencies of resources. There are three types of nodes in a dependency graph:

1. **Resource Node (square):** This node represents a resource in the cloud.
2. **Resource Meta-node (ellipse):** This node represents a group of resource and is itself not a resource.
3. **Provider Configuration Node (diamond):** It represents the end of a dependency graph.

**Resource Behavior**

When a user runs `terraform apply` in the CLI, one of the following things happens to a resource:

1. **Create:** The resource doesn’t already exist and available in the configuration. Hence a resource is created.
2. **Update In-place:** Resources whose arguments are changed go through this process.
3. **Destroy:** Resource that exists in state (real world) but no longer exists in the configuration are destroyed.
4. **Destroy and Re-create:** Resources whose arguments have changed, but cannot be updated in-place due to limitations to the API.

## Parts of Terraform

Terraform is logically split into two parts:

1. **Terraform Core:** It is the main application that provisions infrastructure by reading the configurations. It uses RPC to communicate with the plugins.
2. **Terraform Plugin:** It is a codebase that exposes an implementation for automating infrastructure provisioning for a specific provider or a service.

Both are written in Go programming language.

## Resources

1. **Terraform Best Practices:** [Website](https://www.terraform-best-practices.com/)
2. **Terraform: Up and Running:** [Book](https://www.terraformupandrunning.com/)
