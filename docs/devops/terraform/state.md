# Remote State 
Terraform documentation on [Terraform Backend](https://www.terraform.io/language/settings/backends/azurerm)

Microsoft documentation on [Remote State Storage](https://docs.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=azure-cli) for terraform

Here's another good article by [Matthew Davis](https://matthewdavis111.com/terraform/terraform-azure-remote-state/) which contains the info to use an SAS token for your backend


## Create Azure Storage

Storing Terraform state remotely allows multiple developers to work on the same
environment from different machines. Terraform state can be stored in Azure and then
referenced in the backend configuration of your Terraform scripts. Public access is allowed to Azure storage to store Terraform state.
First create the remote state storage account in your subscription using the 
Azure CLI:

```bash
    az login
    az account set --subscription <subscription_id>

    export RESOURCE_GROUP_NAME=<resource-group-name>
    export STORAGE_ACCOUNT_NAME=<storage-account-name>
    export CONTAINER_NAME=<container-name>

    az group create --name $RESOURCE_GROUP_NAME --location uksouth

    az storage account create --resource-group $RESOURCE_GROUP_NAME --name $STORAGE_ACCOUNT_NAME --sku Standard_LRS --encryption-services blob

    az storage container create --name $CONTAINER_NAME --account-name $STORAGE_ACCOUNT_NAME
```

## Configure Backend

To use remote state in Azure Storage, configure the backend block of your terraform block:
```hcl
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
connection to create the backend for you. If you're deploying your infrastructure using Terraform 
Cloud or Azure DevOps you can create an SAS token to authenticate to the backend Storage Account.
To obtain SAS certificate for a Storage Account, you can use the following Azure CLI commands:
```bash
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

## Using Backend Config

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

## Inspecting State

Whether you use a remote or local state storage, you can inspect the configuration of 
individual items using `terraform state show`. For instance if you have defined 
a resource group in your config files as `azurerm_resource_group.rg`, you can 
query the configuration using:

```bash
terraform state show azurerm_resource_group.rg
```

## Importing Resources

You can import resources into your state file using 
[terraform import](https://developer.hashicorp.com/terraform/tutorials/state/state-import?utm_source=WEBSITE&utm_medium=WEB_IO&utm_offer=ARTICLE_PAGE&utm_content=DOCS#define-import-block). 
However you will also need to create the configuration file for the 
imported resource. You can automate the generation of the configuration 
file using configuration-driven import. Create an import.tf, e.g.:

```bash
import {
  id = "resource_id"
  to = "azurerm_linux_virtual_machine.my_vm"
}
```

You can then run `terraform plan` to generate a corresponding configuration file:

```bash
terraform plan -generate-config-out=generated.tf
```
