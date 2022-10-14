# Core Management Tools

## Resource Management

### Resource Groups

- Put resources with the same lifecycle into a single resource group. Deleting a resource group deletes all the resources within it
- A resource group is logical collection of Azure resources. Use tags to further organise resources
- Once created, a resource group can not be renamed
- Resources can be moved from one resource group to another. When moving resource between resource groups the resource groups are locked for write and delete operations until the move completes
- Not all resources support being moved
- The resources in a resource group can reside in different regions
- The location for a resource group determines where the resource group metadata resides
- Resources can interact with resources in other resource groups
- Resource groups can be used as a scope for access permissions

### Azure Resource Manager

- Provides a consistent management layer for actions from PowerShell, CLI, Portal, REST clients and SDKs
- The Azure Resource Manager API passes requests to the Azure Resource Manager service which authenticates and authorises the request
- The Azure Resource Manager service then passes the request to the appropriate resource providers
- Azure Resource Manager locks can be used to prevent changes or deletion of resources and groups

Resource providers offer a set of resources and operations for working with Azure services. The 
resource provider defines a set of REST operations. Common Resource providers include:

1. Microsoft.Compute
2. Microsoft.Storage
3. Microsoft.Web
4. Microsoft.KeyVault

Before using a resource provider it must be registered in your Azure subscription. Some 
resource providers are registered by default for each subscription. When you create a resource
through the Portal, the resource provider is automatically registered. Resources defined 
in ARM or Bicep templates also result in automatic registration of the corresponding 
resource provider. If the template results in the creation of services from that are not 
explicitly defined in the template, then the resource providers may need to be registered 
manually beforehand. 

Registering a resource provider requires the `/register/action` permission, 
which is included in the 'Contributor' and 'Owner' roles. You can view the current 
resource provider registrations in the 'Settings' menu for a subscription. 

Use Resource Explorer in the Portal to see the list of available Providers. Each provider
will have attributes detailing the resource types the service provides and the locations,
api versions and capabilities associated to the resource type. 

PowerShell provides the `Get-AzResourceProvider` and `Register-AzResourceProvider` cmdlets for working with resource providers:

    Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState 

    Get-AzResourceProvider -ListAvailable | Where-Object RegistrationState -eq "Registered" | Select-Object ProviderNamespace, RegistrationState | Sort-Object ProviderNamespace

    Register-AzResourceProvider -ProviderNamespace Microsoft.Batch

    Get-AzResourceProvider -ProviderNamespace Microsoft.Batch

    (Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes.ResourceTypeName

Azure CLI provides this functionality in the `az provider` command:

    az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table

    az provider list --query "sort_by([?registrationState=='Registered'].{Provider:namespace, Status:registrationState}, &Provider)" --out table

    az provider register --namespace Microsoft.Batch

    az provider show --namespace Microsoft.Batch

    az provider show --namespace Microsoft.Batch --query "resourceTypes[*].resourceType" --out table


### Azure Portal

- [Azure Portal Documentation](https://learn.microsoft.com/en-us/azure/azure-portal/)
- [Azure Portal Sandbox](https://learn.microsoft.com/en-us/training/modules/tour-azure-portal/)

Azure Mobile App

Azure PowerShell

- Available on Windows, Linux and Mac
- Comes as a module to add to Windows Powershell or Powershell Core 
- Az module is open-source and available on [github](https://github.com/Azure/azure-powershell)
- [Azure Powershell Reference](https://learn.microsoft.com/en-us/powershell/azure/?view=azps-6.5.0)
- [PowerShell Sandbox](https://learn.microsoft.com/en-us/training/modules/introduction-to-powershell/)

Azure CLI

- Available on Windows, Linux and Mac
- Good for one-off management, administrative or reporting tasks
- Use `az find <search_string>` to get help
- [Azure CLI Reference](https://learn.microsoft.com/en-us/cli/azure/)
- [Azure CLI Sandbox](https://learn.microsoft.com/en-us/training/modules/control-azure-services-with-cli/)

Cloud Shell

- Use PowerShell or Azure CLI from a Web browser or mobile phone
- Run either Bash or Powershell environment on an Ubuntu Linux container
- Terraform also available from CloudShell
- Runs on a temporary host on a per-session, per-user basis
- Times out after 20 minutes of inactivity
- Persists $HOME in a 5GB image held in the file share created the first time you run Cloud Shell

ARM Templates

- Define infrastructure for repeatable deployments using declarative templates
- You can deploy, update or delete resources using a template
- Templates can be re-used in different environments
- ARM templates allow you to work with resources in your solution as a group
- Resource deployments are created in parallel
- Validation stage checks for dependancies and allows for easier rollback of deployments
- Use RBAC to define access control to all services
- ARM templates are idempotent
- Can be used to trigger PowerShell or CLI scripts
- [ARM Sandbox](https://learn.microsoft.com/en-us/training/modules/control-and-organize-with-azure-resource-manager/)

## Resource Monitoring

Azure Advisor

- Included no-cost service
- Evaluates your Azure resources and offers recommendations:
    - <a href="https://docs.microsoft.com/azure/advisor/advisor-high-availability-recommendations" target="#">reliability</a>
    - <a href="https://docs.microsoft.com/azure/advisor/advisor-security-recommendations" target="#">security</a>
    - <a href="https://docs.microsoft.com/azure/advisor/advisor-performance-recommendations" target="#">performance
    - <a href="https://docs.microsoft.com/azure/advisor/advisor-operational-excellence-recommendations" target="#">operational excellence</a>
    - <a href="https://docs.microsoft.com/azure/advisor/advisor-cost-recommendations" target="#">reduce costs</a>
- Recommendations available in the Portal, through the API and via notifications
- Recommendations can also be downloaded from the Portal as CSV or PDF

Azure Monitor

- A service for collecting, analysing and acting on telemetry from both cloud and on-premises resources
- Used to maximise availability and performance of applications and services
- Included no-cost service
- Data collected in central repositories
- Data can be viewed in Azure Monitor Dashboard or viewed in PowerBi or Kusto queries
- Can be used to setup custom alerts
- Metrics are collected for a resource automatically as soon as the resource is created
- VMs and other compute resources require an agent (Azure Monitor Agent) to collect metrics from the guest OS
- Azure Monitor Logs collects log and performance data from configured resources. A Log Analytics workspace needs to be configured and then the different sources need to be configured to send their data to the workspace

Azure Service Health

- Free service providing alerts on:
    - Service Issues
    - Planned Maintenance
    - Health Advisories
    - Root Cause Analysis for outtages
- More detailed and tailored information than <a href="https://status.azure.com" target="#">Azure Status</a>

