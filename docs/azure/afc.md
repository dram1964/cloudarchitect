# Core Concepts

Cloud Computing: Delivery of computing services over the internet

Cloud Models

- Public Cloud: (Multi-Tenant) services available over the internet and available to anyone who wants to purchase them
- Private Cloud: (Single-Tenant) computing resources available exclusively to one business or organisation. Resources can be on-prem or hosted by a service provider
- Hybrid Cloud: combines Public and Private Cloud by allowing resources to be shared across both


Cloud Benefits

- High Availability. Availability is the percentage of time a resource is available to service a request. High Availability can be met by failing-over to a replica in the same Region. 
- Scalability: vertically or horizontally to manually handle additional workloads:
    - vertically: add RAM or CPU to existing resource
    - horizontally: adding more instances of the resource
- Elasticity: autoscaling, so that apps always have resources to meet changing demand of current workload
- Agility: rapid deployment as needs change
- Geo-Distribution: deploy apps to regions around the globe, to gain best performance for the region
- Disaster Recovery: backup, data replication and geo-distribution protects data from disasters. DR strategy relies heavily on Recovery Time Objective (RTO) and Recovery Point Objective (RPO). Disaster Recovery can be met by failing-over to a replica in another region. Replication provides for shorter RTOs and RPOs than can be achieved with backups.
    
Pay-As-You-Go pricing model

- lower operating costs
- run infrastructure more efficiently
- scale as need changes
- PAYG is considered Operational Expenditure (OpEx) as opposed to Capital Expenditure (CapEx) for Private Clouds
  
Cloud Service Models - Shared Responsibility Model

- IaaS: provider configures hardware, but customer is responsible for OS and Networking
- PaaS: provider configures platform, but customer is responsible for deploying applications to the environment. Development Tools, Databases and Business Analytics are PaaS offerings
- SaaS: provider manages application environment, but customer is responsible for their data, devices and accounts.
- Serverless: provider manages everything up to and including language and runtime. Customer responsible for code and logic. Serverless model is targetted at event-based solutions
  
Azure Portal

- designed for resilience and availability
- available in every Azure data centre
- Continuously updated
  
Regions and DataCentres

- All hardware used in Azure is contained in data centres distriubuted across the globe
- Datacentres are grouped into Regions
- Resources are deployed to Regions - no direct access to individual data centres
- A Region is an area containing at least one, but usually more data centres
- Datacentres within a region are nearby and connected via a low-latency network
- Availability Sets are physically separate hardware components within a data centre
    - Availability sets are used to deploy virtual machines to different fault domains and update domains       
            `az availability-set create -g $RESOURCE_GROUP -n $AV_SET_NAME --platform-fault-domain-count 2 --platform-update-domain-count 2`
        
    - Fault domains are physical groupings of resources that share the same power and networking.
    - Update domains are logical groupings used as an update/patching boundary: updates are only applied to one update domain at a time.

- Availability Zones are physically separate data centres within an Azure Region
    - Minimum of three zones per region
    - Each Availability Zone consists of one or more data centres with independant networking, power and cooling
    - Availability Zones within a Region are connected through high-speed, fibre-optic connections
    - Availability Zones act as isolation boundaries
    - Use availability zones to replicate applications and protect against outtages on a particular Availability Zone (local-redundancy or regional-redundancy)
    - Replication across availability zones increases costs by duplicating resources
- Zone-redundant services (SQL Database, zone-redundant storage) are automatically deployed across zones
- Zonal services (VMs, managed disks, IP Address) are deployed to a specific zone, but can be replicated to other zones
- Non-zonal services are globally available
- Region Pairs
    - Each Azure Region is automatically paired with another region in the same geography (UK, Europe, US), at least 300 miles away
    - A Geography can have multiple Regions and defines a data residency and compliance boundary
    - Region pairs are physically connected
    - Allows for failover between regions (geo-redundancy), and protects from regional disasters
    - Azure Site Recovery - protects against Region failures and ensures data residency boundaries
    - Replication across regions increases costs by duplicating resources and data transfers between zones. Data transfers within a Region are not billed: data transfers leaving a Region are billed.
- Storage Redundancy
    - LRS - locally-redundant storage: three copies of your data, replicated synchronously within a single physical location in the primary region
    - ZRS - Zone-redundant storage: three copies of your data, replicated synchronously across three availability zones in the primary region
    - GRS - geo-redundant storage: three copies of your data, replicated synchronously within a single physical location in the primary region. Also provides three copies of your data replicated asynchronously to a single physical location in the secondary region
    - GRS - geo-redundant storage maintains three copies of the data in both regions, but data is only accessible from the primary copy
    - RA-GRS allows read-access from both regions simultaneously
- Edge Zones - allows data processing and code to be executed closer to the end user. Not located in Azure Region data centres
    - Azure Edge Zones - Azure public cloud resources within Microsofts Point-Of-Presence edge-locations. Part of Microsoft's global network
    - Azure Edge Zones with carrier - Azure public cloud resources within Carriers data centre locations. Part of the Carriers global network
    - Azure Private Edge Zones - Azure Stack (private cloud) resources within the customers location or 3rd-party private network. Not part of the Microsoft or Carrier network. Can be connected to Microsoft data centre regions using VPN or ExpressRoute
- A proximity placement group is a logical grouping used to make sure that Azure compute resources are physically located close to each other
    - Useful for workloads where low latency is a requirement
    - Do not provide high availability
    - Need to be created before the resource is added to it: <code>az ppg create</code>
  
Azure Resource Manager

- Azure Resource Manager is the deployment and management service for Azure
- Resource Manager recieves requests from Azure tools (Portal, PowerShell, CLI), APIs and SDKs
- Resource Manager authenticates and authorises the request, and then sends the request to the Azure service which takes the requested action
- ARM templates are JSON files that allow you to manage resources declaratively
- ARM templates allow you to deploy, manage and monitor resources as a group. Templates can be re-deployed through the development lifecycle
- ARM templates can define dependancies between resources and apply access control through RBAC
  
Subscriptions

- A subscription is required to use Azure
- Subscriptions are linked to an account. An account may have one or more subscriptions
- A subscription can be used to define either a billing boundary or access-control boundary
    - Azure generates billing at the subscription level
    - Access-management policies are assigned at the subscription level
    
- Subscriptions can be used to separate
    - Environments
    - Departments
    - Billing
    
- Additional subscriptions can be used to overcome subscriptions limits: e.g. 10 ExpressRoute connections per subscription
- Billing Profiles can be used to create invoice sections within the same billing account
  
Management Groups

