# Miscellaneous Notes

## Publishing Modules

Modules before publishing to Terraform Registry must be pushed to GitHub. The name of the repository must follow the convention:`terraform-<provider>-<module-name>`. Once this is done, you can connect your Terraform Registry account with GitHub and publish the module.

## Integration with Vault

Terraform can be integrated with Vault to retrieve secrets and store it to `variable` blocks. However, there comes a downside to it because Terraform stores it to the state file for later interaction with the infrastructure. Hence, it is required to make sure you store this state file somewhere safe.

But if you use a local config file, for example to store your AWS credentials (~/.aws/credentials), the state file won't store these credentials and AWS provider will read the credentials file everytime you provide it.
