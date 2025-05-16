# Cross-referencing Stacks

In Terraform, cross-referencing stacks is a technique used to create dependencies and share information between different Terraform configurations, which can be essential for managing complex infrastructures that are divided into multiple parts or environments. Here’s a detailed explanation of how cross-referencing stacks work in Terraform:

A "stack" in Terraform typically refers to a set of related resources managed as a single unit. These could be environments (like development, staging, production), different applications, or components of a larger system. Each stack is managed by a separate Terraform configuration.

**Why Cross-Reference Stacks?**

Cross-referencing stacks is crucial *when you need resources in one stack to depend on or interact with resources in another stack. For example, you might have a networking stack that sets up VPCs, subnets, and security groups, and an application stack that deploys servers and databases. The application stack would need information from the networking stack to know where to deploy resources.*

There are several methods to achieve cross-referencing between stacks in Terraform:

**Remote State**

Remote state is the most common method. Terraform stores the state of your managed infrastructure in a remote backend (like AWS S3, HashiCorp Consul, etc.). Other stacks can then read this remote state to get information about resources managed by the first stack.

**Steps to use Remote State:**

1. **Configure Remote State Backend:** Set up a remote backend in the Terraform configuration of the stack whose state you want to share.

   ```hcl
   terraform {
     backend "s3" {
       bucket = "my-terraform-state-bucket"
       key    = "networking/terraform.tfstate"
       region = "us-east-1"
     }
   }
   ```

2. **Data Source to Read Remote State:** In the stack that needs to reference this state, use the `terraform_remote_state` data source to access it.

   ```hcl
   data "terraform_remote_state" "networking" {
     backend = "s3"
     config = {
       bucket = "my-terraform-state-bucket"
       key    = "networking/terraform.tfstate"
       region = "us-east-1"
     }
   }

   resource "aws_instance" "app_server" {
     ami             = "ami-123456"
     instance_type   = "t2.micro"
     subnet_id       = data.terraform_remote_state.networking.outputs.subnet_id
     security_groups = [data.terraform_remote_state.networking.outputs.security_group_id]
   }
   ```

**Output Variables and Manual Sharing**

Another method involves using output variables in one stack and manually sharing these outputs with another stack. This approach is less automated but can be useful for small projects or when you prefer more control over the process.

```hcl
# Network Stack
output "subnet_id" {
  value = aws_subnet.example.id
}

# Compute Stack (another project)
data "terraform_remote_state" "network" {
  backend = "s3"

  config = {
    bucket = "network-stack-bucket"
    key    = "terraform.tfstate"
    region = "us-west-2"
  }
}

resource "aws_instance" "example" {
  subnet_id = data.terraform_remote_state.network.outputs.subnet_id
  # ...
}
```

As seen, we're fetching the output data of the network stack from its state file with the `terraform_remote_state` data block and using it in the compute stack.

**Drawbacks of `terraform_remote_state` Data Source**

1. It can only fetch output data from the root module of the remote state (file).
2. In order to get values of the sub-modules in the remote, you need to do a pass-through of values from the sub-module to root module. It can be done as follows:

   ```hcl
   module "submodule" {
     source = "..."
   }

   output "passthrough_value" {
     value = module.submodule.output.value
   }
   ```

For more information on linking modules, refer [here](https://codeberg.org/apachex692/temp/src/branch/main/providers-modules.md).
