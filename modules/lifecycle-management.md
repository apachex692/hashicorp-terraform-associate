# Terraform State Management

In Terraform, the state refers to a representation of the infrastructure managed by Terraform. It stores information about the resources provisioned, their current configuration, and their relationships. This state information is crucial for Terraform to understand the desired state of the infrastructure and to plan and execute changes accurately.

The `terraform state` command is used to interact with the Terraform state. It provides various subcommands to inspect, modify, and manage the state. Some common `terraform state` subcommands include:

1. **`list`:** Lists all resources tracked in the state.
2. **`show`:** Displays detailed information about a specific resource in the state.
3. **`pull`:** Pulls the current state from remote storage, such as Terraform Cloud or a backend configured with remote state.
4. **`push`:** Pushes the local state to remote storage.
5. **`rm`:** Removes one or more resources from the state.
6. **`mv`:** Moves a resource within the state or renames it.

Managing Terraform state is essential for collaboration, as it allows multiple team members to work on the same infrastructure without conflicts. It's also crucial for ensuring the accuracy and consistency of infrastructure changes across different environments.

By using the `terraform state` command, users can inspect the current state of their infrastructure, make necessary modifications, and synchronize the state between local workspaces and remote storage to maintain a single source of truth for their infrastructure configuration.

Note that any non-read operations made to the state will make Terraform create a backup of the state file `terraform.tfstate`. This is done so that recovery is possible in case anything goes wrong. Backups are not created for read-only operations.

## The `mv` State Command

Let's approach this with a use case scenario. Let's say you have an AWS EC2 instance defined in your Terraform configuration named `web_server`, and you want to rename it to `app_server`. You have your infrastructure already running. But you don't want to teardown the VM and recreate it just for changing a name right? In that case, we can change the the name of the resource as follows in the state file.

```bash
terraform state mv aws_instance.web_server aws_instance.app_server
```

We need to make the necessary changes in our codebase too.

**Takeaway:** Change state files when you make changes to your codebase that doesn't change the infrastructure itself but something related to it.

**The `init` Command**

The `init` command in Terraform initializes the project by:

1. Downloading Dependencies
2. Installing Dependencies to `./.terraform/` Directory
3. Create a Dependency Lock File

It is generally the first command you will run for a new Terraform project. *Note that if you modify dependencies, you must run `terraform init` again to apply the changes.*

Following are some commands used commonly:

1. **`terraform init -upgrade`:** It upgrades *all the plugins to the latest version* that complies with the configuration's version constraint.
2. `terraform init -plugin-dir=PATH`: This command will specify the target directory to install plugins.

## The `plan` Command

The `plan` command is used to generate an execution plan. It compares the current state of your infrastructure as described by your Terraform configuration files (`*.tf`) with the desired state defined in those files. It then generates a set of actions required to achieve the desired state. Here’s how it works:

1. **Reading Configuration:** Terraform reads all the configuration files in your current working directory and parses them to understand the desired infrastructure state.
2. **State Comparison:** Terraform compares the desired state described in your configuration files with the current state of your infrastructure, which is stored in the Terraform state file (`terraform.tfstate`). This comparison determines what actions (create, update, or delete) are necessary to make the actual infrastructure match the desired configuration.
3. **Plan Generation:** Based on the comparison, Terraform generates an execution plan detailing all the actions it needs to take to reach the desired state. This plan includes which resources will be added, changed, or destroyed.
4. **Output:** Terraform outputs the execution plan to the console in a human-readable format. It provides detailed information about the actions it will take, such as resource names, types, and the operations (create, update, delete) it will perform.

> By running terraform plan before applying changes, you can review and verify the planned actions. This helps prevent accidental or unwanted modifications to your infrastructure.

To run a plan, you typically execute the following command in your terminal or command prompt:

```bash
terraform plan
```

You can also specify a target directory or plan file to generate the plan for a specific set of configurations.

```bash
terraform plan -out=<filepath>
```

Now, you can apply changes from the state file as follows:

```bash
terraform apply <filepath>
```

_Note that Terraform will apply the changes without your approval in this case. Usually you'd be prompted before applying changes._

**Tips**

- **Always Review:** It's essential to carefully review the plan output to ensure that the intended changes align with your expectations and requirements.
- **Save the Plan:** You can save the plan to a file using the `out` option. This is useful for sharing the plan or applying it later without re-calculating.
- **Automate with CI/CD:** Integrate `terraform plan` into your Continuous Integration/Continuous Deployment (CI/CD) pipelines to validate changes automatically before applying them to production environments.

In summary, the `plan` command in Terraform provides a critical step in the infrastructure provisioning process by allowing you to preview and validate changes before they are applied, promoting safety, predictability, and collaboration within your infrastructure management workflow.

## The `refresh` Command

