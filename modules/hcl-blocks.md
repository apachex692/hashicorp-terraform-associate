# HCL Blocks

HashiCorp Configuration Language (HCL) is a configuration language developed by HashiCorp. It is used to write configuration files for various HashiCorp tools such as Terraform, Consul, and Vault.

HCL has a simple and intuitive syntax that allows users to define configuration settings in a human-readable format. It uses blocks, braces, and key-value pairs to represent configuration elements. Here is an example of the syntax:

```hcl
block_type "<block-name>" "<alias>" {
  key1 = value1
  key2 = value2
}
```

In addition to HCL, HashiCorp tools also support JSON format for configuration. JSON (JavaScript Object Notation) is a widely used data interchange format. It uses a key-value pair structure similar to HCL, but with a different syntax. Here is an example of JSON configuration:

```hcl
{
  "key1": "value1",
  "key2": "value2"
}
```

When using HCL or JSON for configuration, the file extensions commonly used are `.tf` for HCL files and **`.tf.json` for JSON files.**

## Terraform Settings

The `terraform` block in HCL is used to define Terraform-specific settings and configuration for a Terraform project. It is typically placed at the beginning of the HCL file. Here are some common attributes that can be included within the `terraform` block:

```hcl
terraform {
  required_version = "x.x.x"

  backend "backend_type" {
  }

  required_providers {
    provider_name = {
      source  = "provider_source"
      version = "x.x.x"
    }
  }

  variable "variable_name" {
  }

  data "data_source" {
  }

  resource "resource_type" "resource_name" {
  }
}
```

- `required_version`: Specifies the minimum required version of Terraform for the project.
- `backend`: Defines the back-end type and configuration settings for storing Terraform state.
- `required_providers`: Specifies the required Terraform provider(s) and their versions.
- `variable`: Declares input variables that can be used within the Terraform configuration.
- `data`: Configures a data source to retrieve data to be used within the Terraform configuration.
- `resource`: Specifies a resource type and its configuration settings to be managed by Terraform.

These are just a few examples of the attributes that can be included within the `terraform` block. The specific attributes used will depend on the requirements of the Terraform project.

## Blocks

In Terraform, a block is a configuration element that defines a particular resource or behavior. There are several types of blocks in Terraform, including provider blocks, resource blocks, data blocks, variable blocks, and output blocks.

1. **`provider` Block:** It specifies the cloud provider or service that Terraform will use, for example:
    
    ```jsx
    provider "aws" {
      region = "us-west-2"
    }
    ```
    
2. **`resource` Block:** It defines a resource that will be managed by Terraform, like an EC2 instance or a security group, for example:
    
    ```jsx
    resource "aws_instance" "example" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t2.micro"
    }
    ```
    
3. **`data` Block:** It retrieves information from a remote system or data source, like an AWS S3 bucket or an external API, for example:
    
    ```jsx
    data "aws_s3_bucket" "example" {
      bucket = "example-bucket"
    }
    ```
    
4. **`variable` Block:** It declares input variables that can be used to parameterize your Terraform configuration, for example:
    
    ```jsx
    variable "region" {
      type    = string
      default = "us-west-2"
    }
    ```
    
5. **`output` Block:** It defines values that are readable and accessible after Terraform applies the configuration, for example:
    
    ```jsx
    output "instance_ip" {
      value = aws_instance.example.public_ip
    }
    
    ```
    
6. **`locals` Block:** These variables are used to store intermediate or computed values that can be used within the configuration, for example:
    
    ```jsx
    locals {
      service_name = "forum"
    }
    ```
    

### `variable` Block

The `input` block in Terraform HCL allows you to define input variables that can be used to parameterize your Terraform configuration. Input variables are used to accept dynamic values from users or other parts of the Terraform configuration.

Here are the attributes that can be included within the `input` block:

- **`description` (optional):** This attribute allows you to provide a description or explanation of the input variable. It helps users understand the purpose and expected value of the variable.
- **`type` (required):** The `type` attribute specifies the data type of the input variable. It can be one of the supported Terraform data types such as `string`, `number`, `bool`, `list`, `map`, `set`, or a user-defined type.
- **`default` (optional):** The `default` attribute specifies a default value for the input variable. If no other value is provided for the variable, Terraform will use this default value. The default value must match the data type specified in the `type` attribute.
- **`validation` (optional):** The `validation` attribute allows you to define validation rules for the input variable. You can specify constraints such as maximum and minimum values, regular expressions, or custom validation functions.
- **`sensitive` (optional):** The `sensitive` attribute can be set to `true` to mark an input variable as sensitive. When a sensitive variable is displayed in the Terraform output or logs, its value will be replaced with ``.

Here is an example of an `variable` block with all the attributes:

```jsx
variable "example_variable" {
  description = ""
  type        = string
  default     = "default_value"

  validation {
    condition     = length(var.example_variable) > 0
    error_message = ""
  }

  sensitive   = false
}
```

In this example, the variable `example_variable` has a description, `string` type, a default value of `default_value`, a validation rule to ensure it is not empty, and is marked as not sensitive.

