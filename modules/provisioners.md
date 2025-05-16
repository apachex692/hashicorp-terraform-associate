# Provisioners

Terraform Provisioners install software, edit files and provision machines created with Terraform. Terraform officially supports to provisioners:

1. Cloud Init (by Canonical)
2. Packer

_Provisioners should be used only as a last resort because whatever they do is not reflected in the state files._

Terraform has deprecated the use of provisioners for tools like Ansible, Chef, Puppet and SaltStack because they don’t follow best practices.

## local-exec and remote-exec Provisioner

The `local-exec` provisioner in Terraform is used to execute commands **on the machine where Terraform itself is run**, typically your local system or a CI/CD runner. This is useful for tasks such as logging outputs, running helper scripts, sending notifications, or triggering external tools. For example, after provisioning an AWS EC2 instance, you might want to log its public IP address to a file on your local machine. Here's how you can do that:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> instance_ips.txt"
  }
}
```

You can also specify a different shell or interpreter if your command requires it. For instance, running a script using Bash would look like this:

```hcl
provisioner "local-exec" {
  command     = "./setup.sh"
  interpreter = ["/bin/bash", "-c"]
}
```

On the other hand, the `remote-exec` provisioner is used to run commands directly **on the remote resource**, such as a VM provisioned in the cloud. This is often used for tasks like updating the OS, installing software, or bootstrapping the machine after it's up and running. It requires a `connection` block that defines how Terraform will access the remote machine—typically via SSH for Linux or WinRM for Windows. Below is an example that installs NGINX on a newly created EC2 instance:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install -y nginx"
    ]
  }
}
```

You can either use `inline` to provide commands directly or use the `script` argument to run a shell script. For example:

```hcl
provisioner "remote-exec" {
  script = "scripts/bootstrap.sh"
}
```

While both provisioners offer flexibility, they break Terraform’s declarative model and are considered a last resort. Provisioners can introduce non-determinism into your infrastructure—if the command fails, Terraform will mark the resource as "tainted" and may destroy and recreate it on the next apply. They also suffer from timing and connectivity issues, especially in cloud environments where VMs may not be ready to accept SSH/WinRM immediately after creation. For most configuration tasks, Terraform recommends alternatives like cloud-init, user-data scripts, or external configuration management tools such as Ansible or Chef.

In summary, use `local-exec` when you need to run commands locally during resource creation, such as for logging or external orchestration, and use `remote-exec` when you need to configure the remote resource directly after it's provisioned. Both are powerful tools, but they should be used sparingly and thoughtfully to maintain Terraform’s idempotence and reliability.

## File Provisioner

The `file` provisioner in Terraform is used to copy files or directories from the machine where Terraform is executed (typically your local system or CI/CD runner) to a remote resource, such as a cloud virtual machine. This provisioner is useful when you need to transfer scripts, configuration files, binaries, or any other data to a remote host as part of the provisioning process. It works in tandem with a `connection` block, just like `remote-exec`, to establish communication with the remote machine via SSH (for Linux) or WinRM (for Windows).

For instance, if you want to upload a shell script named `setup.sh` from your local system to the `/tmp` directory of a newly provisioned EC2 instance, the configuration would look like this:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }

  provisioner "file" {
    source      = "scripts/setup.sh"
    destination = "/tmp/setup.sh"
  }
}
```

In this example, Terraform uses the SSH connection to securely transfer the file to the target EC2 instance. The `source`can be a relative or absolute path on your local machine, and the `destination` is the absolute path where the file should be placed on the remote system. You can also copy entire directories by specifying a folder as the source; Terraform will recursively copy the contents to the destination.

Another example would be copying a full configuration directory:

```hcl
provisioner "file" {
  source      = "config/"
  destination = "/etc/myapp/config/"
}
```

Keep in mind that the remote path must be writable by the SSH/WinRM user, and the necessary permissions must be in place for the file operation to succeed. Also, while the `file` provisioner is convenient for ad hoc transfers, it has the same drawbacks as other provisioners: it introduces imperative steps that are not part of Terraform's core declarative model. If the copy operation fails, the resource may be marked as tainted, and it can lead to unpredictable behavior if re-applied.

For more reliable and repeatable provisioning, especially at scale, it's generally better to use mechanisms like baked-in AMIs, container images, or cloud-init scripts that automatically handle bootstrapping during instance launch. Still, the `file` provisioner remains useful in scenarios where lightweight, dynamic file transfer is necessary right after a resource is created.

## Null Resources

In Terraform, a null resource is a resource that doesn't create or manage any infrastructure. It is simply a placeholder that can be used to trigger provisioners or perform other tasks that don't require a physical or virtual resource.

The null_resource behaves like any other resource in Terraform, but it doesn't have any associated lifecycle management. It can be used to define dependencies, provisioners, and triggers.

**Use of `null_resource` Block with Provisioners**

The `null_resource` block can be used in combination with provisioners to execute scripts or commands that are not directly related to any specific resource. It allows you to perform additional configuration or actions based on the outcome of other resources.

For example, you can use a `null_resource` block with provisioners to execute a script after the creation of a set of resources or to perform a cleanup task before destroying resources. Here is an example of using `null_resource` with provisioners:

```hcl
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