The `refresh` command is used to update the state of all resources in the Terraform **state file** (`terraform.tfstate`) before making any changes. This ensures that Terraform has the latest information about the current state of your infrastructure. Terraform will actually check the remote infrastructure for this and update the state file.

This command exists because in the past, when one ran the `apply` or `plan` command, the changes were generated solely based on the changes made to the config files and the deployed infrastructure changes were not taken into account. However, it is not suggested nowadays.

This command is kept only for backwards compatibility. Terraform automatically checks the remote for changes and incorporates it into the state file after one approves when running the `plan` or `approve` command.

## The `refresh-only` Option

The `-refresh-only` option is the underlying alias to the above command. The `terraform refresh` command is an alias to:

```bash
terraform apply -refresh-only -auto-approve
```

As you can see from the command, there's a reason the `refresh` command was deprecated because it modifies the state files without providing the review of changes to the user. This is dangerous because let's say the user has entered wrong credentials to authenticate with the provider, then his infrastructure cannot be accessed and Terraform will think there are no resources and overwrite the state file without your permission to create all the resources.

## Replacing Unhealthy Resources

Sometimes, one may encounter an unhealthy or degraded resource. For example, a virtual machine went haywire because of OOM fault. In such cases, what an noob would do is to delete the resource manually and then run `terraform apply` to recreate the resource. But there's a pro way to do it.

`terraform taint <resource-identifier>.<alias>[[<index>]]` command marks a resource as degraded or damaged, indicating that this resource will be destroyed and recreated during the next apply operation. This *can be particularly useful when a resource is in an undesirable or unexpected state, but its configuration hasn’t changed.* Terraform basically forces the recreation of resources even if the configuration matches the current state.

> This command is deprecated and you should use the “-replace” option of terraform apply to achieve the same behavior. This command works only for one resource at a time.

## Importing Resources From Real-world

The `terraform import` command in Terraform is a powerful tool that allows you to import existing infrastructure into your Terraform state. *This is particularly useful when you have resources that were created outside of Terraform but you want to manage them using Terraform going forward.*

The `terraform import` command is used to import existing resources into your Terraform state. By doing so, Terraform gains awareness of the resource's existence and can manage it alongside other resources declared in your Terraform configuration files (`*.tf`).

> Note: Before using the command, you'll have to write the HCL configuration manually so that you can later relate the remote resource with the resource block written in HCL.

1. **Identify Resource:** You specify the resource you want to import by providing its resource type and identifier. The resource type is typically the provider name followed by the resource name, like `aws_instance` for an AWS EC2 instance.
2. **Locate Resource:** Terraform uses the provided resource type and identifier to locate the existing resource in your infrastructure.
3. **Import Resource State:** Terraform retrieves the current state of the identified resource and creates an entry for it in the Terraform state file (`terraform.tfstate`). This allows Terraform to track the resource's attributes and manage its lifecycle.

**Syntax**

The basic syntax for the `terraform import` command is as follows:

```bash
terraform import <resource-type>.<alias> <identifier>
```

- **`<resource-type>`:** The type of resource you want to import (e.g., `aws_instance`).
- `<alias>`: The name of the resource as defined in your Terraform configuration files.
- `<identifier>`: The unique identifier of the existing resource you want to import. For an AWS VM, it is the instance ID.

> Note: The import command will only work with certain resources. Read the plugin documentation before using it.

Let's say you have an existing AWS EC2 instance with the instance ID `i-1234567890abcdef0`, and you want to import it into your Terraform state:

```bash
terraform import aws_instance.example i-1234567890abcdef0
```

**Tips**

- **Resource Attributes (XXX):** After importing a resource, Terraform only knows about the resource's existence and its unique identifier. You'll need to define the resource's attributes (e.g., instance type, AMI ID) in your Terraform configuration files.
- **State Management:** Ensure that you manage Terraform state files securely, especially when importing existing resources, as they contain sensitive information about your infrastructure.
- **Provider Support:** Not all resources are importable, and support for resource import varies across Terraform providers. Refer to the documentation of your provider for specific details.

**Use Cases**

- **Brownfield Environments:** When you have existing infrastructure that was not provisioned using Terraform, you can import those resources to manage them with Terraform.
- **Migration to Terraform:** During migration from other infrastructure management tools or manual setups, `terraform import` helps bring existing resources under Terraform's management without recreating them.

In summary, the `terraform import` command enables you to incorporate existing infrastructure into your Terraform workflow, allowing you to manage all your resources consistently and transparently using Terraform's declarative configuration approach.

## Summary

1. **Importing**: Use the `terraform import` command to import resources that are not in the state file. Note that you must update your configuration files so that you can map the remote resource to an resource block.
2. **Refresh**: Use the `terraform apply --refresh-only` to update the state files with the changes made to the infrastructure that were not through Terraform.