- Used to manage access, policies and compliance for multiple subscriptions
- Subscriptions within a management group automatically inherit the governance conditions of the parent Management Groups. These conditions can not be overwritten at lower levels in the hierarchy
- Subscriptions within a management group must trust the same Azure AD tenant
- 10,000 Management Groups limit in a single directory
- A Management Group tree supports up to six levels of depth
- Management Groups and Subscriptions can have only one direct parent
- Management Groups and Subscriptions exist in a single hierarchy in each directory
  
Azure Marketplace

- searchable catalogue of services optimised and certified to run on Azure
- includes access to solutions from independant providers and MS partners
  
Azure Free Account

- free access for 12 months
- credit to spend for first 30 days
- 25 always-free products
- requires credit card and MS or Github account
  
Azure Free Student Account

- free access for 12 months
- $100 credit to spend for first 12 monts
- free access to certain software developer tools
  

# Core Services

## Compute
Virtual Machines - IaaS service

- [VM Types]("https://docs.microsoft.com/azure/virtual-machines/sizes")
    - General Purpose (A, B, D family): balanced CPU-to-Memory ratio
        - A Series: for tesing and development
        - B Series: burstable workloads
        - D Series: general-purpose workloads
    - Compute Optimised (F family): high CPU-to-Memory. Best suited to workloads where CPU is more likely to be a bottle-neck. For example, web servers, application servers, network appliances and batch processes
    - Memory Optimised (E, G, M family): high Memory-to-CPU ration. Best suited for memory-intensive workloads. For example, relational databases, in-memory anlytics
    - Storage Optimised (L family): have high disk-throughput and I/O. Best suited for data analytics, data warehousing
    - GPU Optimised (N family): have GPUs. Best suited for compute-intensive, graphics, gaming, visualisations, video-conferencing, video-streaming workloads
    - High Performance (H family): most powerful CPU VMs available providing high-speed network throughput. Suitable for compute and network workloads. For example SAP Hana
