Continue <a href="https://docs.microsoft.com/en-us/training/paths/introduction-to-azure-postgres/" target="#">here</a>

# Core Concepts
Azure Database for PostgreSQL is a PaaS service that provides a fully-managed, highly available
and horizontally scalable instance of community PostgreSQL in the cloud. 

Azure Database for PostgreSQL is available in three modes on Azure:

- Single Server: suitable for deployments that do not require low latency or extensive customisations
- Flexible Server: suitable for deplyments requiring zone-resilient high-availability, predictable performance, custom maintenance window, cost optimisation controls and simpler developer experience
- Hyperscale (Citus): delivers scale across multiple machines. Hyperscale can be deployed and managed on-prem, on the edge, or on multi-cloud environments using Azure Arc

Each deployment option offers 99.99% uptime. Data is automatically encrypted and backed-up. Automatic
Threat Detection makes it easier to mitigate threats. 

Administrative overheads are reduced with Azure providing optimised performance, 
automated maintenance and updates for the hardware, OS and database engine and 
automated management and analytics

Hyperscale is based on a PostgreSQL extension called Citus and is recommended for data needs 
exceeding 100GB. Hyperscale allows you to scale-out to multiple servers and the scaling process 
is handled automatically in Azure. The servers can be globally distributed so as to be closer to 
the users connecting to them

Azure places the database engine in a compute container and the data files in Azure Storage. 
Separating the database engine and the data facilitates automated maintenance. The compute container
can be selected during deployment, and altered later allowing for simple scale-up or scale-down. 

Automated backups allow you to restore the data to any point in time within the last 35 days. Servers
can be replicated on up to 5 read-only copies improving the performance of read-only workloads 
such as analytics. Storage can be configured to 'auto-grow', ensuring there is always enough 
storage available.
