# Terraform Architecture

Terraform is a tool to consume upstream APIs. The Azure REST API exposes a client library that 
encapsulates the request-response objects of the API. The Terraform Azure provider uses this 
client library to connect and invoke remote API endpoints for Azure. Each provider is a plugin
that be added to the Terraform core, allowing Terraform to interact with multiple upstream APIs
Terraform maintains a state file that is created the first time a Terraform configuration is 
executed in a working directory. The state file is a JSON representation of the remote
infrastructure and is used by Terraform to plan and execute the changes to be made to the 
infrastructure when Terraform commands are run.

# Terraform Blocks

HCL scripts are used to declare the desired state using a series of code blocks. Each block has a type and serves a specific purpose:

## terraform

The terraform block is used to configure the terraform version, which providers will be used and the backend for the configuration

    terraform {
    required_providers {
        azurerm = {
        version = "=2.86"
        source = "hashicorp/azurerm"
        }
    }

    backend azurerm {}
    }


## provider
Used to configure provider plugins:
```
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
```
provider "azurerm" {
  client_id         = var.client_id
  client_secret     = var.client_secret
  subscription_id   = var.subscription_id
  tenant_id         = var.tenant_id

  features {}
}
```


## variable
Used to accept external values as parameters to be used by all other blocks, apart from the terraform block

## resource
Used to represent a resource instance in the remote infrastructure that will be managed by terraform. The resource block defines the desired resource configuration

## output
Used to get information about the infrastructure after deployment. The 'terraform output' 
command can also be used to retrieve the value of an output variable. The variable value can 
be retrieved directly or as part of an expression:
```
variable "compass" {
  type = string
  default = "West"
}

output "direction" {
  value = var.compass
}

output "region" {
  value = "UK ${var.compass}"
}
```

## data
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


## module
Used to include external moudles in your scripts

## locals
blank for now

# Terraform Workflow

The general workflow for deploying infrastructure in Terraform is:

- Initialise the working directory (terraform init)
- Validate the scripts (terraform validate)
- Run the plan (terraform plan)
- Deploy the infrastructure (terraform apply)
- Destroy the infrastructure (terraform destroy

Terraform 'init' is executed in the directory containing the Terraform scripts 
and will initialise the state file and download required providers and modules 
defined by the scripts. Modules and providers are downloaded and stored in the 
hidden '.terraform' folder in the working directory.
Terraform 'validate' can be used to validate the syntax of the scripts in the 
working directory: there is also a 'fmt' command available to apply best 
practice formatting to the scripts
The 'terraform plan' command is used to determine what changes will be 
made when the scripts are applied. It compares the target infrastructure to the 
current state file and then compares the declared configuration to the state 
file. The difference between the two allows terraform to determine what changes
need to be made when the configuration is applied and to output these changes as
a plan
After reviewing the plan, 'terraform apply' can be used to apply the 
desired configuration as defined in the scripts to the target infrastructure. 
If the apply is successful, Terraform generates a state file which, by default, 
is strored in the current working directory as 'terraform.tfstate'.
The 'terraform destroy' command can be used to remove resources from 
environment and also from the state file. The '-target' option will allow 
you to specify a specific resource to remove:
```
terraform destroy -target azurerm_virtual_machine.user_vm[1]
```

# Terraform Variables

Terraform variable blocks allow an existing configuration to be tailored 
using external input values. A variable block starts with the keyword 'variable' 
followed by the variable name and a block of attributes associated to the variable.
These attributes include:

- type ("string", "bool", "number", "object", "map", "list", "tuple", "sets", "any")
- default: a default value to use if none is supplied
- description: a text string explaining the use of the variable
- validation: a block with code used to validate the value supplied for the variable
- sensitive: a boolean value to determine if the variable value is displayed by the Terraform CLI

Variables and their default values can be declared within the same scripts that you use to define
your infrastructure code. However, it is often preferrable to create a separate 'variables.tf' for 
this purpose
Variables values can be assigned on the command-line:

`terraform plan -var azure_region="UK South"`

This becomes impractical if you want to override several default values. Instead you can 
additionally create a 'variable definitions' file. The variable definitions file should consist of only variable assignments:
```
azure_region = "West Europe"
list_of_fruit = ["mango", "pineapple", "blueberry", "grape"]
res_tags = {
    location   = "UK South"
    department = "Accounts"
    owner      = "Freda Perry"
}
```

Terraform will automatically load variable defintion files named either 'terraform.tfvars' or 
files ending in '.auto.tfvars'. If you prefer to use JSON format for your variable definition files, 
Terraform will also load any files named 'terraform.tfvars.json' or ending in 'auto.tfvars.json'. 

```
{
  "azure_region" : "West Europe",
  "list_of_fruit" : ["mango", "pineapple", "blueberry", "grape"],
  "res_tags" : {
    "location"   : "UK South",
    "department" : "Accounts",
    "owner"      : "Freda Perry"
  }
}
```

Variables can also assigned in environment variables by prefixing the variable name with "TF_VAR_":
```
export TF_VAR_LOCATION="UK South"
```

The corresponding variable "LOCATION" will need to be declared in your scripts to be accessible:
```
variable "LOCATION" {
  default = ""
}

