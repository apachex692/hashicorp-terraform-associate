# Providers & Modules

## Providers

Providers are organization who provide plugins for Terraform that allows you to interact with their platform. Providers comes in three tiers:

1. **Official:** Published by the company that owns the technology or service.
2. **Verified:** Actively maintained, up-to-date and compatible with both Terraform and providers.
3. **Community:** Published by a community member but no guarantee for maintenance or compatibility.

> Important (exam question): Only official and verified modules are displayed when you search in the searchbar of the Terraform Registry website.

Providers and modules used in the project can be listed with the `terraform providers` command.

**Setting Aliases for Providers**

Aliases are used for providers in Terraform to differentiate between multiple instances of the same provider configuration. This is useful when you have different configurations for the same provider, such as when interacting with multiple regions or environments.

To set an alias for a provider, you can use the `alias` parameter in the provider block. Here's an example:

```hcl
provider "aws" {
  alias  = "east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

resource "aws_instance" "example" {
  provider = aws.east
}
```

In this example, we have two provider configurations for the AWS provider, with different regions specified. We assign aliases "east" and "west" to differentiate between them.

When using the provider in a resource block, you can specify the alias using the `provider` parameter. In the `aws_instance` resource block, we set the provider to `aws.east`, indicating that we want to use the provider configuration with the "east" alias.

By using aliases, you can easily manage and reference different provider configurations within your Terraform project.

## Modules

In Terraform, `module` and `provider` blocks serve distinct purposes and are essential for organizing and managing infrastructure as code.

A `module` block in Terraform is used to encapsulate a group of resources that are used together. Modules are a way to organize and reuse configurations by creating reusable, composable, and maintainable infrastructure components.

- **Definition:** A module is a container for multiple resources that are used together. A root module is the main configuration used to deploy infrastructure, while child modules can be called within the root module or other modules.
- **Usage:** Modules allow you to abstract and encapsulate complex logic and reuse it across different parts of your infrastructure.
- **Syntax:**
  ```hcl
  module "example" {
    source = "./path_to_module"
    variable1 = "value1"
    variable2 = "value2"
  }
  ```
- **Source:** The `source` argument specifies the location of the module's source code, which can be a local path, a Git repository, or a Terraform Registry module.

### Cross Referencing Modules (advanced)

We can reference one module in another module in Terraform. This can be useful when we don't have a Terraform configuration in the same directory as the root configuration (module). We can reference a module in another module as follows:

```hcl
module "another_module" {
  source         = "..."

  # Module Inputs
  submodule_var1 = true
  submodule_var2 = ""
}
```

As seen, we can include the module in our configuration and also we can pass values to it's variable from here itself. We can access the resources inside the module by using the `module.another_module.<whatever>` format.

Note that explicitly referencing like this is only useful for a module in another directory or some remote location. For a sub-module (module inside a root module placed inside a directory in the same level as the root module), this isn't necessary because Terraform automatically considers it while performing operations and varibles can be passed through envs and tfvars. Although, sometimes we may need to (`terraform_remote_state` scenario) pass-through output data from sub-module to root module. In that case, we can include the sub-module in the root module and retrieve the output of it like follows:

```hcl
# Network Stack (sub-module)
output "subnet_id" {
  value = aws_subnet.example.id
}

# Compute Stack (root module)
module "network_stack" {
  source = "..."
}

resource "aws_instance" "example" {
  subnet_id = module.network_stack.network.outputs.subnet_id
  # ...
}
```

In this case, we passed output data from sub-module to root module. Pretty cool right?

## Difference Between Providers and Modules

Providers are plugins that are written by the respective providers that allows us to interact with their services through Terraform. They provide all the resource blocks, data blocks etc. Now, a module on the other hand, is a collection of resources built upon providers that makes our life easier by writing resources that does the task we want. For example, instead of writing code to create an EC2, VPC, security group, instance key etc. to host a web server, I can simply write:

```hcl
module "web_server" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "2.0.0"

  instance_count = 2
  ami            = "ami-0c55b159cbfafe1f0"
  instance_type  = "t2.micro"
  subnet_id      = "subnet-abc12345"
}
```

And that's it. My resource will be automatically created for me.