Using the `variable` block, you can define flexible and reusable Terraform configurations by allowing users to provide customized values through input variables.

### Passing Values to `variable` Block

The `variable` block in Terraform allows you to define input variables that can be used to parameterize your Terraform configuration. There are several ways to pass values to the `variable` block in Terraform:

1. **Environment Variables:** You can set environment variables with the naming convention `TF_VAR_variable_name` to pass values to input variables. For example, to pass a value to an input variable named `region`, you can set the environment variable `TF_VAR_region` with the desired value.
2. **Command-line Flags:** You can pass values to input variables using command-line flags when running the `terraform` command. For example, you can use the `var` flag followed by the variable name and value to pass a value to an input variable. For instance, `terraform apply -var="region=us-west-2"`.
3. **Terraform Variable Files:** You can create Terraform variable files with `.tfvars` extension and specify input variable values in these files. Terraform automatically loads the `terraform.tfvars` file in the current directory. You can also specify a variable file explicitly with the `var-file` flag when running the `terraform` command. You can also use the `.auto.tfvars` as the file extension to automatically load multiple variable files.
4. **Default Values:** You can define default values for input variables directly in the `input` block. If no other value is passed to the input variable, Terraform will use the default value. For example:
    
    ```jsx
    variable "region" {
      type      = string
      default   = "us-east-1"
      sensitive = false
    }
    ```
    

These are the commonly used methods to pass values to the `input` block in Terraform. By using these methods, you can make your Terraform configurations more flexible and reusable.

### Precedence of Passing Values to Variables

When passing values to Terraform input blocks, the precedence is as follows:

1. Command-line Flags (`var`) (highest precedence)
2. Terraform Variable Files (`.tfvars` or `.tfvars.json`)
3. Environment Variables (`TF_VAR_*`)
4. Default Values (lowest precedence, gets overridden by above methods)

For example, let’s say for a variable, both command-line flag and environment variable is specified. Then, the command-line flag will be used as it has higher precedence.

It's important to note that when using multiple methods to pass values, the method with the highest precedence will take precedence over others.

### `output` Block

The `output` block in Terraform HCL allows you to define values that are readable and accessible after Terraform applies the configuration. Output values can be useful for displaying important information, such as IP addresses, resource identifiers, or computed values.

Here is the syntax for defining an `output` block:

```jsx
output "<output-name>" {
  value = <expression>
}
```

- `<output-name>`: This is the name of the output value, which can be used to reference it later.
- `<expression>`: This is the value or expression that will be assigned to the output.

Here's an example of an `output` block:

```jsx
output "instance_ip" {
  value = aws_instance.example.public_ip
}
```

In this example, the `output` block defines an output value named `instance_ip`, which is assigned the value of the `public_ip` attribute of the `aws_instance.example` resource.

One can also use the `terraform output [<output-name>]` to get the output in CLI. Additionally, you can use the `raw` and `json` flag to display the output as raw text and in JSON format respectively.

### `locals` Block

The `locals` block in Terraform HCL allows you to define local variables that can be used to store intermediate or computed values within your configuration. **Local variables are only accessible within the same module or configuration where they are declared.**

Here is the syntax for declaring a `locals` block:

```hcl
locals {
  variable_name = "<expression>"
}
```

- `variable_name`: This is the name of the local variable, which can be used to reference it later.
- `<expression>`: This is the value or expression that will be assigned to the local variable.

Here's an example of a `locals` block:

```hcl
locals {
  instance_type = "t2.micro"
  instance_count = 2
}
```

In this example, the `locals` block declares two local variables named `instance_type` and `instance_count`.

To reference a local variable within your Terraform configuration, you can use the following syntax:

```hcl
local.<variable_name>
```

*Note here that `local` is used to reference the variable instead of `locals`.*

For example, if you want to use the `instance_type` local variable within a resource block, you can do the following:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = local.instance_type
}
```

In this example, the `instance_type` local variable is referenced using `local.instance_type`.

Local variables can be useful for simplifying and organizing your configuration by allowing you to reuse computed values throughout your code. They can also help improve readability and maintainability by providing meaningful names for intermediate values.

*It's important to note that local variables are evaluated during the Terraform graph construction phase and their values are determined before any resources are created or modified.*

### `resource` Block

In Terraform, a `resource` block is used to define infrastructure components. It specifies the type of resource (e.g., server, storage bucket) and its desired configuration. Terraform uses this configuration to create, update, or destroy resources in the infrastructure.

```hcl
resource "resource_type" "resource_name" {
  argument_name = value
}
```

- **`resource_type`:** Specifies the type of the resource, such as `aws_instance` for an EC2 instance.
- **`resource_name`:** An identifier for the resource within the module, which allows referencing it elsewhere.
- **Configuration arguments:** Define the properties of the resource, like size, tags, or behavior.

**Example:** The `aws_instance` resource block is used to provision an EC2 instance in AWS.

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "ExampleInstance"
  }
}
```

## Data Types

### Primitive Types

