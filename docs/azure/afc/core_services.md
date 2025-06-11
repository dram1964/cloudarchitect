# Core Services

Moved to [Azure Compute](../compute.md)

## Networking

Moved to [Azure Networking](../networking.md)


## Storage

Moved to [Azure Storage](../storage.md)

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

