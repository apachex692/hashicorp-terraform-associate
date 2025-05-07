# Provisioners

Terraform Provisioners install software, edit files and provision machines created with Terraform. Terraform officially supports to provisioners:

1. Cloud Init (by Canonical)
2. Packer

*Provisioners should be used only as a last resort because whatever they do is not reflected in the state files.*

Terraform has deprecated the use of provisioners for tools like Ansible, Chef, Puppet and SaltStack because they don’t follow best practices. Provisioners are defined inside a resource block as follows:

```jsx
resource "<resource-name>" "<alias>" {
  provisioner "<provisioner-type>" {
  }
}
```

```jsx
provisioner "<provisioner-type>" {
}
```

## `local-exec` Provisioner

The `local-exec` provisioner is used to execute scripts in the local machine after the configuration has been provisioned. Consider the scenario where the public IP of an VM provisioned by Terraform is needed by Ansible to add that instance to management. `local-exec` can help in such cases. A local environment can be:

1. Local Machine
2. Build Server (on which Terraform runs)
3. Terraform Cloud Run Environment

Following are the three modes for passing commands to the `local-exec` resource:

1. Inline (pass actual commands as a list)
2. Script (pass the location of a script)
3. Scripts(pass the location of scripts as a list)

Following are the attributes that can be set inside the `local-exec` provisioner block:

1. `command`
2. `working_dir`
3. `interpreter` (BASH, PowerShell etc.)
4. `environment` (dictionary, used to pass environment variables)

You can use only one mode at a time.

## `remote-exec` Provisioner

It is used to execute scripts in the remote machine after it is provisioned. It is useful for execution of simple commands. For more complex commands, management tools like Cloud-Init, Puppet or Ansible must be used. The modes and commands are similar to `local-exec`.

## `file` Provisioner

It is used to copy files from the local machine to the newly created remote machine. The three attributes that can be set for this provisioner are:

1. `source` (file on local machine)
2. `content` (allows writing text directly instead of passing files)
3. `destination` (file/directory on remote machine)

## Connection

A connection block tells a provisioner or resource on how to establish a connection with the remote machine. Connection can be SSH-based (for GNU/Linux-based machines) or WinRM (for Windows-based machines).

## Null Resources in Terraform

In Terraform, a null resource is a resource that doesn't create or manage any infrastructure. It is simply a placeholder that can be used to trigger provisioners or perform other tasks that don't require a physical or virtual resource.

The null_resource behaves like any other resource in Terraform, but it doesn't have any associated lifecycle management. It can be used to define dependencies, provisioners, and triggers.

## Use of `null_resource` Block with Provisioners

The `null_resource` block can be used in combination with provisioners to execute scripts or commands that are not directly related to any specific resource. It allows you to perform additional configuration or actions based on the outcome of other resources.

For example, you can use a `null_resource` block with provisioners to execute a script after the creation of a set of resources or to perform a cleanup task before destroying resources.

Here is an example of using `null_resource` with provisioners:

```jsx
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo $HELLO"
  }

  provisioner "remote-exec" {
    inline = [
      "echo $XDG"
    ]
  }

  provisioner "file" {
    source      = "local/path/to/file.txt"
    destination = "/remote/path/to/file.txt"
  }

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = aws_instance.example.public_ip
  }
}
```

In this example, the `null_resource` block is used to define multiple provisioners: `local-exec`, `remote-exec`, and `file`. These provisioners will execute commands on the local machine, remote machine, and copy a file to the remote machine, respectively.

The `connection` block is used to establish an SSH connection with the remote machine, which is an `aws_instance` resource named `example`.

By using the `null_resource` block with provisioners, you can extend the functionality of Terraform and perform various actions and configurations beyond resource management.
