# References

- [How to query Azure resources using the Azure CLI](https://techcommunity.microsoft.com/t5/itops-talk-blog/how-to-query-azure-resources-using-the-azure-cli/ba-p/360147)

# JMES Query Syntax

By default, Azure CLI queries return JSON output. You can change the output format
using the --output switch. Typical values are either 'table' or 'tsv'.

The query option can be used to filter the fields returned. Where a command 
returns a single row in table output, you can query by field name

`az account show --query id`

For commands returning more than one row or object, you need to use the 'flatten' operator in the query

`az account list --query [].id -o table`

To return multiple values in the output, specify these in a comma separated list:

`az account list --query [].[id,name] -o table`

You can rename the columns by specifying a dictionary instead of an array: note that the query is now in double-quotes:

`az account list --query "[].{Subscription:id,Name:name}" -o table`

You can also specify an index value to return a specific row: indexes begin at 0 for the first element:

`az account list --query "[1].{Subscription:id,Name:name}" -o table`

An array slice can also be used:

`az account list --query "[1:4].{Subscription:id,Name:name}" -o table`

The slice specification equates to: "return the element at index 1 and all 
elements up to, but not including, the element at index 4". Three elements (4 - 1) are returned 
In addition to using array indexing, you can filter the results using filter 
expressions:

`az account list --query "[?name == 'mySubscription')].{Subscription:id,Name:name}" -o table`

`az account list --query "[?contains(name, 'sub')].{Subscription:id,Name:name}" -o table`

So far we've been selecting fields that return a scalar value. However some fields will be dictionaries: use the dot operator to select a single element from each dictionary:

`az account list --query "[].[user.type]" -o table`

If the query returns a single row, you can use the tsv output format to assign the value to a variable or to directly use the value in a command:
	
    SUB=$(az account list --query "[?contains(name, 'research')].[id]" -o tsv)
    `az account set --subscription $SUB`

    az account set --subscription $(az account list --query "[?contains(name, 'research')].[id]" -o tsv)`
	

# Storage
Create a storage account and container
	
    RESOURCE_GROUP_NAME={{ page.resgroup }}
    LOCATION=uksouth
    STORAGE_ACCOUNT_NAME={{ page.stgname }}
    CONTAINER_NAME={{ page.containername }}

    az group create --name $RESOURCE_GROUP_NAME --location $LOCATION

    az storage account create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $STORAGE_ACCOUNT_NAME --sku Standard_LRS \
    --encryption-services blob

    az storage container create \
    --name $CONTAINER_NAME \
    --account-name $STORAGE_ACCOUNT_NAME
	

Upload a file to blob storage:

	az storage blob upload \
    --account-name $STORAGE_ACCOUNT_NAME \
    --container-name $CONTAINER_NAME \
    --file $FILE_NAME --name $CONTAINER_FILE

Upload a folder to blob storage:

	az storage blob upload-batch \
    --destination $CONTAINER_NAME \
    --account-name $STORAGE_ACCOUNT_NAME \
    --destination-path $CONTAINER_FOLDER \
    --source $LOCAL_FOLDER

List Storage Account keys:

`az storage account keys list --account-name $STORAGE_ACCOUNT_NAME`

Generate and capture SAS keys to access a storage container:
	
    ACCOUNT_KEY=$(az storage account keys list \
    --resource-group $RESOURCE_GROUP_NAME \
    --account-name $STORAGE_ACCOUNT_NAME 
    --query '[0].value' -o tsv)

    END_DATE=date -u -d "1 year" '+%Y-%m-%dT%H:%MZ'

    SAS_KEY=$(az storage container generate-sas \
    -n $CONTAINER_NAME \
    --account-key $ACCOUNT_KEY \
    --account-name $STORAGE_ACCOUNT_NAME \
    --https-only \
    --permissions dlrw \
    --expiry $END_DATE -o tsv)

The 'azcopy' command can also be used to move data quickly into and out of Azure 
Storage Accounts. Logon to Azure:

    azcopy login

Create a container:

    azcopy make 'https://${STORAGE_ACCOUNT_NAME}.blob.core.windows.net/${CONTAINER_NAME}'

Copy a folder to the container (creates the *data_folder* folder in the target container):

    azcopy copy '/data/data_folder' 'https://${STORAGE_ACCOUNT_NAME}.blob.core.windows.net/${CONTAINER_NAME}'
	

# Deploying ARM Templates
Deploy infrastructure from an ARM Template:
	
    RESOURCE_GROUP=$RESOURCE_GROUP_NAME
    TEMPLATE_FILE=/path/to/template.json

    az group deployment create \
    --name blanktemplate \
    --resource-group $RESOURCE_GROUP \
    --template-file $TEMPLATE_FILE
	

You can start with a blank template:
	
    {
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "functions": [],
    "variables": {},
    "resources": [],
    "outputs": {}
    }
	

You can then add resources as you expand the deployment. For instance to add a storage account, 
add a storage account resource to the template:
	
    "resources": [{
        "name": "learn132uksstorage",
        "type": "Microsoft.Storage/storageAccounts",
        "apiVersion": "2021-04-01",
        "tags": {
            "displayName": "learn132uksstorage"
        },
        "location": "[resourceGroup().location]",
        "kind": "StorageV2",
        "sku": {
            "name": "Standard_LRS",
            "tier": "Standard"
        }
        "properties": {
            "supportsHttpsTrafficOnly": true
        }
    }],
	

Then you can create a new deployment using:
	
    templateFile="azuredeploy.json"
    today=$(date +"%d-%b-%Y")
    DeploymentName="addstorage-"$today

    az deployment group create \
    --name $DeploymentName \
    --resource-group $RESOURCE_GROUP \
    --template-file $templateFile
	

You can make your template re-useable by using parameters instead of hard-coded values. For instance,
to use a parameter to specify the Storage Account type, add the following parameter to the parameters
section:
	
    "parameters":{
    "storageAccountType": {
        "type": "string",
        "defaultValue": "Standard_LRS",
        "allowedValues": [
            "Standard_LRS",
            "Standard_GRS",
            "Standard_ZRS",
            "Premium_LRS"
        ],
        "metadata": {
        "description": "Storage Account type"
        }
    },
    },
	

Then replace the hard-coded value for the Storage Account type:
	
    "resources": [{
        "name": "learn132uksstorage",
        "type": "Microsoft.Storage/storageAccounts",
        "apiVersion": "2021-04-01",
        "tags": {
            "displayName": "learn132uksstorage"
        },
        "location": "[resourceGroup().location]",
        "kind": "StorageV2",
        "sku": {
            "name": "[parameters('storageAccountType')]"
        }
        "properties": {
            "supportsHttpsTrafficOnly": true
        }
    }],
	

When you deploy the template, you can give a value for the parameter, otherwise the default value
is used:
	
    templateFile="azuredeploy.json"
    today=$(date +"%d-%b-%Y")
    DeploymentName="use-stg-parameter-"$today

    az deployment group create \
    --name $DeploymentName \
    --resource-group $RESOURCE_GROUP \
    --template-file $templateFile
    --parameters storageAccountType=Standard_LRS
	

Template deployments also allow you to specify outputs that will be returned after a successful 
deployment. The reference() function can be used to retrieve the runtime state of the resource:
	
    "outputs": {
      "storageEndpoint": {
        "type": "object",
        "value": "[reference('learntemplatestorage123').primaryEndpoints]"
      }
    }
	

We can also add a parameter for the Storage Account Name, and additionally specify allowed values for
any of our parameters. A completed ARM template could now look something like this:
	
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
        "storageName": {
            "type": "string",
            "minLength": 3,
            "maxLength": 24,
            "metadata": {
            "description": "The name of the Azure storage resource"
            }
        },
        // This is the allowed values for an Azure Storage account
        "storageSKU": {
            "type": "string",
            "defaultValue": "Standard_LRS",
            "allowedValues": [
                "Standard_LRS",
                "Standard_GRS",
                "Standard_RAGRS",
                "Standard_ZRS",
                "Premium_LRS",
                "Standard_GZRS",
                "Standard_RAGZRS"
            ],
            "metadata": {
                "description": "Allowed SKUs for Storage Accounts"
            }
        }
        },
        "functions": [],
        "variables": {},
        "resources": [{
            "name": "[parameters('storageName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2021-04-01",
            "tags": {
                "displayName": "[parameters('storageName')]"
            },
            "location": "[resourceGroup().location]",
            "kind": "StorageV2",
            "sku": {
                "name": "[parameters('storageSKU')]",
                "tier": "Standard"
            }
        }],
        "outputs": {
            "storageEndpoint": {
                "type": "object",
                "value": "[reference(parameters('storageName')).primaryEndpoints]"
            }
        }
    }
	

# Service Principals
Create a service principal with Contributor role on the specified subscription:

	az ad sp create-for-rbac \
    --scopes /subscriptions/$SUBSCRIPTION_ID \
    --role Contributor --name $SP_NAME

Grant keyvault permissions to a service principal:
	
    SP_OBJECT_ID=$(az ad sp list --display-name $SERVICE_PRINCIPAL --query [].objectId -o tsv)
    az keyvault set-policy --name $KEYVAULT_NAME \
    --secret-permissions get list --object-id $SP_OBJECT_ID
	

# Key Vaults
Create a Key Vault and add a secret:
	
    az keyvault create --name $KEYVAULT_NAME \
    --resource-group $RESOURCE_GROUP_NAME \
    --location $LOCATION --tags $TAG_VALUE

    az keyvault secret set \
    --vault-name $KEYVAULT_NAME \
    --name client-id --value $SECRET_VALUE
        

# Configure Defaults
The az configure command can be used to set and unset default parameter values. To set
a default resource group for each az command use:

`az configure --defaults group=$RESOURCE_GROUP`

To unset the default use:

`az configure --defaults group=''`

To list current defaults use:

`az configure --list-defaults`

