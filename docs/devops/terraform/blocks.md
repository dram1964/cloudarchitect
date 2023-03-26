# Terraform Blocks

HCL scripts are used to declare the desired state using a series of code blocks. Each block has a type and serves a specific purpose:

## Terraform

The terraform block is used to configure the terraform version, which providers will be used and the backend for the configuration. Required providers must be specified in the terraform block. 

Provider sources can be listed as hostname/namespace/name. 
If the provider is in the hashicorp namespace then only the name is needed. If the provider is 
not listed in a registry, then terraform will attempt to find the provider in the local 
terraform plugins directory: `~/.terraform.d/plugins/${host_name}/${namespace}/${type}/${version}/${target}`

```hcl
terraform {
    required_providers {
        azurerm = {
            version = "=2.86"
            source = "hashicorp/azurerm"
        }
    }

    backend azurerm {}
}
```

## Provider

Used to configure provider plugins:

```hcl
provider "azurerm" {
  features {}
}
```

Sensitive values should be supplied as environment variables. The azurerm 
provider will automatically use the following environment variables, if set:

- ARM_CLIENT_ID
- ARM_CLIENT_SECRET
- ARM_SUBSCRIPTION_ID
- ARM_TENANT_ID

Alternatively, you can specify these values in the provider block:

```hcl
provider "azurerm" {
  client_id         = var.client_id
  client_secret     = var.client_secret
  subscription_id   = var.subscription_id
  tenant_id         = var.tenant_id

  features {}
}
```


## Variable

Used to accept external values as parameters to be used by all other blocks, apart from the terraform block. See [Variables](../variables)

## Resource

Used to represent a resource instance in the remote infrastructure that will be managed by terraform. The resource block defines the desired resource configuration. 

Resource blocks declare a resource type and a name. Resource types always begin with the provider
name followed by an underscore: `azurerm_` or `random_` or `aws_` or some other provider. 

Each provider, provides a unique list of resources: the random provider currently provides: 

- random_id
- random_integer
- random_password
- random_pet
- random_shuffle
- random_string
- random_uuid

## Output

Used to export data about your resources. The data can be used to configure other parts
of your infrastructure or as a data source for another Terraform workspace. 

Output values can be set directly from variables or as part of an expression:

```hcl
output "direction" {
  value = var.compass
}

output "region" {
  value = "UK ${var.compass}"
}
```

Outputs should be defined with a 'description' attribute, to explain the intent
and content of the output. Terraform stores output values in the state, and therefore
the state needs to be applied before the output values are accessible. Use 
`terraform output` to see the output values. To see a specific output, include its name:
`terraform output region`. String values are retrieved in double-quotes: use the `-raw` 
switch to retrieve the value without quotes: 
```bash
curl $(terraform output -raw web_url)
```

`terraform output` also supports a `-json` flag to retrieve data in JSON format. 

output blocks also support the 'sensitive' attribute, to prevent accidentally 
displaying sensitive data on the console. Sensitive outputs are stored in plain text 
in your state file. Terraform will also display sensitive outputs if queried directly by
name or in JSON outputs. 

## Data

See [Terraform Data Sources](https://developer.hashicorp.com/terraform/language/data-sources) 
documentation.

A data block can be used to read information from existing resources. For 
example, to retrieve secrets from a keyvault, the following data blocks can 
be used:
```
data "azurerm_key_vault" "projectkv" {
  name = var.keyvault_name
  resource_group_name = var.keyvault_rg
}

data "azurerm_key_vault_secret" "admin_user" {
  name = var.administrator_login
  key_vault_id = data.azurerm_key_vault.projectkv.id
}

data "azurerm_key_vault_secret" "admin_password" {
  name = var.administrator_password
  key_vault_id = data.azurerm_key_vault.projectkv.id
}

resource "azurerm_virtual_machine" "virtual_machine" {
  count = 3
  ...
  os_profile {
    computer_name = ${var.machine_name}-${count.index}
    admin_username = data.azurerm_key_vault_secret.admin_user.value
    admin_password = data.azurerm_key_vault_secret.admin_password.value
  }
  ...
}
```

`azurerm_key_vault` and `azurerm_key_vault_secret` are data sources provided by the 
azurerm provider. 

The `terraform_remote_state` data source can be used to access output data from another
workspace: 

```hcl
data "terraform_remote_state" "cvm" {
  backend = "local"

  config = {
        path = "../vm-image-gallery/terraform.tfstate"
  }
}
```

The above example uses a local backend to access state from the local filesystem. To 
access remote state from Terraform cloud, set the backend to 'remote': 

```hcl
data "terraform_remote_state" "lz" {
  backend = "remote"

  config = {
    organization = "my-terracloud-org"
    workspaces = {
      name = "lz"
    }
  }
}
```

## Module

Used to include external modules in your scripts. See [Modules](../modules)

## Local

locals allow you to set a name for the value of an expression. Locals differ from variables, 
in that they cannot be set directly from user input (for example using `-var` or `.tfvars` or `TF_VAR_` expressions). 

Locals can be used in much the same way as variables, but are prefixed with `local.` instead of 
`var.`. 

Locals can be useful to ensure default tag values are applied to resources:

```hcl
locals {
  required_tags = {
    project     = var.project_name,
    environment = var.environment
  }
  tags = merge(var.resource_tags, local.required_tags)
}
```

Now referring to `local.tags` will pick up the tags specified in `var.resource_tags` and
additionally ensure that the `project` and `environment` tags are also included.

