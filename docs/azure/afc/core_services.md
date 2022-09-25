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

