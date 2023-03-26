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