output "region" {
  value = var.LOCATION
}
```


## String Variables
Values for string variables are assigned with double-quoted strings:
```
variable "azure_region" {
  type        = string
  default     = "UK West"
  description = "The Azure region to deploy resources"
}

output "az_region" {
  value = var.azure_region
}
```
Terraform supports string interpolation to allow the generation of strings by
combining variable values and string values. String interpolation expressions 
are declared within a double-quoted string and references to variable values 
or other resources are enclosed within '${}':
```
name = "vm-${var.user_name}"
```
Terraform also supports 'String Directives' which allow the generation of 
strings using loops or conditional statements. String directives are declared
within double-quoted strings and the control statements are enclosed within 
'%{}':
```
output "users_created" {
  value = "%{ for val in var.user_names }${val} %{ endfor }"
}
```
Note that the string directive will output everything between the '%{}' blocks:
so that the space after '${val}' is also part of the output in the above example
String directives also support 'if' statements and HEREDOCS:
```
enable_public_ip = <<EOT
%{ if var.location == "UK South" }
true
%{ else }
false
%{ endif }
EOT
```

## List Variables
Lists are assigned as comma-separated lists inside square brackets:
```
variable "list_of_fruit" {
  type    = list(string)
  default = ["apple", "pear", "orange", "peach"]
}

output "first_fruit" {
  value = var.list_of_fruit[0]
}

output "all_fruits" {
  value = var.list_of_fruit
}
```

## Map Variables
Map variables are collection types, and contain key-value pairs. The values should adhere to the 
declared datatype. Each key should be unique, where the keys and values are of type string:
```
variable "res_tags" {
  type       = map(string)
  default = {
    location   = "UK West"
    department = "Finance"
    owner      = "Fred Perry"
  }
}

output "this_department" {
  value = var.res_tags["department"]
}
```

## Object Variables
Object variables are collection types, with key-value pairs where the values for each key can 
be of different types:
```
variable "user_account" {
  type = object({
    logon    = string
    email    = string
    roles    = list(string)
    projects = map(string)
    enabled  = bool
  })
  default = {
    logon = "fred001"
    email = "fred@example.com"
    roles = ["user", "report writer", "remote access", "acr pull"]
    projects = {
      primary = "solaris"
      second  = "aix"
      third   = "rhel"
    }
    enabled = true
  }
}

output "user_details" {
  value = var.user_account.logon
}

output "main_project" {
  value = var.user_account.projects.primary
}
```

## Set Variables
Sets are collections that can contain only unique values of the same datatype:
```
# Although the value "two" is included twice in the default assignment
# the value "two" will only occur once in the actual set variable
variable "myset" {
  type    = set(string)
  default = ["one", "two", "three", "four", "two"]
}

output "set_members" {
  value = var.myset
}
```
Elements of a set are identified only by their value and don't have any index or key to select
with, so it is only possible to perform operations across all elements of the set

## Tuple Variables
Tuples are similar to lists but each value in the list can of a different type:
```
variable "user_info" {
  type    = tuple([string, bool, number])
  default = ["Finance", true, 1]
}

output "tuple_values" {
  value = var.user_info
}

