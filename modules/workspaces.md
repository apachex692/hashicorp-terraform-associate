# Terraform Workspaces

Let's say you're working on a Terraform module to create a user service in Terraform. You write the module and hand it over to the development team. They want to test it in the staging environment first and then deploy to the production. Now the production and deployment configs are different, even through the infra is the same. For example, the team may use `t2.micro` for testing and `m2.large`instance class for production.

In that case, we can create two different `tfvars` file and choose the respective variable file when applying changes. However, the state file remains the same and hence when the config mentioned in `tfvars` change for each environment, Terraform will destroy the previous environment's infra and create a new one with the new config file.

So the problem is having one state file for many environments (production, development, staging etc.). To overcome this, Terraform Workspaces were introduced.

Terraform Workspaces provide a way to manage multiple environments (like development, staging, production) within a single Terraform configuration. They *allow you to maintain separate state files for each environment,* helping to isolate resources and manage different stages of an infrastructure lifecycle. Here’s a detailed explanation:

1. **Isolation of State:** Each workspace has its own state file. This means changes in one workspace do not affect another, ensuring isolation between different environments.
2. **Default Workspace:** When you start using Terraform, a default workspace named `default` is created automatically. This is the initial workspace and acts as the baseline.
3. **Creating and Switching Workspaces:** You can create new workspaces and switch between them using Terraform CLI commands.

## Use Case Example

Imagine you have a Terraform configuration for deploying infrastructure on AWS. You need separate environments for development, staging, and production. Using workspaces, you can manage these environments as follows:

1. **Initial Setup:**

   ```bash
   terraform init
   ```

2. **Creating Workspaces:**

   ```bash
   terraform workspace new dev
   terraform workspace new staging
   terraform workspace new prod
   ```

3. **Switching to Development Workspace:**

   ```bash
   terraform workspace select dev
   ```

4. **Applying Configuration in Development:**

   ```bash
   terraform apply -var-file=<path-tfvars-file>
   ```

5. **Switching to Staging Workspace:**

   ```bash
   terraform workspace select staging
   ```

6. **Applying Configuration in Staging:**

   ```bash
   terraform apply -var-file=<path-tfvars-file>
   ```

**Benefits**

- **Environment Isolation:** Workspaces provide a clear separation of environments, preventing unintended interactions between them.
- **Ease of Management:** Managing multiple environments using the same configuration file simplifies infrastructure as code practices.
- **Consistency:** Ensures that each environment can be managed with consistent configurations while keeping their states isolated.

**Considerations**

- **Scoped State:** Resources created in one workspace do not affect or interfere with resources in another workspace.
- **Naming Conventions:** Consistent naming conventions for workspaces can help in easily identifying and managing environments.
- **Resource Limitations:** Although workspaces help manage environments, ensure that resource limits (like AWS limits) are considered and managed accordingly.

## Storage

Files are stored under the `terraform.state.d` directory in the root location. Inside this folder, there are sub-folders for different workspaces and inside which the actual state files are stored.

**Accessing Workspace Details**

To get the details of the workspace in HCL, you can use the `terraform.workspace` variable.

In summary, Terraform Workspaces are a powerful feature for managing multiple environments using the same infrastructure configuration, providing isolation and ease of management while ensuring consistency across different stages of the infrastructure lifecycle.
