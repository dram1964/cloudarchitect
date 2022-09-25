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
