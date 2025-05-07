# Workflows

## Individual Workflow

1. **Write**: Begin by defining your infrastructure in Terraform configuration files (`.tf` files). These files describe the resources you need, such as virtual machines, storage, and networking configurations, using the HCL (HashiCorp Configuration Language).
2. **Plan**: Run the `terraform plan` command to generate an execution plan. This plan shows what actions Terraform will take to reach the desired state defined in your configuration files. It helps in verifying that the changes are as expected before applying them.
3. **Apply**: Execute the `terraform apply` command to implement the changes described in the plan. Terraform will prompt for confirmation before making any modifications to the infrastructure.
4. **Manage State**: Use `terraform state` commands to inspect and manage the current state of your infrastructure. Terraform keeps track of the state to know what resources exist and their configurations.

## Team Workflow

1. **Version Control**: Store your Terraform configuration files in a version control system (VCS) like Git. This allows multiple team members to collaborate on the infrastructure code, keep track of changes, and revert to previous versions if needed.
2. **Code Review**: Use pull requests to review code changes. Team members can review and discuss proposed changes before merging them into the main branch. This helps maintain code quality and share knowledge across the team. Team members can attach the `plan`output to the pull request for the members to review.
3. **One More Review:** Multiple developers work on multiple branches and hence the plan attached in PR might not be the same after merging it to the main branch. It is at this point that the team asks questions about the potential implications of applying the change. Do we expect any service disruption from this change? Is there any part of this change that is high risk? Is there anything in our system that we should be watching as we apply this? Is there anyone we need to notify that this change is happening?

## Team Workflow with Terraform Cloud

1. **Collaboration**: Terraform Cloud provides a central platform for managing Terraform configurations, states, and variables. It enhances collaboration by allowing teams to work together more effectively.
2. **Runs and Workspaces**: Organize your Terraform configurations into workspaces. Each workspace corresponds to an environment (e.g., dev, staging, production) and maintains its own state file. This helps manage different environments independently.
3. **Remote Operations**: Offload Terraform operations to Terraform Cloud. This ensures consistent execution and state management, as Terraform Cloud handles the infrastructure provisioning and state storage.
4. **Policy Enforcement**: Implement Sentinel policies to enforce governance and compliance checks. Sentinel policies allow you to define rules that must be followed during Terraform operations, ensuring that your infrastructure adheres to organizational policies.
5. **Notifications and Integrations**: Set up notifications to alert team members of important events, such as plan and apply operations. Integrate Terraform Cloud with other tools and services, such as Slack or email, to streamline workflows and improve visibility.

**Takeaway:** Terraform Cloud takes care of managing state, variables and secrets safely in one place inside workspaces. Automated plans are generated for both pull requests and merges.

## References

1. [Core Workflow](https://developer.hashicorp.com/terraform/intro/core-workflow)

## Projects

1. [VCS Based Workflow](https://github.com/apachex692/lad-terraform-cloud-vcs)
2. [CLI Based Workflow](https://github.com/apachex692/lad-terraform-cloud-cli)
