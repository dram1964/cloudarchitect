# Compute

## [Virtual Machines](https://learn.microsoft.com/en-us/azure/virtual-machines/) - IaaS service

Consider the following when planning VM deployment:

- Network
    - Network addresses and subnets are difficult to re-configure. Consider network topology before deploying any VMs
- Name for VM
    - VM names can be 15 characters long for Windows machines and up to 64 characters for Linux machines. The name should include components that identify:
        - resouce type (vm)
        - Environment (dev, test, prod)
        - Location
        - Instance
        - Product or Service (Outlook, SQL, etc.)
        - Role (security, web, messaging, etc.)
- Location of VM
    - VMs should be deployed in a Region close to the customer, to ensure best performance and meet any regulations around data residency. However, not all machine sizes are available in all regions and prices for the same machine size can vary between regions
- Size of VM
    - Choose a machine size to match the expected workload. VMs can be re-sized to provide an agile, elastic approach to machine sizing. Resizing a VM can require a restart or result in a change of IP
- Estimate Cost
    - Two separate costs associated with each VM: 
        - compute 
            - priced per-hour but billed per-minute. No charge for VMs that are stopped and deallocated. Reserved instances can result in reduced costs over pay-as-you-go, if you know the VM needs to run continuously or you need budget-predictability. RI VMs can be exchanged or returned for an early-termination fee. 
        - storage
            - storage charges still apply even when the VM is stopped or deallocated. 
- Choose Storage
    - All VMs in Azure have at least two disks: an operating system disk and a temporary disk. Additional data disks can also be added. Premium storage disks provide higher throughput and lower latencies. Use multiple data disks for even higher storage and response times. Manage disks are managed by Azure. All disks are stored as Virtual Hard Disks (VHDs): 
        - OS Disk: contains the pre-installed OS. Registered as a SATA drive. 
        - Temporary Disk: On Windows machines this is labelled `D:` and used to store the `pagefile.sys`. On Linux machines this is registered as `/dev/sdb` and mounted on `/mnt`.
        - Data Disks: the number of data disks and that can be attached to a VM and their type is determined by the VM size. 
- Choose OS
    - Azure bundles the cost of the OS into the price of the VM. [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute) offers images with various application stacks pre-installed. Custom images can be created and uploaded to Azure Storage. Azure only supports 64-bit operating systems.

When configuring a VM via the Portal, a number of tabs are available:

- Basics: project details, admin account and inbound port rules
- Disks: select OS disk and data disks
- Networking: VNets and Load Balancer
- Management: auto-shutdown and backups
- Advanced: configure agents, scripts and VM extensions
- Monitoring
- Tags

Virtual Machine Extensions provide post-deployment configuration and automation tasks for VMs.
VM Extensions can be bundled with the VM deployment or run against existing VMs. VM extensions 
can be managed from the CLI, Powershell, ARM templates and the Portal. 

Custom Scripts Extensions can be added to your VMs. After the Custom Script Extension is added
to the VM, you can provide a Powershell script file to execute commands on the VM. The script files
can be downloaded from Azure Storage or GitHub. Custom Script Extensions have a default timeout value
of 90 minutes.  The script can also be run from the command line:

    Set-AzVmCustomScriptExtension -FileUri https://scriptstore.blob.core.windows.net/scripts/Install_IIS.ps1 -Run "PowerShell.exe" -VmName vmName -ResourceGroupName resourceGroup -Location "location"

Desired State Configuration is a management platform in Windows Powershell. DSC provides a 
declarative syntax for specifying machine configuration. 

Additional Resources:

