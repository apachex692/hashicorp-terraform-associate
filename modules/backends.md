# Back-ends

In Terraform, back-ends are essential for storing the state of your infrastructure, and they come in two primary types: local and remote back-ends.

## Local Back-ends

**Local back-ends** store the state file on the local disk of the machine where Terraform is running. This method is straightforward and easy to set up, making it suitable for small-scale projects, individual developers, or testing environments where collaboration and state locking are not necessary. The simplicity of local back-ends allows for quick initialization and low overhead, but they lack the features required for team collaboration and resilience. For example, a local back-end configuration might look like this:

```hcl
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

> Note: Not providing a back-end will default to using state file in root directory of the project.

## Remote Back-ends

In contrast, **remote back-ends** store the state file in a remote location, such as cloud storage services or dedicated Terraform management services. Remote back-ends provide several advantages, including the ability to support collaboration, state locking, versioning, and enhanced security features like encryption. They are ideal for larger projects and team environments where multiple users need to work on the same infrastructure, as they ensure that the state file is consistently accessible and protected against loss. An example of a remote back-end using Amazon S3 looks like this:

```hcl
terraform {
  backend "s3" {
    bucket = "mybucket"
    key    = "path/to/my/key"
    region = "us-west-2"
  }
}
```

Remote back-ends can be of two types:

1. **Standard:** This type of back-end can only store the state files but cannot execute Terraform commands. For example: S3.
2. **Extensive:** This type of back-end can store the state files and also execute Terraform commands. For example: HashiCorp Cloud.

Choosing between local and remote back-ends depends on the scale and requirements of your project. Local back-ends are perfect for simplicity and individual use, whereas remote back-ends are essential for collaboration, reliability, and advanced state management features.

**Special Note: The Terraform Cloud Remote Back-end**

For specifying Terraform Cloud alone, an additional block called `cloud` has been provided by Terraform.

**Legacy:**

```hcl
backend "remote" {
  hostname     = "app.terraform.io"
  organization = "PaperCloud"

  workspaces {
    prefix = "my-app-"
  }
}
```

> Note: We can provide the complete name of the workspace or provide a part of it like shown above and you'll be prompted to choose from workspaces with matching prefix when you run terraform apply.

**Recommended (only for Terraform Cloud):**

```hcl
terraform {
  cloud {
    hostname     = "app.terraform.io"
    organization = "PaperCloud"

    workspaces {
      tags = ["app"]
    }
  }
}
```

Here's a list of different remote back-end providers for Terraform:

1. **Amazon S3**
   - **Versioning**: Yes (with S3 versioning enabled)
   - **Locking**: Yes (with DynamoDB)
2. **Azure Storage**
   - **Versioning**: Yes
   - **Locking**: No
3. **Google Cloud Storage (GCS)**
   - **Versioning**: Yes (with GCS versioning enabled)
   - **Locking**: No
4. **HashiCorp Consul**
   - **Versioning**: No
   - **Locking**: Yes
5. **Terraform Cloud**
   - **Versioning**: Yes
   - **Locking**: Yes
6. **PostgreSQL**
   - **Versioning**: No
   - **Locking**: Yes
7. **`etcd` (key-value data store)**
   - **Versioning**: No
   - **Locking**: Yes
8. **Alibaba Cloud OSS**
   - **Versioning**: Yes
   - **Locking**: No

## Dynamic Back-ends

Sometime the back-end can be dynamic, meaning you'd have different back-ends for one configuration. In that case, we can store the config separately in a file or pass it as command-line arguments.

For example, consider the block:

```hcl
terraform {
  backend "remote" {
    # Empty
  }
}
```

In some file called `config.remote.tfbackend`, we can have configurations in key-value pairs like:

```hcl
organization = some

workspace {
  name = some
}
```

And while initializing the back-end, we can run:

```bash
terraform init -backend-config=./config.remote.tfbackend
```

Or we can pass them directly as:

```bash
# Different Back-end (Consul)
terraform init \
    -backend-config="address=demo.consul.io" \
    -backend-config="path=example_app/terraform_state" \
    -backend-config="scheme=https"
```

# State Locking

State locking in Terraform is a mechanism that prevents multiple users or processes from making concurrent changes to the state file. This ensures that the state file, which keeps track of the infrastructure resources, remains consistent and avoids conflicts that can arise from simultaneous updates.

**Importance of State Locking**

1. **Consistency:** Prevents simultaneous operations that could cause inconsistencies in the infrastructure state.
2. **Conflict Prevention:** Avoids race conditions where two processes might try to modify the same resources at the same time.
3. **Safety:** Ensures that when one user is making changes, others cannot interfere, which could potentially corrupt the state file.

When a Terraform operation that modifies the state is initiated (e.g., `terraform apply`, `terraform plan`, `terraform destroy`), Terraform attempts to acquire a lock on the state file. If it successfully acquires the lock, it will proceed with the operation. If it cannot acquire the lock because another process holds it, the operation will fail with an error indicating the state is locked.

**Backends Supporting State Locking**

Terraform supports state locking with certain backends. Some common backends that support state locking include:

- **Amazon S3 with DynamoDB table:** Uses DynamoDB for the locking mechanism.
- **Azure Storage:** Uses Azure Blob Storage for locking.
- **Google Cloud Storage (GCS):** Provides native locking.
- **HashiCorp Consul:** Uses Consul's distributed key-value store for locking.

**Example: State Locking with Amazon S3 and DynamoDB**

Let's consider an example where we use an S3 bucket to store the state file and a DynamoDB table for state locking.

**Step-1: Create an S3 Bucket and DynamoDB Table:**

1. **Create an S3 bucket** to store the Terraform state file.
2. **Create a DynamoDB table** to manage state locking. The table should have a primary key named `LockID` of type `String`.

**Step-2: Configure Backend in Terraform:**
In your Terraform configuration file (e.g., `main.tf`), configure the backend to use the S3 bucket and DynamoDB table:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "path/to/my/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "my-terraform-lock-table"
  }
}

provider "aws" {
  region = "us-west-2"
}

resource "aws_s3_bucket" "example" {
  bucket = "my-example-bucket"
}
```

