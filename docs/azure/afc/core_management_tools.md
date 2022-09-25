# Core Management Tools

## Resource Management

Azure Portal

Azure Mobile App

Azure PowerShell

- Available on Windows, Linux and Mac

Azure CLI

- Available on Windows, Linux and Mac
- Good for one-off management, administrative or reporting tasks

CloudShell

- Use PowerShell or Azure CLI from a Web browser or mobile phone
- Run either Bash or Powershell environment
- PowerShell in CloudShell runs on an Ubuntu Linux container
- Terraform also available from CloudShell

ARM Templates

- Define infrastructure for repeatable deployments
- Declarative syntax
- Resource deployments are created in parallel
- Validation stage checks for dependancies and allows for easier rollback of deployments
- ARM templates are idempotent
- Can be used to trigger PowerShell or CLI scripts

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

