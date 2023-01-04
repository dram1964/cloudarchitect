# [Azure Storage](https://learn.microsoft.com/en-us/azure/storage/)

Azure Storage is a cloud storage solution supporting blobs, files, disks, messages, tables and other
types of data. 

A Storage Account needs to be created to store data objects. The Storage Account can be used to:

- View Storage Objects
- Configure Network and Security settings
- Perform Data Management and Monitoring
- Diagnose Problems
- Move and Migrate data
- Apply Governance controls
- View activity logs
- Control access

There are two types of Storage Account: Standard and Premium. Premium is intended for low-latency data access and uses SSDs. You cannot convert between Standard and Premium storage: instead you will 
need to create a new Storage Account and migrate the data from the original Account. 

Azure storage provides a unique namespace for your data, accessible over HTTP or HTTPS. 
Namespaces are formatted: <code>https://$STORAGE_ACCOUNT_NAME.$SERVICE_NAME.core.windows.net</code>. For example:

- Blobs: https://$STORAGE_ACCOUNT_NAME.blob.core.windows.net
- Files: https://$STORAGE_ACCOUNT_NAME.file.core.windows.net
- Queues: https://$STORAGE_ACCOUNT_NAME.queue.core.windows.net
- Table: https://$STORAGE_ACCOUNT_NAME.table.core.windows.net
- Data Lake: https://$STORAGE_ACCOUNT_NAME.dfs.core.windows.net

You can also configure a custom domain to access blob data in a Storage Account. However, 
there is no native support for HTTPS access to blobs with custom domains. 

The following limits apply to Storage Accounts:

- Number of storage accounts per region per subscription: <strong>250</strong>
- Maximum storage account capacity: <strong>5 PiB</strong>
- No limit to number of containers, blobs, files, tables, etc
- Maximum request rate per storage account: <strong>20,000 requests per second</strong>
- Maximum ingress per storage account (US/Europe): <strong>10 Gbps</strong>
- Maximum ingress per storage account (outside US/Europe): <strong>5 Gbps</strong>
- Maximum egress: <strong>50 Gbps</strong>

Ingress rates may be increased by raising a support request with Microsoft. Data is durable, 
secure, scalable, managed, accessible. Azure storage encryption is enabled by default and 
cannot be disabled.

Use the 'Firewalls and virtual networks' settings on your storage accounts to restrict access to 
specifici virtual networks or public IPs. Subnets and Virtual networks must exist in the 
same Azure region or region pair as the storage account. 

Azure Storage access can be granted via AAD, Shared Keys for the Storage Account, SAS tokens or 
read-only anonymous access. SAS tokens granted access for a specified period of time. Stored 
access policies give you the option to revoke permissions without having to regenerate the 
storage account keys. 

## Azure Blob Storage

Blob Storage is optimised for massive amounts of unstructured data such as text or binary data.
Blobs are stored in Containers, which act like folders to organise the blob storage.

- Ideal for:
    - serving images/files to a browser
    - storing files for distributed access (eg. to support an installation)
    - streaming video/audio
    - storing backups
    - storing data for analysis
    - storing up to 8TB of data for VMs
- Access tier can be set at blob level, during or after upload. Lifecycle Management on the Storage Account allows you to configure rules to transition blobs between tiers.
- Blob storage comes in three forms:
    - Page Blob: used to hold random access (unordered) files, such as VM disks (managed disks are stored in Microsoft storage accounts)
    - Block Blob: used to hold objects that are ordered, consecutive and non-random, such as backups
    - Append Blob: used for information that is added in consecutive order, such as log files

Blob Storage setup has the following configuration:

- Container options
    - Name: must be unique within the Storage Account, 3-63 characters (a-z0-9\-) long
    - Access Level: 
        - Private (default): deny anonymous access to the container and blobs
        - Blob: allow anonymous read access to blobs only
        - Container: allow anonymous read access to container and blobs
- Blob Types and Upload options
- Access Tier
    - Hot - optimised for data that is accessed frequently: highest storage costs, lowest access costs
    - Cool - accessed infrequently and stored for at least 30 days: higher access costs and lower storage costs than hot tier. Suitable for data that is accessed infrequently but needs to be available immeadiately, e.g. backup and DR files
    - Archive - rarely accessed, stored for at least 180 days: lowest storage cost and highest access cost. Data is stored off-line and will take several hours before the first byte is avaiable. If data is removed before 180 days, it will incur early deletion charges. 
    - Premium Blob Storage: uses SSDs for I/O intensive workloads
- Lifecycle rules
    - transition blobs to cooler storage and/or deletion
    - define daily rule-based conditions to run at the Storage Account level
    - apply rule-based conditions to containers or sets of blobs
- Replication
    - define a replication policy from one storage account to another
    - both source and destination must be in the Hot or Cool tier - but they don't need to be in the same tier
    - requires blob versioning enabled on source and destination
    - doesn't support replication for blob snapshots
    - replication can be used to reduce access latency, by keeping copies closer to end-users
    - can support compute workloads by allowing the same set of blobs to be accessed in different regions

There are several options for uploading Blob Data:

- Storage Explorer
- AzCopy
- Azure Data Factory
- Azure Storage Data Movement library (.Net)
- blobfuse (linux filesytem support)
- Azure Data Box Disk
- Azure Import/Export

## Azure Disks

Persistent block storage for Azure VMs: data disks have a maximum capacity of around 32 TB

## Azure Data Lake

Azure Data Lake is the Hadoop Distributed Flie System (HDFS) as a service

  
## Azure File Storage

Azure File Storage provides cloud-hosted file shares. Files can be accessed using SMB or NFS from 
on-prem or VMs in Azure. Files are encrypted at rest, and SMB ensures that encryption in transit. 
Files can be access using a URL or SAS URL token. 

- Ideal for:
    - Applications that use file shares
    - Sharing files between VMs or Users
  
## Azure Queue Storage

Azure Queue Storage is used to store and retrieve messages. Queue messages can be up to 64KB and
each queue can hold millions of messages. Queues can be used to store lists of messages 
to be processed asynchronously.

You can use a queue in conjuction with Azure Functions to respond to a user action. For example, 
every time a new file is added to a storage container, you can write a message to the queue. 
An Azure Function can then retrieve the message and run a custom action. 

## Azure Table Storage

Azure Table Storage is now part of Azure Cosmos DB, a fully-managed, NoSQL database service. In 
addition to the Azure Table Storage service, there's an Azure Cosmos DB Table API offering 
throughput-optimised tables, global distribution and automatic secondary indexes. 

Table Storage is ideal for storing structured or relational data. 

## Data Box

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