output "user_enabled" {
  value = var.user_info[1]
}
```

# Remote State 
Terraform documentation on [Terraform Backend](https://www.terraform.io/language/settings/backends/azurerm)

Microsoft documentation on [Remote State Storage](https://docs.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=azure-cli) for terraform

Here's another good article by [Matthew Davis](https://matthewdavis111.com/terraform/terraform-azure-remote-state/) which contains the info to use an SAS token for your backend

Storing Terraform state remotely allows multiple developers to work on the same
environment from different machines. Terraform state can be stored in Azure and then
referenced in the backend configuration of your Terraform scripts. Public access is allowed to Azure storage to store Terraform state.
First create the remote state storage account in your subscription using the 
Azure CLI:
```
az login
az account set --subscription <subscription_id>

export RESOURCE_GROUP_NAME=<resource-group-name>
export STORAGE_ACCOUNT_NAME=<storage-account-name>
export CONTAINER_NAME=<container-name>

az group create --name $RESOURCE_GROUP_NAME --location uksouth

az storage account create --resource-group $RESOURCE_GROUP_NAME --name $STORAGE_ACCOUNT_NAME --sku Standard_LRS --encryption-services blob

az storage container create --name $CONTAINER_NAME --account-name $STORAGE_ACCOUNT_NAME
```
To use remote state in Azure Storage, configure the backend block of your terraform block with:
- resource_group_name
- storage_account_name
- container_name
- key - the name of the state store file to be created
```
terraform {
  required_version = "~>v1.1.5"
  required_providers {
    azurerm = {
      version = "~>2.36"
      source  = "hashicorp/azurerm"
    }
  }
  backend "azurerm" {
    resource_group_name  = "<resource-group-name>"
    storage_account_name = "<storage-account-name>"
    container_name       = "<container-name>"
    key                  = "project1.terraform.tfstate"
  }
}
```
When you run `terraform init`, terraform will need to authenticate to Azure to create
the backend. If you have already logged-in with Azure CLI, terraform will use this authenticated 
connection to create the backend for you. If your deploying your infrastructure using Terraform 
Cloud or Azure DevOps you can create an SAS token to authenticate to the backend Storage Account.
To obtain SAS certificate for a Storage Account, you can use the following Azure CLI commands:
```
ACCOUNT_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP_NAME --account-name $STORAGE_ACCOUNT_NAME --query '[0].value' -o tsv)
END_DATE=`date -u -d "1 year" '+%Y-%m-%dT%H:%MZ'`
SAS_KEY=$(az storage container generate-sas -n $CONTAINER_NAME --account-key $ACCOUNT_KEY --account-name $STORAGE_ACCOUNT_NAME --https-only --permissions dlrw --expiry $END_DATE -o tsv)
ARM_SAS_TOKEN=$SAS_KEY
```
Terraform will automatically use the SAS key if you assign this to the environment variable 
$ARM_SAS_TOKEN, or you can add the value to your backend config using 'sas_token':

```
  backend "azurerm" {
    resource_group_name  = "<resource-group-name>"
    storage_account_name = "<storage-account-name>"
    container_name       = "<container-name>"
    key                  = "project1.terraform.tfstate"
    sas_token		 = var.sas_key_value
  }
```


Alternatively you could use an access key for the Storage Account to authenticate the backend. This can also be done using an environment variable:
```
ACCOUNT_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP_NAME --account-name $STORAGE_ACCOUNT_NAME --query '[0].value' -o tsv)
export ARM_ACCESS_KEY=$ACCOUNT_KEY
```
The SAS token is the better option since this gives access to the Storage Blob for the backend 
which is what is needed in this case: the Storage Account Key gives access to other resources 
in the storage account. Either way, the access key should be stored and retrieved from a key vault
for added security:
```
export ARM_SAS_TOKEN=$(az keyvault secret show --name terraform-backend-key --vault-name myKeyVault --query value -o tsv)
```
Rather than placing the values for the backend-config keys in your 
Terraform scripts you can store these values in a config file and use the '-backend-config' option in during terraform initialisation to use those values:
```
#### BEGIN /app/backend_config.conf #######
resource_group_name = ""
storage_account_name = ""
container_name = ""
key = ""
sas_token = ""
#### END /app/backend_config.conf   #######
```

```
terraform init --backend-config=/app/backend_config.conf
```
The value for sas_token can be omitted from the backend-config file, and 
instead supplied using the ARM_SAS_TOKEN environment variable.

# Terraform Meta-Arguments
## Count
The 'count' attribute can be added to any resource and the value of this 
attribute determines the number of resource instances to be deployed. The 
'count' attribute provides and 'index' property which can be use with 
variable interpolation to assign unique names to each resource created. The 
index begins at 0.
```
resource "azurerm_virtual_machine" "user_vm" {
  count = 3
  name = "${var.vm_name}-0${count.index}"
...
...
}

