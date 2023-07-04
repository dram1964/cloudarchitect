# Azure Monitor

Azure Monitor is based on a
[common monitoring data platform](https://learn.microsoft.com/en-us/azure/azure-monitor/data-platform)
that enables you to collect and analyse Logs and Metrics from multiple resources. Logs are
useful for complex analysis using log queries. Metrics support real-time monitoring. 
Use Monitor > Activity Logs to see all activity for the Tenant. You can add filters for 
Resource, Resource Group, Resource Type, Operation and Events. 

Azure Monitor data is collected and stored in tables based on the [resource provider](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-services-resource-providers) 
namespaces (which is part of each resource id) using two data stores: Metrics and Logs. Platform
Metrics are automatically collected and sent to Azure Monitor, but Resource logs need to be 
configured for collection and storage. Resource Logs can be configured by adding 
Diagnostic Settings to the resource. Diagnostic settings define the Source (type of log and 
metric data) and Destination (Log Analytics Workspace, Storage Account, Event Hub). The same
Source can be routed to one of each destination type per diagnostic setting. To route the
same Source to multiple destinations of the same type, you will need to configure a 
diagnostic setting per destination. A maximum of 5 diagnostic settings can be configured
per resource.
Change Analysis monitors alerts you to service issues, outages, component failures or
other change data, and can be enabled in Change Analysis in the Azure portal. 

Azure Monitor includes Insights which are out-of-the-box monitoring and troubleshooting 
experiences.

Azure Monitor allows you to visualise data in Workbooks and Dashboards. Dashboards allow you
to combine different kinds of data (Visualisations) into a single pane in the Azure Portal. 
Azure Workbooks is a feature of Azure Monitor that provides a flexible canvas for analysing and
visualising collected data. Workbooks allow you to combine data from multiple sources in a single
report. Workbooks support data from:

- Logs
- Metrics
- Azure Resource Graph
- Alerts
- Workload Health
- Azure Resource Health
- Azure Data Explorer

Workbooks are provided with Insights or can be built using pre-defined templates. 

Monitor supports alerts and event-driven actions, for example run-books or autoscale. Azure 
Monitor can integrate with Event Hubs. 
Alerts can be sent over email or text message. 
Action groups can be configured for alerts, defining the list of recipients and actions to take. 

You can access Monitor either from the Monitoring section of a resource blade or using the 
Monitor service.

Azure Monitor is enabled by default - as soon as you create a subscriptions and start to add
resources, Azure Monitor starts collecting data. Activity logs record when resources are
created or modified. Metrics tell you about the resource performance. Azure Monitor Metrics 
data are retained for 3 months (93 days).

Some services require an agent (Azure Monitor Agent) installed to collect metrics and logs:

- Azure Cloud Services
- Azure Virtual Machines
- Azure Virtual Machine Scale Sets
- Azure Service Fabric

The Azure Monitor agent has replaced the Log Analytics agent, the diagnostic extension, and the
Telegraf agent.

Log data can be consolidated in a Log Analytics Workspace and allow you to keep data for up to 
2 years. The Workspace must be configured 
initially and then the different sources need to be configured to send their data to 
the Workspace. 

Additionally, applications will require the Application Insights SDK or auto-instrumentation 
(via an agent) to collect information and write it to the Azure Monitor data platform.

Azure Monitor stores log data in Azure Monitor Logs (aka Log Analytics) workspace. A workspace
is an Azure resource that serves as an administrative boundary or geographic location for 
data storage. The workspace is also a container where you collect and aggregate data. In a 
workspace, you can isolate data by granting different access rights to users. Log data is
organised into tables. Each workspace allows configuration of settings such as: 

- pricing tier
- retention
- data capping

RBAC roles are used to restrict user access to data in the logs workspace. Workspaces are hosted
on dedicated clusters which are automatically deployed and managed by Azure. Custom clusters
can be used if you require greater control or ingestion rates greater then 500GB. 

Planning Considerations:

- Do you need logs stored in specific regions for data sovereignty
- Avoid outbound data transfers by having a workspace in the same region as the resources it manages
- Does the system support multiple departments or business groups or do you need a consolidated view

Deployment Models

- Centralised: All logs stored in a central workspace and administered by a single team. Administrative overhead to maintain access control per user. Also known as the *hub-and-spoke* model
- Decentralised: Each team has their own workspace created in a resource group they own and manage. Log data is segregated per resource. Difficult to analyse data across groups
- Hybrid: implements both centralised and decentralised models. Can become complicated to manage and still results in problems accessing data across groups

Access Modes

- Workspace Context: users can view all logs in their workspace
- Resource Context: users access logs for specific resources, resource groups or subscriptions in their workspace. Queries are scoped to only data associated with the resources they have access to. The resouce-context model is a new feature, where log data preserves user access roles from the resource.

Effective monitoring combines Azure Insights with Azure Workbooks. 

## Metrics Explorer

The Metrics item in the menu for any Azure resource, allows you to access charts
of the metrics collected for the resource. Selecting a graph opens the Metrics
Explorer. From the Metrics Explorer, you can chart values from multiple metrics
and pin them to a Dashboard.

## Data Explorer

Azure Data Explorer is a fast and highly scalable data exploration service for log and telemetry
data. It can handle multiple data streams from diverse sources. Data Explorer is designed for
big data analytics providing support for near-real-time analytics, pattern recognition, time
series analysis, anomaly detection, forecasting and machine learning. ADE is integrated with 
ML Services such as Databricks and AML to allow the building and deployment of models. Supports
cost effective storage of long-term data. 

# Further Reading

- [sources of data in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/data-sources)
- [Infrastructure metrics and logs](https://learn.microsoft.com/en-us/azure/architecture/framework/scalability/monitor-infrastructure)
- [Performance Efficiency](https://learn.microsoft.com/en-us/azure/architecture/framework/scalability/monitor)
- [Azure Data Explorer](https://learn.microsoft.com/en-us/azure/data-explorer/data-explorer-overview)
- [Holistic Monitoring Strategy](https://learn.microsoft.com/en-us/training/modules/design-monitoring-strategy-on-azure/)
- [Incident Response](https://learn.microsoft.com/en-us/training/modules/incident-response-with-alerting-on-azure/)
- [Intro to Data Explorer](https://learn.microsoft.com/en-us/training/modules/intro-to-azure-data-explorer/intro-to-azure-data-explorer/)
- [Hands-On Storage Troubleshooter](https://learn.microsoft.com/en-us/training/modules/monitor-diagnose-and-troubleshoot-azure-storage/)
- [Design a Log Analytics Architecture](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/workspace-design)

Both Defender for Cloud and Microsoft Sentinel use Azure Monitor Logs as their underlying 
logging data platform. 

## Defender for Cloud

Defender for Cloud can be used to monitor the security of your workloads. Defender for Cloud
integrates well with Azure PaaS services and can be enabled for autoprovisioning on IaaS 
service (via an agent) such as VMs.

Use Defender for Cloud to:

- Understand the Security Posture of your Architecture
- Indentify and address risks and threats
- Secure a complex infrastructure
- Secure Hybrid infrastructure

Defender for Cloud uses Azure Monitor Logs to collect data. Data is collected using the 
Log Analytics agent. 

Defender for Cloud gives detailed analyses of data security, network security, identity and
access, and application security allowing you to build better infrastructure through 
recommendations. 

Defender for Cloud supports Just-In-Time access controls for VMs. Adaptive Application Controls
allow you to control which applications are allowed to run on your VMs.

Defender for Cloud provides security alerts for your resources. These can be drilled into and
allows actions to be taken based on events that are grouped into incidents. 

## Application Insights

Distributed traces are a series of related events that follow a user request through a 
distributed system. Distributed tracing in Azure Monitor is enabled through the Application 
Insights SDK or installing the correct agent or library. 

Application Insights can be used to:

- Analyse issues with your applications
- Improve application development lifecycle
- Measure user experience and analyse user behaviour

Instrumentation refers to the collection of monitoring data. You can enable instrumentation 
for Application Insights in your apps by using an agent or SDK. For some App Services, 
Application Insights can be enabled from a toggle in the Application Insights blade for 
the App Service. Application Insights collects instrumentation and displays 
interactive visualisations. Application Insights also detects app dependancies to support
distributed tracing and application topology views (Application Map). The Performance pane
allows you to track slow requests. By adding instrumentation to your web pages via JavaScript,
you can collect usage information such as number of users, sessions, events, OS versions and
user location. The Availability pane allows you to create availability tests to continuously
monitor the health of your app. Alerts can be configured with alert rules. 

Application Insights can be used to monitor resource usage and performance for VMs and 
Kubernetes using Azure Monitor VM Insights and Azure Monitor Container Insights. Enabling
VM Insights adds the required extensions and configuration to your VMs and VM Scale Sets. 
Similarly you can monitor the health of controllers, nodes and containers in your 
Kubernetes clusters using Container Insights.

Application Insights Hub provides guided troubleshooting to triage and isolate issues by 
service category. 

## Microsoft Sentinel

Microsoft Sentinel can be used to collect data across the enterprise and allow you to perform
threat and anomaly hunting. Incidents allow you to group related alerts. Playbooks can 
be used to automate your response to alerts. Hunting queries can be built either from 
scratch or from template queries developed by Microsoft security researchers. Azure Notebooks
for Microsoft Sentinel are playbooks consisting of investigation or hunting steps. 

Use Microsoft Sentinel to:

- Get a detailed overview of your Enterprise
- Avoid having to use disparate tools
- Use enterprise-grade AI to identify and handle threats

Deploy Sentinel in the Azure Portal by creating a Log Analytics Workspace and adding it to
Sentinel. Use Connectors to link data sources to Sentinel. Connectors exist for both Microsoft
and Non-Microsoft data sources. Sentinel also provides a REST API to connect other sources. 
When you connect a data source, your logs will be synched to Sentinel. 

Sentinel allows you to add alert rules and specify their severity. Alerts are grouped into 
incidents, to ease investigation. When you want to investigate an incident, you can change
its status from New to In Progress. Clicking the 'Investigate' link, will display an
Investigation Map. The Investigation Map will also show a timeline for related events. 

Automate response to incidents using Playbooks. Playbooks use Logic Apps connected via triggers
to Sentinel alerts. Configure the Logic App trigger to be 'Sentinel Alert triggered' and add
the desired actions (e.g. block user account or ip address). When the playbook is configured, 
it can be added to the Sentinel Alert rule, in the Automated Response section. 

## CLI commands for [Azure Monitor](https://learn.microsoft.com/en-us/cli/azure/monitor/log-analytics/)

- Create a workspace for Monitor Logs

        az monitor log-analytics create --resource-group $RG_NAME --workspace-name $WS_NAME

- List tables in your workspace

        az monitor log-analytics table list --resource-group $RG_NAME --workspace-name $WS_NAME --output table

- Change the retention time for a table

        az monitor log-analytics table update --resource-group $RG_NAME --workspace-name $WS_name --name $TABLE_NAME --retention-time 45

- Create regular exports of Log Analytics data to a Storage Account with [az monitor log-analytics data-export create](https://learn.microsoft.com/en-us/cli/azure/monitor/log-analytics/workspace/data-export#az-monitor-log-analytics-workspace-data-export-create)

- Create links between workspaces and Azure resources with [az monitor log-analytics workspace linked-service create](https://learn.microsoft.com/en-us/cli/azure/monitor/log-analytics/workspace/linked-service)

- To create and manage your own Storage Account linked to Azure monitor use [az monitor log-analytics linked-storage](https://learn.microsoft.com/en-us/cli/azure/monitor/log-analytics/workspace/linked-storage?view=azure-cli-latest)

- To manage intelligence packs use [az monitor log-analytics workspace pack](https://learn.microsoft.com/en-us/cli/azure/monitor/log-analytics/workspace/pack?view=azure-cli-latest)

- To manage saved searches in log analytics use [az monitor log-analytics workspace saved-search](https://learn.microsoft.com/en-us/cli/azure/monitor/log-analytics/workspace/saved-search?view=azure-cli-latest)

- Workspaces can be deleted with [az monitor log-analytics workspace delete](https://learn.microsoft.com/en-us/cli/azure/monitor/log-analytics/workspace?view=azure-cli-latest#az-monitor-log-analytics-workspace-delete). Use the --force option for a hard delete, otherwise the workspace will be recoverable for 2 weeks after deletion

- Create a Diagnostic Setting

        az monitor diagnostic-settings create  \
        --name KeyVault-Diagnostics \
        --resource /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault \
        --logs    '[{"category": "AuditEvent","enabled": true}]' \
        --metrics '[{"category": "AllMetrics","enabled": true}]' \
        --storage-account /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount \
        --workspace /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myresourcegroup/providers/microsoft.operationalinsights/workspaces/myworkspace \
        --event-hub-rule /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhub/authorizationrules/RootManageSharedAccessKey


## Terraform Scripts for Azure Monitor

Create a Log Analytics Workspace

    resource "azurerm_log_analytics_workspace" "log-central" {
      name                = "log-central"
      resource_group_name = "rg-log-analytics"
      location            = "uk_south"
      sku                 = "PerGB2018"
      retention_in_days   = 90

      tags = var.tags

    }

Create a Key Vault Diagnostic Setting

    resource "azurerm_monitor_diagnostic_setting" "diag-key-vault-01" {
        name                       = "diag-key-vault-01"
        target_resource_id         = azurerm_key_vault.key-vault-01.id
        log_analytics_workspace_id = azurerm_log_analytics_workspace.log-central.id

        log {
            category       = ""
            category_group = "audit"
            enabled        = true

            retention_policy {
            enabled = true
            days    = 0
            }
        }

        log {
            category       = ""
            category_group = "allLogs"
            enabled        = false

            retention_policy {
            days    = 0
            enabled = false
            }
        }

        metric {
            category = "AllMetrics"

            retention_policy {
            enabled = false
            }
        }
    }

## Platform Logs

Platform Logs are available at different layers of Azure:

1. Resource Logs are for Azure Resources and provide insight into operations that were performed
inside an Azure Resource (e.g. retrieving data from a database or access a key vault secret). This layer is referred to as the **data plane**
2. Activity Logs are Azure Subscriptions and provide insight into operations performed on each
resource in the subscription from the outside (e.g. configuration changes). This layer is 
referred to as the **management plane**.
3. AAD Logs provides information on sign-in and changes to AAD for the tenant.

## Log Analytics Charging

Log Analytics and Application Insights charge for the data they ingest. The Pay-As-You-Go tier
is charged at £2.48 per GB per day and allows up to 5GB of data per billing account. Commitment
tiers provide savings if the volume of data matches the commitment tier. For example the
100GB per day commitment is charged at £211.56 per day and works out as £2.12 per GB per day
assuming that you consume 100GB per day. 

Additional ingestion charges are made for:

1. Basic Log ingestion: for logs that can be managed without full analytics capabilities. Charged
at £0.54 per GB of data ingested and £0.006 per GB of data searched
2. Log Data Retention: for logs that are stored for archive purposes. Charged at £0.022 per GB
per month and £0.006 per GB of data searched. Archived logs can be restored to enable full
log analytics capabilities and this is charged at £0.108 per GB per day
3. Log Data Exports: Log Analytics data that is exported is charged at £0.108 per GB 
4. Web Tests: Application Insights provides tests for service health which are charged at:

    - £0.0006 per scheduled web test execution
    - Free for ping testing
    - £8.635 for test execution for multi-stage web tests
5. Platform logs sent to Log Analytics attract no additional cost over the ingestion cost. When
the logs are sent to Event Hub or Storage or MarketPlace Partners there is a charge of £0.270 per GB. 
6. Log alerts and Notifications also attract additional charges. See the [Log Analytics Pricing](https://azure.microsoft.com/en-gb/pricing/details/monitor/) page for full details
