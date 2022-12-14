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

Virtual Machine Scale Sets - IaaS service

- deploy and manage a set of identical, load-balanced VMs
- provides scaling and auto-scaling for VMs

Azure Container Instances - PaaS service to run your container instances

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