- Naming convention: [Family] + [Sub-Family] + [# of vCPUs] + [Constrained vCPUs] + [Additive Features] + [Accelerator Type] + [Version]
- Additive Features are:
    - a: AMD-based processor
    - b: Block Storage performance
    - c: confidential
    - d: VM has local temp disk
    - i: isolated size
    - l: low-memory state
    - m: memory-intensive size
    - s: premium storage capable (although newer sizes without 's' can also support premium storage
    - t: tiny memory size
    - NP: node packing
    - P: ARM CPU
- Examples
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

## Networking
Azure Virtual Network

- Azure VNet is an IaaS resource and enables communication with Azure resources
- A VNet is a software-defined, private network
- VNets can be isolated or connected to public or private networks
- Control traffic into a VNet using a VPN Gateway, an NSG or a firewall
    - A firewall can control access to multiple VNets across multiple subscriptions
    - A VPN can only be applied to a single VNet
    - NSG are applied at the Subnet or Device level
- VNets belong to a resource group and can only be part of one Region
- Traffic entering VNet and region is not billed: only traffic leaving the VNet and Region is billed
- VNets can be segmented into subnets. Azure reserves the first four IP Addresses and the last IP address in each subnet:
    - The first IP address (x.x.x.0) is reserved for the VNets network address
    - The second IP address (x.x.x.1) is the default gateway
    - The third and fourth addresses are reserved for the VNets Azure DNS
    - The last IP address (x.x.x.255) is the broadcast address
- Azure applies default routes to a VNet to allow resources to communicate. These cannot be editted or removed. User-Defined Routes (UDR) can be assigned to override default routes
- UDR is applied with a route table resource combined with an NVA (Network Virtual Appliance) allowing for network segmentation and traffic control
- External VNet routing is either:
    - Hot-Potato routing: packet is passed to the internet as soon as possible
    - Cold-Potato routing: packets stay on Microsoft network as long as possible. This means packet is handed to an Edge Point of Presence to be delivered to its final destination. Cold-Potato routing ensures the packets spends the least time on the internet, and results in fast delivery and lower latency
- Isolation and Segmentation - private IP address space using public or private IP address ranges. Public IP address are not internet routable
- Internet communication - VMs can connect to the internet by default. To connect to the VM from the internet you can assign a Public IP address or put the VM behind a public load balancer
- Communication between Azure resources - some services (VMs, App Services, AKS, VMSS) can communicate via the Virtual Network. Other services (CosmosDB, Service Bus, Key Vault, Storage Accounts, Azure SQL Db) require Service Endpoints
- Communication between VNets requires either VNet Peering or a VPN gateway
- Communication with On-Prem Services
    - Point-to-Site VPN: connects a single device to an Azure VPN using OpenVPN, SSTP and IKEv2 protocols
    - Site-to-Site VPN - link on-prem VPN Gateway to the Azure VPN Gateway to support hybrid connectivity. Supports legacy policy-based (static) connections or newer route-based (dynamic) connections. Supports IKEv1 and v2. Authentication is by pre-shared keys only. Can also be used to connect two Azure VNets to provide encryption where public IP addresses are being used. VNet peering is however the recommended way to connect to Azure VNets. 
    - Azure ExpressRoute - dedicated, private connectivity that doesn't travel over the internet. Traffic is carried over a private, managed network between your resources and Microsofts data centres. Supports built-in redundancy, dynamic routing and BGP
- Route Network Traffic - Azure routes traffic between subnets on any connected Virtual Network, On-Prem networks and the Internet. Route tables can be used to control routing between subnets. BGP routes can be propagated from on-prem routers to Azure VPN gateways, Azure Route Server and ExpressRoute
- Connect Virtual Networks - Vitual Network Peering allows you to connect VNets together. VNet peering ensures that traffic remains on the Microsoft backbone and ensures fast, reliable and low-latency connectivity. Regional VNet peering is used to connect VNets from the same region: Global VNet peering is used to connect VNets from different Regions.
- Address Space - the address space for each VNet needs to be unique in your subscription and for any other network that you wish to connect to
- NAT Gateway - you can configure a subnet to use a static outbound IP address when accessing the internet
- Bastion Host - can be enabled for the VNet
- DDoS Protection - can be enabled for the VNet
- Firewall - can be enabled for the VNet
- NSGs
    - Filter traffic to and from Azure resources within a VNet
    - Uses a deny-by-default policy - all traffic is denied unless explicitl allowed
    - Rules are defined by specifying:
        - Source of traffic
        - Source port of traffic
        - Destination of traffic
        - Destination port of traffic
        - Protocol used
    - Rules are assigned an action: either allow or deny
    - Rules are assigned a priority to determine the order in which they are processed
    - Higher priority rules override the lower priority rules
    - Rules can be defined to control traffic into and out of:
        1. a VNet
        2. Subnets on the VNet
        3. a VM
    - Can be associated with each subnet in the VNet
    - An NSG can be associated with multiple Subnets and NICs, but a Subnet and NIC can only have one NSG associated
    - NSGs can only be associated to resources in the same region and subscription. Firewalls are required to filter traffic across multiple subscriptions
    - Use <code>az network nsg list</code> command to list NSGs
    - Use <code>az network nsg rule list</code> to inspect NSG rules
    
ASGs

- Application Security Groups
- Group VMs based on application structure: databases, logic, web
- ASG can be used in an NSG to allow or deny traffic
- Members of an ASG must be on the same VNet
- Where rules refer to two ASGs, they must both be on the same VNet
- Only 3,000 ASGs allowed per subscription

Route Table - default route tables are created for each subnet in the VNet. Route tables can be added to customise routing

Subnet Delegation - designate a subnet to be used for a dedicated service

Azure Load Balancer

Azure Application Gateway

- Load balancer for service endpoints
- Provides a Web Application Firewall (<strong>WAF</strong>) to protect applications from inbound threats
- Application Gateway WAF is deployed to a region only. The Application Delivery Network (ADN) that comes with Azure Front Door provides Layer 7 filter at a global level
  
Azure VPN Gateway

- Connect two trusted private networks using an encrypted tunnel over a public, untrusted network
- Deployed in a dedicated subnet of the VNet
- A VNet can only have one VPN gateway and the VPN Gateway can only be associated with one VNet
- VNet address spaces can not overlap with on-prem network address spaces
- Use a pre-shared key for authentication
- Internet Key Exchange (IKE) is used to set up a security association (an agreement of the encryption) between two endpoints
- IPSec suite used to encrypt/decrypt data based on the security association
- VPN is either:
    1. Policy-based
        - Specifies static IP address for encryption through the tunnel
        - supports IKEv1 only
        - relies on static route definitions
        - primarily used where legacy on-prem VPN devices require them
    2. Route-based
        - IPSec tunnel modelled as a network interface or vitual tunnel interface
        - IP routing decides whether to use IPSec tunnel
        - supports IKEv2
        - Can use dynamic routing protocols
        - more resilient to creation of new subnets
- Deploy VPN Gateway
    - GatewaySubnet - add a subnet called <strong>GatewaySubnet</strong> to the VNet with a netmask of at least /27
    - Public IP address - Basic SKU Public IP needed as target for on-prem VPN device
    - Local Network Gateway: defines the on-prem network address spaces that will be connecting to Azure and the on-prem VPN used. The on-prem VPN appliance is defined either by IP Address or FQDN
    - Virtual Network Gateway - routes traffic between the VNet and on-prem network
    - Connection: creates a logical connection between the Local Network Gateway and the VPN Gateway
- Availability - VPN gateways are, by default, configured in an active/standby configuration. With BGP routing, active/active configurations are available, allowing connections to multiple on-prem VPN devices. VPN gateways can also be used for ExpressRoute failover or in Zone-redundant configuration.

Azure DNS

Azure Content Delivery Network
- Includes Web Application Firewall (WAF) service
  
Azure DDoS Protection

- Discards DDoS traffic at the Azure network edge  
- Protects against over-consumption of resources
- A singel DDoS protection plan is enabled at the tenant level and used across multiple subscriptions
- Default Basic plan is free
- Standard SKU adds protection from:
    - volumetric attacks
    - protocol attacks
    - resource layer (application) attacks

Azure Traffic Manager
- a DNS-Based traffic load balancer
- lets you distribute traffic across global Azure regions
  
Azure ExpressRoute
- extends on-prem network into Azure via a connectivity provider
- connections don't go over the public internet
- connection is private but not encrypted
- Layer 3 connectivity (IP Address)
- Dynamic Routing
- Connection methods
    - Colocation at a cloud exchange
    - Point-to-point connections between on-prem and Azure
    - Any-to-any connections - integrates Azure to your WAN connection
    - Directly from ExpressRoute sites
      
  
Azure Network Watcher

Azure Firewall

- Cloud-based network security service that protects your Azure VNets across multiple regions and subscriptions
- Stateful firewall - analyses connection not just individual packets
- Offers high availability and unlimited scalability
- Uses a static, public IP for your virtual network resources
- Integrated with Azure Monitor to enable logging and analytics
- User-Defined Routing used to control traffic
- Inbound and Outbound Filtering rules
- Application rules - allowed FQDNs that can be accessed from a subnet
- Inbound Destination Network Address Translation (DNAT)
- Outbound Source Network Address Translation (SNAT)
- Normally Deployed to a central VNet to control general network access
- Premium SKU adds:
    - TLS inspection
    - Intrusion Detection Systems
    - Intrusion Prevention Systems
    - URL Filtering
    - Web Categories
    
- Third-Party NVAs are available via the Azure Marketplace as an alternative to Azure Firewall

Azure Virtual WAN

## Storage
Azure storage provides a unique namespace for your data, accessible over HTTP or HTTPS. 
Namespaces are formatted: <code>https://$STORAGE_ACCOUNT_NAME.$SERVICE_NAME.core.windows.net</code>. For example:

- Blobs: https://$STORAGE_ACCOUNT_NAME.blob.core.windows.net
- Files: https://$STORAGE_ACCOUNT_NAME.file.core.windows.net
- Queues: https://$STORAGE_ACCOUNT_NAME.queue.core.windows.net
- Table: https://$STORAGE_ACCOUNT_NAME.table.core.windows.net
- Data Lake: https://$STORAGE_ACCOUNT_NAME.dfs.core.windows.net

The following limits apply to Storage Accounts:

- Number of storage accounts per region per subscription: <strong>250</strong>
- Maximum storage account capacity: <strong>5 PiB</strong>
- No limit to number of containers, blobs, files, tables, etc
- Maximum request rate per storage account: <strong>20,000 requests per second</strong>
- Maximum ingress per storage account (US/Europe): <strong>10 Gbps</strong>
- Maximum ingress per storage account (outside US/Europe): <strong>5 Gbps</strong>
- Maximum egress: <strong>50 Gbps</strong>

Ingress rates may be increased by raising a support request with Microsoft
Data is durable, secure, scalable, managed, accessible.
Azure storage encryption is enabled by default and cannot be disabled
A Storage Account needs to be created to store data objects. The Storage Account can be used to:

- View Storage Objects
- Configure Network and Security settings
- Perform Data Management and Monitoring
- Diagnose Problems
- Move and Migrate data
- Apply Governance controls
- View activity logs
- Control access

There are two types of Storage Account: Standard and Premium. Premium is intended for low-latency data access.

Azure Blob Storage: Unstructured data - can hold any type of data. Blobs are stored in Containers, which act like folders to organise the blob storage.

- Ideal for:
    - serving images/files to a browser
    - storing files for distributed access
    - streaming video/audio
    - storing backups
    - storing data for analysis
    - storing up to 8TB of data for VMs
- Blob Storage offers three access tiers to make storage more cost effective:
    - Hot - optimised for data that is accessed frequently: highest storage costs, lowest access costs
    - Cool - accessed infrequently and stored for at least 30 days: higher access costs and lower storage costs than hot tier
    - Archive - rarely accessed, stored for at least 180 days: lowest storage cost and highest access cost. Data is stored off-line and will take several hours before the first byte is avaiable
- Access tier can be set at blob level, during or after upload. Lifecycle Management on the Storage Account allows you to configure rules to transition blobs between tiers.
- Blob storage comes in three forms:
    - Page Blob: used to hold random access (unordered) files, such as VM disks (managed disks are stored in Microsoft storage accounts)
    - Block Blob: used to hold objects that are ordered, consecutive and non-random, such as backups
    - Append Blob: used for information that is added in consecutive order, such as log files

  
Azure File Storage: provides cloud-hosted file shares. Files can be accessed using SMB or NFS. Can be accessed from on-prem or VMs in Azure. Files are encrypted at rest, and SMB ensures that it is encrypted in transit. Files can be access using a URL or SAS URL token. 

- Ideal for:
    - Applications that use file shares
    - Sharing files between VMs or Users
  
Azure Queue Storage

Azure Table Storage

Data Box

- Data Box is a physical storage device with a maximum capacity of 80TB
- Use device to copy data to or from Azure Storage
- Data Box can be used for: 
    - One-time migration - for large (>40TB) datasets
    - Initial bulk transfer - followed by incremental loads over the network or with Data Box Gateway
    - Periodic Uploads - where network connectivity is limited

Data Box Gateway is a service that allows you to securely transfer large amounts of data to and from Data Box

- Data Box Gateway consists of: 
    - a virtual device hosted on-prem
    - a gateway resource on Azure
    - a local Web UI for management and support
- Files can be copied to the gateway using SMB or NFS
- Files written to the Gateway are transferred to Azure Storage
- Data Box Gateway can be used to:
    - Cloud Archival - copy 100s TBs of data to Azure Storage
    - Continuous Data Ingestion
    - Incremental Transfers following a Bulk Transfer - incremental loads after an initial Data Box load

## Mobile
Create backend services for iOS, Android and Windows apps

## Databases
Azure Cosmos DB

- globally distributed and horizontally scalable table storage
- 99.99% SLA for reads and writes
- row-level access control and data encryption in transit and at rest
- multi-model support
- supports schema-less data
- stores data in Atom-Record-Sequence (ARS)
- data is then projected as an API. APIs include:
    - SQL
    - MongoDB
    - Cassandra
    - Tables
    - Gremlin
  
Azure SQL Database

- Fully-managed PaaS offering - including upgrades, patching, backups and monitoring
- Based on latest version of Microsoft SQL Server database engine
- 99.99% availability
- Single database or elastic pools available
- Priced by vCore and Data Transaction Units (DTU)
- Supports basic, general-purpose, business-critical and hyperscale tiers
- Scale-up and scale-down between tiers is available
- No horizontal scaling
- support for relational and non-relational data including: graphs, JSON, spatial and XML data
- Use Azure Database Migration Service to migrate on-prem SQL Servers to Azure SQL Database
- Use the Microsoft Database Migration Assistant to generate a report to guide the migration process
- Only provides a single collation option

Azure Database for MySQL

- Based on MySQL CE, versions 5.6, 5.7 and 8.0
- 99.99% availability
- Built-in security, fault-tolerance and data protection
- Automatic backups: Point-in-time restore up to 35 days previous
- Use Azure Database Migration Service to migrate on-prem MySQL Servers to Azure Database for MySQL
- Supports Single-Server, Flexible-Server and Hyperscale (Citus)
- Hyperscale (Citus) provides horizontal scaling
  
Azure Database for PostgreSQL

- Based on the open-source PostgreSQL database engine
- Built-in high-availability
- Various service tiers available
- Automatic backups: Point-in-time restore up to 35 days previous
- Encryption at rest and in-transit
- Dynamic scalability
- Available in Single Server or Hyperscale (Citus) editions
- Hyperscale offers horizontal scaling across multiple machines using sharding

SQL Server Managed Instance

- PaaS offering, similar to Azure SQL Database but with support for more features: 
    - see <a href="https://docs.microsoft.com/en-us/azure/azure-sql/database/features-comparison?view=azuresql" target="#">feature comparison</a>
    - additional collations
    - SQL Server Agent available
    - Supports more functions, procedures, statements
    - Time Zone choice
- Good choice for lift-and-shift migrations
- Full SQL Server Enterprise Edition database engine
- Supports SQL Authentication or AAD Authentication
- Suports single-instance or instance pools

Azure Database Migration Service - manage migrations to cloud

Azure Cache for Redis

Azure Database for MariaDB

## Web

Azure App Service - PaaS offering to deploy front-end web applications

- App Service Plan determines how much resource is allocated
- App Service Plan defines the number of VMs your App Service will use, the type and size of the VMs. You can also define the region, OS and application stack to use
- An App Service Plan can be used to host multiple App Services
- App Service Plans are available in a number of tiers:
    - Free and Shared: intended for testing and development. No autoscaling, hybrid connections or VNet connections
    - Basic: intended for low-traffic usage. No autoscale or traffic management features. Built-in load balancing across instances. Custom domains and hybrid connectivity are supported
    - Standard: intended for production workloads. Supports autoscaling, custom domains, hybrid and VNet connectivity
    - Premium: intended for higher scale and performance workloads. Support for all features in Standard plus private endpoints
    - Isolated: mission critical workloads running in a private, dedicated environment
- App Service Plans are billed, even when no apps are running. This is because the VMs are still running
- Built-in load-balancer and traffic manager
- Supports Web Apps, API Apps, WebJobs and Mobile Apps
- Scale-out and Scale-Up are available for Basic and above service plans
  
Azure Notification Hubs - push notifications to any platform from any back end

Azure API Managment - publish APIs

Azure Cognitive Search - search as a service

Web Apps feature of Azure App Service

Azure SignalR Service

## IoT

Devices equipped with sensors can send data over the internet to an Azure endpoint.
Message data can then be analysed. Additionally software updates can be sent to devices from Azure.

- IoT Hub - build your own IoT solutions. IoT PaaS solution. Messaging Hub for secure, bi-directional communications for devices, to monitor, collect data, update software and control the device.
- IoT Central - consume IoT solutions. IoT SaaS solution. Builds on IoT Hub by providing a dashboard to connect, monitor and manage IoT devices. Starter templates are available for various common scenarios. Device developers need to create code to run on the devices that match the chosen template.
- Azure Sphere is an end-to-end IoT solution covering everything from the hardware used to message handling. There are three key components:
    - Azure Sphere micro-controller unit, responsible for processing the signals from the sensors
    - Customised Linux OS used to run the vendors software
    - Azure Sphere Security Service (AS3) for authenticating to Azure and establishing a secure connection
- Choose Azure Sphere if device security is critical

## Big Data

Traditional ETL is not appropriate to handle the volume, velocity, complexity and format of Big Data. ELT is preferred. Typical components of a Big Data architecture are:

- Data Sources: databases, log files, file stores, IoT, social media
- Data Storage: Data Lakes and Blob Storage
- Real-Time Message ingestion: Azure Event Hubs, Azure IoT Hubs, Kafka
- Batch Processing: HDInsight and Databricks
- Stream Processing: Azure Stream Analytics and HDInsight
- Analytical Data Store: Synapse Analytics, SQL Data Warehouse, HDInsight
- Analysis and Reporting: Synpase Analytics, Azure Analysis Services, PowerBi, Excel
- Orchestration: Data Factory

Azure Synapse Analytics

- PaaS solution
- big-data analytics service
- fully managed data warehouse
- rebranded Azure SQL Data Warehouse
- built on the Massively Parallel Processing (MPP) technology
- supports Apache Spark as a fully-managed service
- Azure Synapse Studio provides a Web-Frontend to manage and interact with data
- Typically used to prepare data
  
Azure HDInsight

- open-source, enterprise-level, fully-managed, big-data analytics service
- Uses Hadoop framework for distributed processing and analysing big datasets through clusters
- also supports a variety of other cluster types:
    - <a href="https://docs.microsoft.com/en-us/azure/hdinsight/spark/apache-spark-overview" target="#">Apache Spark</a>
    - <a href="https://docs.microsoft.com/en-us/azure/hdinsight/hadoop/apache-hadoop-introduction" target="#">Apache Hadoop</a>
    - <a href="https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-introduction" target="#">Apache Kafka</a>
    - <a href="https://docs.microsoft.com/en-us/azure/hdinsight/hbase/apache-hbase-overview" target="#">Apache HBase</a>
    - <a href="https://docs.microsoft.com/en-us/azure/hdinsight/storm/apache-storm-overview" target="#">Apache Storm</a>
    - Machine Learning Services
- Supports ETL, data warehousing, machine learning and IoT
- Typically used to prepare data
  
Azure Databricks

- Apache Spark PaaS service
- Databricks runs on fully managed Spark clusters
- Databricks workspace is a web-based frontend to manage and interact with data
- data analytics and AI solution
- Typically used to prepare and train datasets
- Apache Spark environment supports:
    - Python, including scikit-learn, TensorFlow, PyTorch
    - Scala
    - R
    - Java
    - SQL

Azure Data Lake Analytics

- on-demand analytics job-service
  
## AI

- Artificial Intelligence
    - The ability of computers to imitate intelligent human behaviour
- Machine Learning
    - The ability for computers to improve at tasks through experience (learning). This is known as training and will produce a model that can be deployed and is said to be 'trained'. ML is based on algorithms whose output improves over time as they are fed more data (training datasets).
- Deep Learning
    - The ability for a computer to train itself to perform a task. Based on multi-layered neural networks and is a subset of ML

Azure Machine Learning 

- A platform for making predictions. With Azure Machine Learning you can:
    - Obtain data
    - Define how to handle missing or bad data
    - Split data into training and test datasets
    - Deliver data to the training process
    - Train predictive models
    - Create pipelines to schedule compute-intensive experiments to score the predictive models
    - Deploy the best-performing algorithm as an API to an endpoint to be consumed in real-time by other applications
- Azure Machine Learning is recommended when you need complete control over the design and training of your algorithm using your own data
  
Azure ML Studio - collaborative visual workspace

Cognitive Services

- Pre-built machine learing models, including:
    - Vision - image processing
    - Speech - text-to-speech and speech-to-text
    - Translator
    - Personaliser - predict user behaviour, offer personalised recommendations
    - Decision support
    - Natural Language Processing
- Accessed by developers via the Cognitive Services API
  
Azure Bot Service and Bot Framework - create virtual agents that understand and reply to questions. Used for automating repetitive tasks. May rely on other Cognitive services. Web App Bot templates can be used to get started. 

## DevOps

Azure DevOps

- A suite of services for the software development lifecylcle. Services include:
    - Azure Repos - source code repositories
    - Azure Boards - Agile project managment suite
    - Azure Pipelines - a CI/CD pipeline automation tool
    - Azure Artifacts - a repository for artifacts that can be fed into CI/CD and testing pipelines
    - Azure Test Plans - automated test tool
- DevOps offers a more granular permissions model than GitHub
- DevOps provides a more sophisticated level of project mananagement and reporting tools

GitHub and GitHub Actions

- Git is a decentralised source-code managment tool
- GitHub is a hosted version of Git. GitHub offers additional functionality:
    - source code review via comments and questions
    - project managment tools
    - issue tracking
    - CI/CD automation
    - Wiki
    - Run from on the Cloud or on-prem
- GitHub supports workflow automation
- GitHub offers similar tools to DevOps, and is more trusted in the open-source community. DevOps is aimed at enterprise-development, with stronger support for project managment and planning
- GitHub has limited permissions model over each repository
  
Azure DevTest Labs

- Create on-demand environments to test/deploy applications from deployment pipeline
- Can be used to deploy any Azure Resource that supports ARM template deployment
- Automates provisioning and deprovisioning of environments

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

# Security

## Threat Modeling
The Threat Priority Model can be used to identify your threat priorities. It starts by identifying the Impact of a threat occuring on a resource, then identifies the probability
of this happening by defining the vulnerability and identifying any existing counter-measures (mitigations). The Impact and Probability together define the Threat Priority
## Zero Trust
The attack chain (or kill chain) begins with a compromised account that is then used to gain elevated 
priviledges to mount an attack. Zero Trust uses the principle of <i>Never Trust: Always Verify</i>
## Defence In Depth

A defence-in-depth strategy addresses security at the following layers:

    1. Physical Security - protecting access to buildings and hardware
    2. Identity and Access - authentication and authorisation controls
    3. Perimeter - protecting the network perimeter using DDoS protection and firewalls
    4. Network - limiting communication between and to resources
    5. Compute - secure access to VMs and endpoints
    6. Application - securing applications, remediating vulnerabilities
    7. Data - securing access to data



- Microsoft Defender for Cloud (previously Azure Security Centre and Azure Defender)
    - Monitor security posture for Azure and on-prem resources. Security Posture (<strong>CIA</strong>):
        - Confidentiality - sensitive data must be kept protected and accessed only by those who should have access through the principle of least priviledge
        - Integrity - confidence that data has not been altered or tampered with
        - Availability - data and systems should be available to those that need them
    - Automatically apply security settings to new resources
    - Provides recommendations
    - Continuously monitor and assess vulnerabilities
    - Block malware with machine learning
    - Define rules for allowed applications (adaptive application controls)
    - Threat detection
    - Just-in-time access control for network ports
    - Secure score reflects compliance to assigned governance controls
    - The Regulatory Compliance dashboard provides overall compliance score and the number of failing assessments
    - Adaptive network hardening monitors network activity compared to NSGs and makes recommendations
    - File integrity monitoring - monitor important files
    - Use Workflow Automation (Logic Apps) to respond to security alerts
  
- Microsoft Sentinel (previously Azure Sentinel)
    - Microsoft Managed (SaaS), cloud-based Security Information and Event Management (SIEM) and Security Orchestration, Automation and Response (SOAR) system
    - Aggregates security data from many different sources, using open-standard logging format
    - Threat detection uses built-in rules (AI) provided as templates or custom rules
    - Threat response can be automated with Azure Monitor Workbooks, to set alerts or send emails. Email will include link to Block or Ignore threat
    - Utilises Microsoft's analytics and threat intelligence
    - Several methods exist for connecting security data sources to Sentinel, including native support for M365, AAD, Syslog, CEF, REST
    - Data is stored in Log Analytics
  
- Azure Key Vault
    - Secure, centralised storage for application secrets
    - Monitor and control access to secrets
    - Simplify management and renewal of certificates
    - Integrates with other Azure services - storage accounts, container registries, event hubs, etc
    - Available in Standard and Premium SKUs
    - Premium adds support for HSM-protected Keys
    - Resources must be in the same region and subscription to access key vault secrets
  
- Azure Dedicated Host
    - VMs hosted on physical servers that are not shared with other customers
    - Host groups provide for high availability
    - Charge is per dedicated host - not per VM running on the host
  
Further Reading:

- <a href="https://docs.microsoft.com/en-us/learn/modules/resolve-threats-with-azure-security-center/" target="#">Resolve Security Threats</a>
- <a href="https://docs.microsoft.com/en-us/learn/modules/design-monitoring-strategy-on-azure/" target="#">Security Monitoring Plan</a>
- <a href="https://docs.microsoft.com/en-us/learn/modules/manage-secrets-with-azure-key-vault/" target="#">Manage App Secrets</a>
- <a href="https://docs.microsoft.com/en-us/learn/modules/configure-and-manage-azure-key-vault" target="#">Manage Key Vault Secrets</a>
- <a href="https://docs.microsoft.com/en-us/learn/paths/implement-resource-mgmt-security/" target="#">Implement Resource Management Security</a>
- <a href="https://docs.microsoft.com/en-us/learn/paths/secure-your-cloud-data/" target="#">Secure Cloud Data</a>
- <a href="https://docs.microsoft.com/en-us/learn/paths/az-400-develop-security-compliance-plan/" target="#">Develop a Security and Compliance Plan</a>
- <a href="https://docs.microsoft.com/en-us/learn/paths/manage-security-operations/" target="#">Manage Security Operations in Azure</a>

# Identity and Governance

## Azure AD
With Cloud Computing and BYOD, identity is now the primary security boundary. Identity is 
establish by the exchange of credentials to establish the identity of an agent 
(authentication). Authorisation is about establishing the levels of access to be granted
to an authenticated agent.

- Azure AD is a cloud-based identity and access management service (SaaS)
- AAD can be used to grant access to Azure Portal, Microsoft 365 and thousands of other SaaS applications
- A Tenant is a representation of an organisation. Each Microsoft 365, Office 365, Azure and Dynamics CRM tenant is automatically an Azure tenant
- An instance of Azure AD is created for each tenant
- Additional tenants can be created from the AAD Overview page
- Custom domain names can be added in addtion to the default <code>$ORG_NAME.onmicrosoft.com</code>. The custom domain should be registered with a domain name registrar
- Azure AD is available with different licences:
    - AAD Free
        - user and group management
        - on-prem directory synchronisation
        - basic reports
        - self-service password change for cloud users
        - SSO across Azure, M365 and many popular SaaS apps
        - Included with any Microsoft Online business service
    - AAD Premium P1
        - paid licence built on top of existing free directory
        - create custom RBAC roles
        - grant hybrid users access to both on-prem and cloud resources
        - dynamic groups
        - self-service group management
        - Microsoft Identity Manager
        - Cloud write-back capabilities allows self-service password reset for your on-prem users
    - AAD Premium P2
        - paid licence built on top of AAD P1
        - AAD Identity Protection:
            - risk-based Conditional Access
            - Privileged Identity Management
            - Just-In-Time access
    - "Pay as you go" feature licences. Additional features such as Business-to-Customer (B2C) provide IAM for customer-facing apps
  
- The AAD Architecture consists of 
    - A primary replica that receives all write operations for the partition it belongs to. The write operation is immeadiately replicated to a secondary replica in a different data centre before returning success to the caller
    - Secondary replicas are used to service all directory reads. Secondary replicas are distributed across different geographies. Data is replicated asynchronously. Directory reads are serviced from data centres that are closest to the customer
- Write scalability is achieved by partitioning the data. Read scalability is achieved by having multiple secondary replicas
- High availability is guaranteed by the services ability to shift traffic across multiple geographically distributed data centres
- If a failure occurs on a primary replica, writes are immeadiately shifted to another replica. This can result in a 1-2 minute loss of write availability.
- Azure AD holds objects known as security principles. These can be one of the following types:
    - Users: can be a member of the tenants Azure AD or a guest. Guests can be B2B (business-to-business) or B2C (third-party identity provider)
    - Application Service Principal: represents the identity of a service or application in Azure
    - Managed Identity Service Principal: used by services or applications in place of a user identity. Can be either system-assigned or user-assigned
    - Device: a device identity
  
- Azure AD provides:
    - Authentication: includes self-service password resets, MFA, smart-lockout, password blacklist
    - SSO
    - Application Management
    - Device Management
  
- Conditional access allows or denies access to resources based on <i>signals</i>: identity, location, service requested, device, device security status. Conditional Access requires an Azure AD Premium P1 or P2 licence. Signals could result in the logon attempt requiring MFA
  
- Azure AD Connect
    - Free download tool used to synchronise user identities between on-prem Active Directory and Azure AD
  

## Governance
Governance is the process of establishing and enforcing rules and policies

- Azure RBAC
    - Role-Based Access Control
    - Pre-defined roles defining access-levels to resources
    - Uses an allow model: roles define what is <strong>allowed</strong>
    - Roles assigned to principals within a scope:
        - management group
        - subscription
        - resource group
        - resource
    - Enforced against any action on a resource that is initiated through Resource Manager
    - Does not apply at the application or data levels
    - It is good practice to assign users to groups and then assign RBAC roles to the group, rather than assign roles directly to user accounts
- Resource Locks
    - Prevents resources being accidentally deleted or modified
    - Accessible from the 'Locks' pane in the Azure Portal
    - Resource locks are inherited by child objects if these exist
    - Two lock-types available
        - CanNotDelete - prevents deletion
        - Read - prevents deletion or modification
    - Read locks can be used on resource groups to prevent additional resources being deployed
- Resource Tags
    - Provide Metadata about a resource
    - Up to 15 tags allowed per resource
    - Tags can be used for:
        - Resource Management
        - Billing and Cost Management
        - Operations Management (SLAs)
        - Security level
        - Governance and Compliance
        - Visualise complex workloads
    - Azure Policy can be used to force resources to inherit tags from their resource group
- Azure Policy
    - Used to define policies that control or audit your resources
    - Groups of policies are called <strong>initiatives</strong>
    - Policy definitions are assigned to a scope:
        - management group
        - subscription
        - resource group
    - Policy assignments are inherited by all child resources in the assignment scope
    - Initiatives can be defined in Azure Portal or via command-line tools
    - New policies can be added to an initiative, and these will be applied without the need to re-assign the initiative
- Azure Blueprints
    - Azure Blueprints allow for the definition of controls at the organisational level
    - Each component in a blueprint is termed an <strong>artifact</strong>
    - Blueprints ochestrate deployments and can include the following types of artifacts:
        - Role assignments
        - Policy assignments
        - ARM templates
        - Resource groups
    - Artifacts can have zero or more parameters
    - Parameter values can be set in the definition or during assignment
    - Several Blueprints exist for ISO 27001
    - Blueprints are backed by the globally available Cosmos DB
    - Unlike ARM templates, Blueprint definitions exist natively in Azure. The relationship between the definition and the assignment is preserved
    - When defining a Blueprint, you will also specify the location to save it to: either a Management Group or Subscription
    - Blueprint definitions begin in draft mode. Before they can be assigned they need to be published
    - Publishing involves assigning a Version string (up to 20 characters)
    - Each version of a single Blueprint can be assigned and different versions of the same Blueprint can be assigned to a subscription
    - When a published Blueprint is altered the original version is retained, along with unpublished changes. When the changes are published a new Version is assigned to the Blueprint definition
    - Blueprints can be assigned at the Management Group level but this needs to be done using the <a href="https://docs.microsoft.com/en-us/rest/api/blueprints/assignments/create-or-update?tabs=HTTP" target="#">REST API</a>
    - Blueprint creation requires the following roles:
        - Microsoft.Blueprint/blueprints/write - Create a blueprint definition
        - Microsoft.Blueprint/blueprints/artifacts/write - Create artifacts on a blueprint definition
        - Microsoft.Blueprint/blueprints/versions/write - Publish a blueprint
    - Blueprint deletion requires the following roles:
        - Microsoft.Blueprint/blueprints/delete
        - Microsoft.Blueprint/blueprints/artifacts/delete
        - Microsoft.Blueprint/blueprints/versions/delete
    - Blueprint assignment requires the following roles:
        - Microsoft.Blueprint/blueprintAssignments/write - Assign a blueprint
        - Microsoft.Blueprint/blueprintAssignments/delete - Unassign a blueprint
    - Built-in Roles also have Blueprint permissions:
        - Owner - all Blueprint related permissions
        - Contributor - can create and delete Blueprint defintions. No assignment permissions
        - Blueprint Contributor - can manage Blueprint definitions. No assignment permissions
        - Blueprint Operator - can assign published blueprints using a user-assigned managed identity. No permissions to create new Blueprint definitions
    - Rules for Updating Assignments:
        - Role Assignments - if the role or assignee has changed, an new role assignment is created. Previously deployed roles are retained
        - Policy Assignments - assignment is updated if the parameters are changed. If the definition is changed a new policy assigment is created and the previous assignment is retained. If the policy assignment is removed from the blueprint, the previous assignment is retained
        - ARM Templates are deployed using a PUT request. Different resources will respond differently (update, replace) to this request
    - Resource Locking in Blueprint Assignments
        - Resource locks applied by a Blueprint assignment are only applied to non-extension resources deployed by the blueprint. Existing resources do not have locks added to them
        - There are three locking modes available:
            - Don't Lock
            - Read Only
            - Do Not Delete
        - Locking mode is applied during artifact deployment resulting from the assignment. A different locking mode can be set by updating the assignment or removed by deleting the assignment. The locking mode cannot be changed outside of Azure Blueprints
        - To prevent a subscription owner from changing the locking mode of a resource deployed during BP assignment, the assigment can be made at the Management Group level. Then only Owners of the management group can change the assignment. Blueprints assigned to a management group must still specify the subscription that the assignment targets and so is still applied at the subscription level
        - The locking mode will result in resources having one of four states depending on where the locking mode is assigned:
            - Not Locked - where 'Don't Lock' mode is selected in BP assignment
            - Cannot Edit/Delete - where 'Read Only' mode is set on the resource group. The resource group is Read Only, but Not Locked resources can be added, changed or deleted from the resource group
            - Read Only - where 'Read Only' mode is set on a resource deployed during the assignment
            - Cannot Delete - where 'Do Not Delete' is set on a resource deployed during the assignment. 'Not Locked' resources can be added, moved, changed or deleted from a resource group in 'Cannot Delete' state
        - Read Only and Delete locks are implemented by Blueprints as RBAC 'deny' assignments. The deny assigments are added by the managed identity of the BP assignment, and can only be removed from the artifact resources by the same managed identity. Therefore the locks can only be removed from within Blueprints. During the assignment, specific principals can be excluded from the 'deny' assignments. Similarly, specific actions (read, write, etc) on specified resources can be excluded from the deny assignment
    - <a href="https://docs.microsoft.com/en-gb/azure/governance/blueprints/samples/caf-foundation/" target="#">CAF Foundation</a>
        - Deploys an Azure Key Vault to host secrets for VMs deployed in the shared services environment
        - Log Analytics is deployed to ensure all actions and services log to a central location
        - Defender Standard deployed
        - Policies
            - CostCenter Tag applied to Resource Groups
            - Append CostCenter Tag to Resources in Resource Groups
            - Allowed Region for Resources and Resource Groups
            - Allowed Storage Account and VM SKUs
            - Require Network Watcher to be Deployed
            - Require secure transfer encryption on Storage Accounts
            - Deny Resource Types
        - Policy Initiatives
            - Enable Monitoring in Microsoft Defender for Cloud (100+ policy definitions)
- Cloud Adoption Framework
    - Consists of tools, documentation and proven practice to drive success during adoption of cloud computing on Azure
    - The following steps are included:
        - Define Your Strategy
            - Define and Document your Motivations - meet stakeholders and leadership
            - Document Business Outcomes - document your goals
            - Evaluate Financial Considerations - identify the return expected from the investment
            - Understand Technical Considerations
        - Make a Plan
            - Digital Estate - inventory of existing assets
            - Initial Organisational Alignment - involve the right people
            - Skills Readiness Plan
            - Cloud Adoption Plan - shared cloud adoption goal
        - Ready your Organisation
            - Azure Setup Guide
            - Azure Landing Zone - build subscriptions to support each major area of your business
            - Expand the Landing Zone
            - Best Practices
        - Adopt the Cloud
            - Migrate
                - Migrate your First Workload
                - Migration Scenarios
                - Best Practices
                - Process Improvements
            - Innovate
                - Business Value Consensus - ensure innovations add value
                - Azure Innovation Guide - build an MVP (Minimum Viable Product)
                - Best Practices
                - Feedback Loops
        - Govern and Manage your Cloud Environments
            - Govern
                - Methodology - consider end-state solution 
                - Benchmark
                - Initial Governance Foundation - MVP
                - Improve the Initial Governance Foundation - iteratively add governance controls
            - Manage
                - Establish a Managment Baseline - minimum commitment to operations management
                - Define Business Commitments - document supported workloads and required management investments for each
                - Expand the Management Baseline
                - Advanced Operations and Design Principles
- Create a Subscription Governance Strategy
    - Teams often start their governance strategy at the subscription level
    - Three main aspects to consider:
        - Billing - take account of billing requirements when planning subscriptions. Use Tags if needed to identify billing department
        - Access Control - isolation of environments based on subscriptions
        - Subscription Limits - subscriptions have resource limitations: e.g. number of ExpressRoute connections
        - Quotas for resources in resource groups are per region rather than per subscription
- Microsoft Trusted Cloud Principles
    - The shared responsibility model means that depending on the service, some responsibilities will transfer to the service provider
    - Microsofts Trusted Cloud Principles means that Microsoft will provides contractual agreements to ensure:
        - Data residency
        - Data security
        - Data privacy
        - Data compliance
    - The <a href="https://www.microsoft.com/trust-center" target="#">Microsoft Trust Center</a> can be used to access detailed statements from Microsoft on their security, privacy and compliance standards
        - The <a href="https://privacy.microsoft.com/privacystatement" target="#">privacy statement</a> details how each Microsoft service interacts with your data
        - The <a href="https://www.microsoft.com/licensing/terms" target="#">Product Terms</a> site lists the terms and conditions of Microsoft product licences. The Data Protection Addendum lists the data processing conditions associated with these licences
        - Legal and Regulatory compliance documentation can be found at https://docs.microsoft.com/azure/compliance

# Cost Management and SLAs

## Cost Management
- <a href="https://azure.microsoft.com/pricing/tco/calculator" target="#">TCO Calculator</a>
    - Define workloads - on-prem infrastructure
    - Adjust Assumptions - Software Assurance discounts, estimated on-prem costs
    - View the Report
- Purchase Azure Services
    - Enterprise Agreement - commitment to spend a pre-determined amount over a 3 year period
    - Web Direct - monthly billing by credit card or invoice
    - Cloud Solution Provider - bills come from CSP
- Cost Factors
    - Resource Type - each resource has a billing meter and unit cost. The unit cost multiplied by the usage quantity will give the resource cost. The unit of measure multiplied by the usage quantity will give the actual usage.
    - Resource Usage - deallocating versus deleting
    - Subscription Type
    - Marketplace Resources - prices set by vendor
    - Location - resources may have different prices in different regions. Network traffic can be reduced by choosing the closest region to your users
    - Zones - outbound data-transfer pricing based on zones
    - Dev/Test subscriptions - subscription with special pricing for Dev/Test workloads
    - Some resources are free - have no billing meter or unit cost:
        - User account and groups
        - Resource groups
        - Virtual Networks
        - Virtual Network Peering
        - Network Interfaces
        - Network Security groups
        - Availability Sets
- <a href="https://azure.microsoft.com/pricing/calculator/" target="#">Azure Pricing Calculator</a> allows you to calculate the cost of a solution based on the resources and usage
- Azure Advisor - identifies unused or underutilised resources
- Spending Limits
- Azure Reservations - pre-pay for a resource for discounted pricing
- Offers - monitor and switch to latest Azure customer and subscription offers
- Cost Management and Billing - Free service to manage costs and billing
    - view and download invoices
    - view payment methods
    - make payments
    - perform cost analysis
    - set alerts
    - create budgets
    - receive recommendations
- Tags - can be used to organise billing data
- Resize VMs
- Deallocate VMs out of hours
- Delete unused resources
- Prefer PaaS to IaaS services - usually cheaper
- Azure Hybrid Benefit - Windows Server and SQL Server licences purchased with Software Assurance can be re-used on VMs in Azure
- a billing zone is a geographical grouping of Azure Regions used to determine billing based on data transfers. Billing applies to both incoming and outgoing data and varies by billing zone. Data transfers between billing zones and regions are billed

## Service Level Agreements
- Formal agreement between service provider and customer
- Accessible at <a href="https://azure.microsoft.com/support/legal/sla/" target="#">Service Level Agreements</a>
- SLA details focus on uptime, but can also include latency
- Service credits are available if SLAs are not met
- Claims for service credits must be submitted by the end of the month following the outtage
- No SLAs for free services
- SLAs and Downtime:
    - 99.9% == 43 minutes and 49 seconds
    - 99.95% == 21 minutes and 54 seconds
    - 99.99% == 4 minutes and 22 seconds
    - 99.999% == 26 seconds
- Multiply the SLAs for each service in your application to calculate the application SLA value
- Improve SLAs by:
    - Using services that have an SLA or improve the existing SLA
    - Adding redundant resources - duplication across regions to create redundancy and reduce overall downtime
    - Adding availability solutions - availability zones have different schedules for maintenance, and therefore overall downtime decreases

## Preview Features
- Preview features in Azure do not come with a warranty or SLA
- Service Lifecycle stages are:
    - Development - not available to the public
    - Private Preview - available to selected audience
    - Public Preview - available to all customers
    - General Availability - available to all customers
- <a href="https://preview.portal.azure.com/" target="#">Azure Portal Previewi</a>
- <a href="https://azure.microsoft.com/updates" target="#">Azure Updates</a>
