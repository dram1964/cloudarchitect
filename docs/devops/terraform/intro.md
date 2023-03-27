# Terraform Overview

Terraform's [configuration language](https://developer.hashicorp.com/terraform/language) 
is used to declare resources representing infrastructure objects. Terraform has a core
set of language features to read the configuration and create a dependancy graph. Plugins
are used to add features to interact with upstream APIs. Plugins are either 'providers' or
'provisioners'.

Provider plugins implement resources through basic CRUD operations.
The Azure REST API exposes a client library that encapsulates the request-response 
objects of the API. The Terraform Azure provider uses this 
client library to connect to and invoke remote API endpoints for Azure. 

Terraform providers exist for a vast array of upstream APIs. The [Terraform Registry](https://registry.terraform.io/browse/providers) provides a list of existing providers.  

Terraform maintains a state file that is created the first time a Terraform configuration is 
executed in a working directory. The state file is a JSON representation of the remote
infrastructure and is used by Terraform to plan and execute the changes to be made to the 
infrastructure when Terraform commands are run. 

Use `terraform state list` to list the objects in the state file. To inspect 
a specific object use `terraform state show <ID>`.
The ID is declared in the resource block, and is the result of joining the resource
type and resource name with a '.'. 

```hcl
resource "azurerm_virtual_machine" "my_vm" {
..
}
```

In the above example, the resource_id is `azurerm_virtual_machine.my_vm`.

# Terraform Workflow

The general workflow for deploying infrastructure in Terraform is:

1. Initialise the working directory: `terraform init`
2. Validate the scripts: `terraform validate`
3. Run the plan: `terraform plan`
4. Deploy the infrastructure: `terraform apply`
5. Destroy the infrastructure: `terraform destroy`

Terraform 'init' is executed in the directory containing the Terraform scripts 
and will initialise the state file and download required providers and modules 
defined by the scripts. Modules and providers are downloaded and stored in the 
hidden '.terraform' folder in the working directory. Once you've initialised the
state you can use `terraform console` to inspect variables and expressions in the 
context of your configuration. 

Terraform 'validate' can be used to validate the syntax of the scripts in the 
working directory: there is also a 'fmt' command available to apply best 
practice formatting to the scripts

The 'terraform plan' command is used to determine what changes will be 
made when the scripts are applied. It compares the target infrastructure to the 
current state file and then compares the declared configuration to the state 
file. The difference between the two allows terraform to determine what changes
need to be made when the configuration is applied and to output these changes as
a plan.

After reviewing the plan, 'terraform apply' can be used to apply the 
desired configuration as defined in the scripts to the target infrastructure. 
If the apply is successful, Terraform generates a state file which, by default, 
is stored in the current working directory as 'terraform.tfstate'. Use 
`terraform show` to see the full state file. When using `azuread` provider
to create users, groups and assign the users to groups, you need to create 
the users before you try to assign group membership. You can run 
`terraform apply -target azuread_user.users` to create the users first, 
before running `terraform apply` to create the groups and group 
memberships.

The 'terraform destroy' command can be used to remove resources from 
environment and also from the state file. The '-target' option will allow 
you to specify a specific resource to remove:

```bash
terraform destroy -target azurerm_virtual_machine.user_vm[1]
```