output vm_ids {
  value = azurerm_virtual_machine.user_vm[*].id
}
```
Terraform stores the list of resources created as an array in the terraform
state and the splatting operator '*' can be used to refer to this array as a 
whole.
The 'count' meta-element can also be used with a list variable, allowing more 
tailored names to be given to the resources created:
```
variable "vm_names" {
  type = list(string)
  default = ["vm_harry", "vm_larry", "vm_barry"]
}

resource "azurerm_virtual_machine" "user_vm" {
  count = length(var.vm_names)
  name = "${var.vm_names[count.index]}"
...
...
}

output vm_ids {
  value = azurerm_virtual_machine.user_vm[*].id
}
```

## For Each
The 'for_each' meta element can be used to iterate a map or set 
to provision multiple resources from a single 
resource block. 'for_each' provides a 'each.key' and an 'each.value' attribute
to further tailor the properties of the resources created:
```
variable "user_machines" = {
  type = map(string)
  default = {
    "vm_harry" : "rg-env-dev",
    "vm_larry" : "rg-env-live",
    "vm_barry" : "rg-env-test"
  }
}

resource azurerm_virtual_machine "user_vm" {
  for_each = var.user_machines
  name = each.key
  resource_group_name = each.value
...
...
}
```
The 'for_each' meta-element can also be used to iterate at the property level: instead of using for_each to define multiple instance of the same resource type, for_each can be used to define multiple instances of a property on a singe resource:
```
resource "azurerm_app_service" "app_service" {
  name = var.app_service_name
  location = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name
  app_service_plan_id = azurerm_app_service_plan.app_service_plan.id

  dynamic "connection_string" {
    for_each = var.connection_strings
      content {
        name = connection_string.name
        type = connection_string.type
        value = connection_string.value
      }
    }

   https_only = true
}
```
Property iterations use the keyword 'dynamic' along with the name of the 
property being iterated

## Depends On
Terraform can identify resource dependancies automatically, and will create
resources with no dependancies in parallel, whilst resources with dependancies
are created sequentially. But where a resource depends on another resources 
behaviour rather than it's data, we have to explicitly declare the dependancy. A 
data dependancy is clear to see: where the value for an attribute is declared by
reference to the value of an attribute of another resource. For example:
```
resource azurerm_virtual_machine "my_database_server" {
...
  resource_group_name = azurerm_resource_group.resource_group.name
...
}
```
Implicit dependancies must be declared as a list of references to other 
resources:
```
resource_azurerm_virtual_machine "my_app_server" {
...
  depends_on = [
      azurerm_virtual_machine.db_server.id,
      azurerm_virtual_machine.app_server.id
  ]
}
```

## Provider
Where multiple providers are being used in Terraform scripts, you can add
a provider attribute to a resource to tell it to use the non-default provider

## Lifecycle
The 'lifecycle' meta-argument can be used to change terraform's default 
behaviour when creating, updating or destroying resources

# Terraform Functions
[Terraform Functions]("https://www.terraform.io/language/functions")

You can use the Terraform console to test out the various built-in functions 
of the Terraform language
## Ternary Operator
Terraform provides the traditional ternary operator '?:' to implement
conditional logic within blocks:
```
variable "user_machines" = {
  type = map(string)
  default = {
    "vm_harry" : "rg-env-dev",
    "vm_larry" : "rg-env-live",
    "vm_barry" : "rg-env-test"
  }
}