In this configuration:

- The `bucket` parameter specifies the name of the S3 bucket.
- The `key` parameter specifies the path within the bucket where the state file is stored.
- The `region` parameter specifies the AWS region.
- The `dynamodb_table` parameter specifies the name of the DynamoDB table used for state locking.

**Step 3: Initialize and Apply Terraform:**

1. **Initialize Terraform:**
   ```
   terraform init
   ```
2. **Apply the configuration:**
   ```
   terraform apply
   ```

When you run `terraform apply`, Terraform will:

- Attempt to acquire a lock in the DynamoDB table.
- If the lock is acquired, proceed with the operation.
- If another process holds the lock, it will retry until the lock is available or timeout.

**Error Message When State is Locked**

If you try to run a Terraform operation while the state is locked by another process, you might see an error message like:

```bash
Error: <error-message>
Lock Info:
  ID:        12345-abcde-67890-fghij
  Path:      path/to/my/terraform.tfstate
  Operation: OperationType
  Who:       user@hostname
  Version:   0.14.0
  Created:   2024-06-13 12:34:56.789 +0000 UTC
  Info:
```

This message indicates that the state is currently locked by another process or user.

State locking in Terraform is a crucial feature that ensures the integrity and consistency of your infrastructure state by preventing concurrent modifications. By using supported backends like S3 with DynamoDB, Azure Storage, GCS, or Consul, you can effectively manage state locks and avoid potential conflicts and inconsistencies.

**Force Unlocking State Files**

Force unlocking the Terraform state is a process used to manually remove a lock on the state file when the lock persists due to a failed operation or if the lock holder cannot release it. This situation might arise due to unexpected termination of the process holding the lock or network issues. Force unlocking should be done cautiously as it can lead to potential inconsistencies if multiple operations are modifying the state concurrently.

**Steps to Force Unlock State:**

1. **Identify the Lock:** Determine if the state is indeed locked and get the lock ID if possible.
2. **Force Unlock:** Use the `terraform force-unlock` command followed by the lock ID to remove the lock.

## Example Scenario

Imagine you are working with a state file stored in an S3 bucket with a DynamoDB table for state locking, and you encounter a situation where the state is locked due to a failed process. Here's how you would proceed to force unlock the state.

**Step-1: Identify the Lock:**
When you run any Terraform command (e.g., `terraform apply`) and the state is locked, Terraform will provide an error message indicating the lock ID. The error message looks something like this:

```bash
Error: <error-message>
Lock Info:
  ID:        12345-abcde-67890-fghij
  Path:      path/to/my/terraform.tfstate
  Operation: OperationType
  Who:       user@hostname
  Version:   0.14.0
  Created:   2024-06-13 12:34:56.789 +0000 UTC
  Info:
```

Note the `ID` field (e.g., `12345-abcde-67890-fghij`) which is the lock ID.

**Step 2: Force Unlock:**
Use the `terraform force-unlock` command with the lock ID to release the lock:

```bash
terraform force-unlock 12345-abcde-67890-fghij
```

Terraform will prompt for confirmation before proceeding. If you confirm, it will forcibly remove the lock.

> Terraform will forcefully unlock the state for the given lock ID! This will not cancel the operation in progress. If someone else is running Terraform against this state, they will encounter errors. Only force unlock if you are sure this is safe.

> Do you really want to force unlock? Enter "yes" to continue: yes

**Force Unlock Considerations**

- **Use as a Last Resort:** Force unlock should only be used when you are certain that no other Terraform operations are running. Using it while another operation is active can lead to state corruption.
- **Verify State Integrity:** After force unlocking, it is a good practice to verify the state and ensure that it reflects the actual infrastructure. Running `terraform plan` can help detect any discrepancies.
- **Resolve Underlying Issues:** Identify and resolve the root cause of the lock not being released properly to avoid repeated issues.

## Summary

Force unlocking in Terraform is a necessary tool for situations where the state lock cannot be released by the usual means. By carefully identifying the lock and using the `terraform force-unlock` command, you can remove persistent locks. However, it should be used with caution to prevent potential inconsistencies and conflicts in your Terraform state.