1. **`string`**: Represents a sequence of Unicode characters.
2. **`number`**: Represents numeric values, including integers and floating-point numbers.
3. **`bool`**: Represents a Boolean value (`true` or `false`).

### Complex Types

1. **`list(<TYPE>)`**: Represents an *ordered* collection of elements of the *same type.*
2. **`map(<TYPE>)`**: Represents a collection of key-value pairs where the keys are strings, and the values are of the specified type.
3. **`set(<TYPE>)`**: Represents an *unordered* collection of unique elements of the *same type.*
4. **`object({KEY = TYPE, ...})`**: The object type represents a collection of attributes, where each attribute has a name and a defined type. It is essentially a structured collection similar to a dictionary in other programming languages.
    
    ```hcl
    variable "example_object" {
      type = object({
        name = string
        age  = number
        active = bool
      })
    }
    ```
    
5. **`tuple([TYPE, ...])`**: The **`tuple`** type represents an *ordered* collection of elements where each element can have a *different type.* It is akin to a fixed-length array with heterogeneous data types.

### Dynamic Type

1. **`any`**: A special type that allows any value or type.
    
    ```hcl
    variable "example_any" {
      type = any
    }
    ```
    

## Resource Meta Arguments

Terraform offers a variety of meta arguments that can be utilized within any block to customize and enhance its functionality. Let's explore the different types of meta arguments available in Terraform:

1. **`depends_on`:** This meta argument allows you to **explicitly define dependencies** between resources, ensuring that they are created or modified in the correct order.
2. **`count`:** By using the `count` meta argument, you can easily create multiple instances of a resource based on a specified count. It can be set by `count = <integer>` and then accessing the index for each iteration with the `count.index` meta variable.
3. **`for_each`:** The `for_each` meta argument provides a convenient way to create multiple instances of a resource based on a `map` or `list` or `object` or `tuple` or `set` data type, basically covering all iterables. This is especially useful when you need to manage a dynamic set of resources. If a set of strings is used, then only `each.value` can be used. If a `map` object is used, then both `each.key` and `each.value` can be used.
4. **`provider`:** With the `provider` meta argument, you have the flexibility to select a non-default provider within a module. This can be setting the `alias` argument in the `provider` block and then specifying it in a resource as `<provider-name>.<alias-name>`. More on this can be found in [here](http://192.168.0.210:3000/apachex692/second-brain/src/branch/main/cloud/hashicorp-terraform-associate-003/providers-modules.md) under the Setting Aliases for Providers.
    
    ```hcl
    provider "aws" {
      region = "us-west-1"
    }
    
    provider "aws" {
      alias  = "secondary"
      region = "us-east-1"
    }
    
    resource "aws_s3_bucket" "primary" {
      provider = aws
      bucket   = "primary-bucket"
    }
    
    resource "aws_s3_bucket" "secondary" {
      provider = aws.secondary
      bucket   = "secondary-bucket"
    }
    ```
    
5. **`lifecycle`:** The `lifecycle` meta argument allows you to customize the lifecycle behavior of resources. You can define actions such as create before destroy or ignore changes, giving you fine-grained control over resource management.
    
    ```hcl
    resource "aws_s3_bucket" "example" {
      bucket = "my-unique-bucket"
      acl    = "private"
    
      lifecycle {
        create_before_destroy = true
        prevent_destroy       = true
        ignore_changes        = [acl]
      }
    }
    ```
    
    1. **`create_before_destroy`:** Ensures that Terraform creates a new resource before destroying the old one during an update. Useful for resources that need to be replaced without downtime.
    2. **`prevent_destroy`:** Prevents Terraform from destroying a resource, even if it is planned for deletion. This is helpful when you want to safeguard certain resources from accidental deletion.
    3. **`ignore_changes`:** Tells Terraform to ignore changes to specific resource attributes, preventing those attributes from being updated or deleted during the apply process. Can be used when you want to retain a resource’s state manually (e.g., when changes are applied outside Terraform).
    4. **`replace_triggered_by`:** Specifies a list of resource attributes or other resources that will trigger a replacement of the current resource if changed.

## Reference to Named Values

| **Type** | **Reference Syntax** |
| --- | --- |
| Resource | `<resource-type>.<name>.attribute` |
| Input Variables | `var.<name>` |
| Local Variables | `local.<name>` |
| Child Module Output | `module.<module-name>.<output-name>` |
| Data Source | `data.<data-name>.<attribute>` |
| Output | `output.<name>` |

## Filesystem and Workspace Info

- **`path.module`:** Path of the module where this expression is placed.
- **`path.root`:** Path of the root of the configuration.
- **`path.cwd`:** Path of the current working directory.
- **`terraform.workspace`:** Name of the currently selected workspace.

## Block-scoped Values

- **`count.index`:** Can be used when the `count` meta argument is used in a block.
- **`each.key` and `each.value`:** Can be used when the `for_each` meta argument is used in a block.
- **`self.<attribute>`:** Can be used to reference an attribute in a block. It is used mostly used in `provisioner` and `connection` block.