resource azurerm_virtual_machine "user_vm" {
  for_each = var.user_machines
  name = each.key
  resource_group_name = each.value
  vm_size = each.key == "vm_larry" ? "Standard_DS1_v2" : "Standard_FS1_v2"
...
...
}
```

## Merge
The merge operator can be used to join two or more maps together:
```
locals = {
  default_tags = {
    "cost center" : "Central 234",
    "created_by" : "terraform"
  }
}

resource "azurerm_virtual_machine" "virtual_machine" {
...
  tags = merge(local.default_tags, var.custom_tags)
...
}
```
If the same key is defined in more than one map, the value from the right-most 
map is retained

## Lookup
The lookup operator can be used to retrieve a single value from a map using its
key:
```
lookup(map, key, default)
```

## For
'For' expressions can be used to iterate over any collection type to output 
another collection:
```
{ for name, location in var.vm_map: lower(name) => lower(location) }
[ for name, location in var.vm_map: "${name}-${location}" ]
```

# Provisioners
Terraform allows 'provisioner' blocks to be added to a resources attributes 
to enable execution of custom scripts or to copy files to the resource. There
are three built-in provisioner types:

- local-exec - used to execute scripts from the local machine
- file - used to copy files to the deployed resource
- remote-exec - used to execute scripts on the remote resource

The 'local-exec' scripts has one mandatory attribute (command) and three
optional attributes (working_dir, interpreter, environment, and when). 
Terraform also provides a 'null_resource' that can be used to execute a 
local script without associating the script execution to a specific resource:
```
resource "azurerm_resource_group" "resource_group" {
  name = var.resource_group_name
  location = var.location

  provisioner "local-exec" {
    command = "echo ${azurerm_resource_group.resource_group.id}"
  }
}

resource "null_resource" "my_web_app" {
  provisioner "local-exec" {
    command = "catalyst.pl"
    working_dir = "./bin"
    interpreter = ["perl", "-wc"]
    environment = {
      DEBUG = 1
      PORT = 8080
    }
  }
}
```
The local-exec provisioner also provides a 'when' attribute. If the value is
set to 'destroy' the command is only executed when 'terraform destroy' is 
called.
The 'file' provisioner is used to copy folders and files to the target 
resource:
```
provisioner "file" {
  source = "./resources/*.xml"
  destination = "/app/resources/"
}
```
The 'remote-exec' provisioner is used to execute scripts and commands on 
remote resources
```
provisioner "remote-exec" {
  inline = [
    "apt-get update",
    "apt-get install -y postgres"
  ]
}
```

# Terraform Modules
Modules in Terraform are reusable libraries of code that can be combined 
with a root configuration to define the required infrastructure. A module 
is a folder of Terraform files with the definition and configuration of 
resources but without a root configuration. Because modules lack a root 
configuration they can be executed directly and have no associated state file.
Modules need to be combined with a root configuration to deploy 
infrastructure. Re-useable modules helps a development team to maintain 
coding standards and policies with the team.
A module folder will typically contain the definition of a group of 
resouces that are logically related and have the same lifecycle. 
Provisioning the module would result 
in provisioning all the resources defined in the module.
Build your modules in a dedicated folder e.g. 'modules > resources'. Inside the
modules\resources folder you would create a separate folder for each module. Each
module folder should contain the following files: 

- dependencies.tf - defines the dependancies of the module (optional)
- main.tf - defines the resources supplied by the module
- outputs.tf - defines the outputs of the module
- variables.tf - defines the variables used by the module
- versions.tf - defines the versions for providers


## Example Resource Group Module
Suppose we want to create a module to ensure that all the resource_groups 
developed by a team are provisioned in the 'UK South' Azure region. We would 
create a 'groups' folder for our module and add the following to our main.tf 
script:
```
resource "azurerm_resource_group" "resource_group" {
  name     = var.resource_group_name
  location = 'UK South'

  tags = var.resource_group_tags
}
```
This script has the location for the resource group hard-coded, but also 
has two variables for the name of the resource group and the tags to be applied.
The variables should be defined in the variables.tf for the module:
```
variable "resource_group_name" { type = string }
variable "resource_group_tags" { type = map(string) }
```
The module should also provide some outputs that can be used both for testing
the module and whilst using the module:
```
output "complete_resource_group" {
  value = azurerm_resource_group.resource_group
}

output "resource_group_identifier" {
  value = azurerm_resource_group.resource_group.id
}