- [Linux VM Sandbox](https://learn.microsoft.com/en-us/training/modules/create-linux-virtual-machine-in-azure/)
- [Windows VM Sandbox](https://learn.microsoft.com/en-us/training/modules/create-windows-virtual-machine-in-azure/)

## [VM Types]("https://docs.microsoft.com/azure/virtual-machines/sizes")

- General Purpose (A, B, D family): balanced CPU-to-Memory ratio. Good for testing, development, small-to-medium databases, low-to-medium traffic web-servers
    - A Series: for tesing and development
    - B Series: burstable workloads
    - D Series: general-purpose workloads
- Compute Optimised (F family): high CPU-to-Memory. Best suited to workloads where CPU is more likely to be a bottle-neck. For example, web servers, application servers, network appliances and batch processes
- Memory Optimised (E, G, M family): high Memory-to-CPU ration. Best suited for memory-intensive workloads. For example, relational databases, in-memory anlytics
- Storage Optimised (L family): have high disk-throughput and I/O. Best suited for data analytics, data warehousing, big-data
- GPU Optimised (N family): have GPUs. Best suited for compute-intensive workloads or graphics rendering or video editing. Typical examples are model training, inference with deep learning, graphics, gaming, visualisations, video-conferencing, video-streaming workloads
- High Performance (H family): most powerful CPU VMs available providing high-speed network throughput. Suitable for compute and network workloads. For example SAP Hana

VM types use the following naming convention to indicate their capabilities:

- Naming convention: [Family] + [Sub-Family] + [# of vCPUs] + [Constrained vCPUs] + [Additive Features] + [Accelerator Type] + [Version]
    - Additive Features are:
    - a: AMD-based processor
    - b: Block Storage performance
    - c: confidential
    - d: VM has local temp disk
    - i: isolated size
    - l: low-memory state
    - m: memory-intensive size
    - s: premium storage capable (although newer sizes without 's' can also support premium storage)
    - t: tiny memory size
    - NP: node packing
    - P: ARM CPU

Examples:

- B2ms - B-series, 2 vCPUs, memory-intensive, premium storage
- D4ds v4 - D-series, 4 vCPUs, local temp disk, premium storage, version 4

## VM Availability

VM resources need to be configured to provide consistent response times and with high availability. 
Azure offers several options to meet these requirements. 

An availability plan should handle responses to:

- Unplanned hardware maintenance - Azure issues an unplanned hardware maintenance event when a 
resource component is predeictied to fail. Live Migration is used to migrate a VM from failing 
hardware to healthy resources and can result in reduced performance during the migration
- Unexpected downtime - occurs when hardware fails unexpectedly
- Planned maintenance - periodic updates to VM Host machines normally have minimal impact on VM availability

An Availability Set is a logical grouping of Virtual Machines performing the same tasks. 
Availability sets run your VMs across multiple physical servers, compute racks, storage units 
and network switches. If a hardware or software failure occurs, only a subset of your VMs will be
affected. By grouping machines into availability sets, you ensure that not all 
the machines are upgraded at the same time during a host operating system upgrade. A VM can only 
be added to an availability set at create time. To add an existing VM, you would need to delete and
recreate it. When using availability sets consider:

- Redundancy: place mutliple VMs in the Availability Set
- Application Tiers: each application tier (front-end, db, controllers, etc) should be placed in its own Availability Set
- Load Balancing: use Load Balancer to distribute incoming traffic to healthy servers
- Managed Disks for block level storage

Each VM in an Availability Set is placed in one update domain and one fault domain. An update
domain is a group of nodes that are upgraded together during a service upgrade: only one update
romain is rebooted at a time. There are 5 non-user-configurable update domains, but you can configure
up to 20. A fault domain is a group of nodes that share a common set of hardware that represents 
a single point of failure.

An Availabilty Zone is a combination of a fault domain and an update domain. By placing your
VMs across multiple Availability Zones, you are spreading your VMs over multiple fault domains
and multiple update domains. Availability Zones are unique physical locations within an Azure
region with independant power, cooling and networking. There is a minimum of three separate
zones in all enabled regions. Zonal services such as VMs, disks and Standard IP addresses, pin 
each resource to a specific zone. Zone-redundant services such as zone-redundant Storage or SQL 
Database are automatically replicated across all zones. 

Scalability for VMs can be either:

1. Vertical: (scale up and scale down) involves increasing or decreasing VM size in response to workloads
2. Horizontal: (scale out and scale in) involves increasing or decreasing the number of VMs to support changing workload. 

Vertical scaling depends on the availability of larger hardware and usually requires stopping and 
restarting a VM. 

VM Scale Sets allow you to deploy and manage a set of identical VMs. VMSS can automatically 
increase or decrease the number of VM instances to provide true auto-scaling. VMSS can be 
scaled automatically, manually or both. VMSS supports use of both Azure Load Balancer or Azure 
Application Gateway for layer-4 or layer-7 distribution. VMSS can support up to 1000 instances or 
600 instances if using custom VM images. 

When configuring a VMSS, you define:

- Scaling Policy: Manual or Auto and minimum and maximum instances allowed
- Scale Out: CPU threshold, duration and number of instances to increase by
- Scale In: CPU threshold, duration and number of instances to decrease by
- Scale-in Policy: determines the order in which VMs are removed. VMs are balanced across availability zones and fault domains, then deleting the VM with the highest instance ID (default) or the newest or the oldest VM.

## Azure Container Instances 

Azure Container Instances is a PaaS service to run your container instances. ACI provides:

- Fast Startup Times
- Public IP and DNS connectivity
- Custom Sizes: nodes can be scaled dynamically
- Persistent Storage: using Azure Files
- Virtual Network Deployment: containers can be deployed to a VNET

A Container Group is a collection of containers that get scheduled on the same 
host machine and share lifecycle, resources, local network and storage. Container
Groups work similarly to a Docker compose project. Container 
groups can be deployed via ARM templates or a YAML file. The YAML file approach
is only recommended when deploying Container Instances with no additional resources
such as file shares. Container groups share an IP Address, port range and FQDN. 

Container Instances uses Docker and thus ensures that the containers run 
in the same platform both locally and in Azure. 

Container Instances are deployed with the following default FQDN: `https://${DNS_NAME_LABEL}.${LOCATION}.azurecontainer.io` 

Azure Kubernetes Service - PaaS service for hosting and orchestrating containers

- The master node or control plane is managed by Microsoft
- The worker nodes containing pods are managed by the customer
- VMs are defined to run the nodes and pods
  
Azure Service Fabric

Azure Batch - large-scale, high-performance batch jobs

- Starts pool of compute VMs
- Installs applications and data
- Runs jobs
- Identifies failures
- Requeues work
- Scales down as work completes
  
Azure Functions - serverless compute

- Event-driven execution of code: REST request, timer or message from another application
- Intended for code that can be executed in seconds
- Coding support for C#, Python, JavaScript, Typescript, Java and Powershell
- Functions can be created in the Portal, VS Code or using Azure CLI
- Automatic scaling to meet demand
- Stateless environment
- Durable Functions allow Functions to be chained together whilst maintaining state
- Functions allow developers to focus on the code, without worrying about the underlying infrastructure
  
Azure Logic Apps - serverless orchestration

- Event-driven execution of workflows
- Low Code or No Code development platform
- Used to automate and orchestrate tasks, business processes and workflows
- Integration solution for apps, data and systems
- Web-based designer
- Apps are triggered by other Azure services using connectors
- Gallery includes over 200 connectors

Azure Virtual Desktop - use cloud-hosted versions of Windows from any location

- User profiles containerised using FSLogix
- Centralised identity management with Azure AD
- Enable MFA and Conditional Access
- Multi-session Windows 10 deployment
- Bring your own licences - Azure Hybrid Benefit
- Session Host VMs can be shared between multiple users or dedicated to a single user, using AAD group assignments
- Sessions can be allocated in depth mode (all sessions go to the same host until it reaches its limit) or breadth mode (sessions are automatically connected to the next host in the pool)
- VMs can be started when users connect to them or left permanently on

