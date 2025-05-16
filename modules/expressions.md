# Expressions

Expressions in HashiCorp Configuration Language (HCL) are used to dynamically compute and define values within the configuration files. They allow you to perform mathematical calculations, manipulate strings, and make decisions based on conditions. Expressions in HCL are written using a combination of operators, functions, and variables, providing flexibility and automation in your configuration logic.

## Types and Values

Value is the result of an expression and every value has a type. Following are the types available in HCL.

1. Primitive Types
   1. `string` (always use **double quotes** as they can interpret escape sequences better)
   2. `bool`
   3. `number` (integer and float)
2. Complex Types
   1. `list` (tuple)
   2. `map` (object)
   3. `set`
3. No Type
   1. `null` (**note:** null represents the absence or omission when you want to the *underlying default of a provider’s resource configuration option.)*

## Strings and Templates

String interpolation allows you to evaluate an expression between the markers `${}` and convert it to a string:

```hcl
"echo ${ var.name } | grep \"lol\""
```

String directive allows you to evaluate conditions based on the logic marked between the `%{}` markers:

```hcl
"echo %{ if var.name != "" }${var.name}%{ else }anonymous%{ endif }"
```

These interpolations can be used with heredoc too. To strip whitespaces created by the directives, you can use the `~` at the end of a block like `~}`:

```hcl
"echo %{ if var.name != "" ~}${var.name}%{ else ~}anonymous%{ endif ~}"
```

## Escape Sequences

Following are the escape sequences followed:

1. `\n`
2. `\r`
3. `\t`
4. `\"`
5. `\\`
6. `\u` and `\U` for Unicode Character from Multilingual/Supplemental Planes

**Special Escape Sequences**

1. `$${` (prevents interpolation sequence)
2. `%%{` (prevents template directive sequence)

Terraform also supports the “heredoc” style following the UNIX standards. It can be defined as:

```hcl
<<EOF
echo "$HELLO"
bash
ls -a -l -c
EOF
```

## Operators and Loops

All operators supported in a general programming language are supported in HCL too. `for` loops can be used in HCL to apply transformation to data types as follows:

1. **Result as a Tuple**
   ```bash
   [for country in var.countries: upper(country)]
   ```
2. **Result as a Object**
   ```bash
   {for country in var.countries: country => upper(country)}
   ```
3. **Using Index-value Pairs in List**
   ```bash
   [for index, value in var.countries: "${index}: ${value}"]
   ```
4. **Looping a `map` Object**
   ```bash
   [for key, value in var.people: length(key) + length(value)]
   ```

Note that the output of these expressions will be ordered from A-Z.

## Splat Expressions

Splat expressions in Terraform are used to extract specific attributes from a list or a map of objects returned by a resource or data source. They enable you to reference individual elements within a list or map without needing to know the exact index or key.

For example, if you have a list of objects representing AWS instances, you can use a splat expression to extract specific attributes like instance IDs or private IP addresses:

```hcl
aws_instance.example[*].id
```

This would return a list of instance IDs from the `aws_instance.example` resource. Consider the following `resource` block:

```hcl
resource "aws_instance" "example" {
  count = 3
  ami   = "ami-123456"
  instance_type = "t2.micro"
}
```

The way you’d access, let’s say the attribute ID of these instances would be as follows:

```hcl
output "first_instance_id" {
  value = aws_instance.example[0].id
}

output "second_instance_id" {
  value = aws_instance.example[1].id
}
```

But with splat expressions:

```hcl
output "instance_ids" {
  value = aws_instance.example[*].id
}
```

This would return a list of values from the `example_map` variable. The same can be applied for iterators with `map` data type as source:

```hcl
resource "aws_instance" "example" {
  for_each = {
    us_east_1 = "ami-123456"
    us_west_1 = "ami-654321"
  }
  ami           = each.value
  instance_type = "t2.micro"
}

# Without
output "us_east_1_instance_id" {
  value = aws_instance.example["us_east_1"].id
}

output "us_west_1_instance_id" {
  value = aws_instance.example["us_west_1"].id
}

# With
output "instance_ids" {
  value = aws_instance.example[*].id
}
```

Splat expressions are particularly useful when you want to iterate over multiple resources or data sources and perform operations on their attributes dynamically. They provide a concise and flexible way to work with complex data structures in Terraform.

## Dynamic Blocks

Dynamic blocks in Terraform allow dynamic creation of repeatable nested block. The structure of the dynamic block is as follows:

```hcl
dynamic "<actual-block-name>" {
  for_each = <complex-object>
  iterator = "<iterator-name>"

  content "<block-name>" {
    <parent-block-attribute> = "<value>"
  }
}
```

- The `<actual-block-name>` is the actual name of the block that is nested inside the parent block.
- The `iterator` argument (optional) sets the name of a temporary variable that represents the current element of the complex value. If omitted, the name of the variable defaults to the label of the dynamic block (`"<actual-block-name>"` in the example above).
- The `content` sub-block actually is the block it’ll be translated to after looping. Once the iteration is done, we’ll have N blocks of the following format.
  ```hcl
  <block-name> {
   ...
  }
  ```
- Keys and values of this complex block can be accessed as: `<actual-block-name>|<iterator-name>.key|value.<attribute-name>`.

Let’s say you need to create a bunch of ingress rules for an EC2 Security Group. Then:

1. You will define a `locals` block with the following variables:
   ```hcl
   locals {
     ingress_rules = [
       {
         port        = 443
         description = "HTTPS"
       },
       {
         port        = 22
         description = "SSH"
       }
     ]
   }
   ```
2. Now, inside the `resource` block, you can create blocks dynamically as follows:

   ```hcl
   resource "aws_security_group" "website_sgp" {
     name   = "website-sgp"
     vpc_id = "vpc-xxxxxxxx"

     dynamic "ingress" {
       for_each = local.ingress_rules

       content {
         description = ingress.value.description
         from_port   = ingress.value.port
         to_port     = ingress.value.port
         protocol    = "tcp"
         cidr_blocks = ["0.0.0.0/0"]
       }
     }
   }
   ```