output "resource_group_name" {
  value = azurerm_resource_group.resource_group.name
}

output "resource_group_tags" {
  value = azurerm_resource_group.resource_group.tags
}

output "resource_group_location" {
  value = azurerm_resource_group.resource_group.location
}
```
We can then define the version for the azurerm provider in our 
versions.tf:
```
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=2.51.0"
    }
  }
}
```

## Testing the Module
The simplest test that can be performed is:
```
terraform fmt -check
```
This will check that the basic syntax for your module files are correct. More
sophisticated testing will require using the module in a test configuration. 
Each module that you build in the modules\resources folder should have a 
corresponding folder in the modules\fixtures folder. In the fixtures folder you 
will define a root configuration that uses the module. Instead of using the 
azurerm_resource_group module we'll use the groups module. Our main.tf file in 
modules\fixtures\groups will look like this:
```
terraform {
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = "=2.51.0"
    }
  }
}

provider "azurerm" {
  features {}
}

module "resource_group" {
  source = "../../resources/groups"
  resource_group_name = var.resource_group_name
  resource_group_tags = var.resource_group_tags
}
```
Note that we have not specified a location for the resource group, but we
have specified two variables for the resource group. We'll need to therefore 
define these variables somewhere. This could be done in main.tf, but can also 
be done in a variables.tf file in the modules\fixtures\groups folder:
```
variable resource_group_name { type = string }
variable resource_group_tags { type = map(string) }
```
We can also specify the desired values for these variables in a 
'testing.auto.tfvars' file:
```
resource_group_name = "rg-uks-testing-modules"
resource_group_tags = {
  "project_name": "deep dive terraform",
  "created_by": "David Ramlakhan",
}
```
Because we are using the modules\fixtures\groups folder for testing our 
module, we'll also want to access all of the outputs defined by the module. So,
we need also to add an outputs.tf script:
```
output "complete_resource_group" {
  value = module.resource_group.complete_resource_group
}

output "resource_group_identifier" {
  value = module.resource_group.resource_group_identifier
}

output "resource_group_name" {
  value = module.resource_group.resource_group_name
}

output "resource_group_tags" {
  value = module.resource_group.resource_group_tags
}

output "resource_group_location" {
  value = module.resource_group.resource_group_location
}
```
Note that the ouput values all refer to the module outputs. With the main.tf, 
variables.tf, outputs.tf and the testing.auto.tfvars file in place we can test
our module using the usual terraform workflow:
```
terraform init
terraform validate
terraform plan
terraform apply 
terraform destroy
```
If we later alter the scripts in the module, then we'd have to re-run terraform
init to pick up the changes.

## Module Versioning
Based on this blog at [Gruntwork](https://blog.gruntwork.io/how-to-create-reusable-infrastructure-with-terraform-modules-25526d65f73d)
When using the module in a configuration, the source for our modules can be 
set to reference a github repository:
```
module "resource_group" {
  source = "github.com/<owner>/<repo_name>.git//modules/resources/groups"
  resource_group_name = var.resource_group_name
  resource_group_tags = var.resource_group_tags
}
```
Note that the double forward-slash is required to specify the path to the 
module within the repository.
If your modules are stored in a private repo, then it is easier to use ssh 
instead of https to access the repo:
```
module "resource_group" {
  source = "git::git@github.com/<owner>/<repo_name>.git//modules/resources/groups"
  resource_group_name = var.resource_group_name
  resource_group_tags = var.resource_group_tags
}
```
You can tag the commits to simplify the process of using different versions 
of the module in your different environments:
```
git tag -a "v0.0.1" -m "First release of the groups module"
git push --follow-tags
# make some changes to the module...
git add .
git commit -m "Made some experimental changes to groups module"
git push origin master
# now add a tag for the new version
git tag -a "v0.0.2" -m "Second release of the groups module"
git push --follow-tags
```
Now in your different environments you can reference the module version by using the tag. For v0.0.1, use:
```
source = "git::git@github.com:/<owner>/<repo_name>//modules/resources/groups?ref=v0.0.1"
```
For v0.0.2, use:
```
source = "git::git@github.com:/<owner>/<repo_name>//modules/resources/groups?ref=v0.0.2"
```
