# Fundamentals of the Databricks Lakehouse Platform

## What is a Data Lakehouse

In the 1980s, Data Warehouses were developed to organise huge volumes 
of data for analytics and BI. Data Warehouses are organised with 
pre-defined schemas. 

Drawbacks for traditional data warehouses were:

- They failed to provide support for unstructured data 
- they had long processing times
- they could not cope with the increasing velocity and volume of data from digital sources

In the 2000s, Data Lakes were developed to cope with Big Data: handling structured, 
semi-structured and unstructured data generated in huge volumes and high velocity.
Multiple data types could be stored side-by-side in cloud object-stores. Built to handle 
streaming data. Drawbacks of Data Lakes:

- do not support transactional data
    - lack of ACID transaction support
- do not enforce data quality (data swamps)
    - lack of schema enforcement
- slow analysis performance
- governance concerns

The drawbacks led to companies developing Data Warehouses and Data Lakes side-by-side, 
leading to complex, silo-ed environments involving copying data backwards and forwards
between the specialist systems. 

The Data Lakehouse emerged to provide a platform to unify data, analytics and AI workloads. 
Built on a Data Lake, the Data Lakehouse can handle all types of data, becoming a single
source of truth. Data Lakehouses provide:

- Transaction support 
- Schema enforcement and governance
- Data Governance
- BI Support
- Decouples storage from compute
- Open storage formats (e.g. Apache Parquet)
- Support for diverse data types
- Support for diverse workloads
- End-to-end streaming for real-time reports

## What is a Data Lakehouse

Databricks was founded in 2013 by the original founders of Apache Spark, 
Delta Lake and MLflow. The Lakehouse platform was first proposed in 2021, 
based on a paper entitled: 'Lakehouse: A New Generation of Open Platforms
that Unify Data Warehousing and Advanced Analytics'. The Lakehouse 
paradigm specifies:

- Platform for all data processing workloads (AI, ML, SQL Analytics, BI, and streaming)
- One security and governance approach for all data assets
- A reliable data platform for all data types

This is met in Databricks through

- Delta Lake - for data reliability and performance
- Unity Catalog - for governance
- Persona-based use cases

The Databricks Lakehouse Platform provides instant, serverless compute
and is built on open standards and open source and is multicloud. 

## Architecture and Security

### Data Reliability and Performance

Databricks Lakehouse Platform solves the problems of Data Lakes using 
two technologies: 

- Delta Lake
    - file-based open source storage format
    - provides ACID transaction guarantees
    - scalable data and metadata handling (using Spark)
    - audit history and time travel through a transaction log
    - schema enforcement and schema evolution
    - support for deletes, updates and merges
    - unified streaming and batch data processing
    - runs on top of existing Data Lake technologies
    - uses Delta tables based on Apache Parquet
- Photon
    - next-generation query engine
    - supports Spark and SQL APIs but provides improved performance

### Unified Governance and Security

Protecting against data breaches. Databricks Lakehouse Platform addresses
the data and AI governance challenges through:

- Unity Catalog
    - unified governance solution for all data assets
    - uses SQL to define and enforce fine-grained access controls on all data and AI assets on any cloud
    - provides one consistent model to discover, access and share data
    - provides single source of truth for all user identities and data assets
    - access can be controlled by rows or columns and using attribute-based controls
    - provides a detailed audit trail of who has performed what actions against the data
    - provides an interface for data search and discovery
    - provides data lineage and supports impact analysis for data changes
- Delta Sharing
    - open solution to securely share live data to any computing platform
    - data providers retain the ability to track and audit usage
    - share data without copying it
    - privacy-safe data clean-rooms
    - REST protocol to share access to part of a cloud dataset
- Divided Architecture
    - control plane
        - where the applications reside
        - mangaged backend services that Databricks provides. Held in Databricks' own cloud account. 
        - Databricks runs the workspace application and manages notebooks, configuration and clusters
        - Notebook, configuration and log data is encrypted at-rest and in-transit
            - User Identity and Access:
                - Table ACLs
                - IAM instance profiles
                - Securely stored access keys
                - Secrets API
    - data plane
        - where the data and compute resources reside
        - runs inside the business owners own cloud account
        - clusters use the latest, hardened server images
        - clusters are short-lived and often terminated after a job
- Databricks support staff require a support ticket to access a workspace
    - ticket is time-limited
    - allows access to specific group of employees
- Compliance on Multicloud
    - SOC 2 Type II
    - ISO 27001
    - ISO 27017
    - ISO 27018
- Additional compliance per cloud-provider
    - FedRAMP High
    - HITRUST
    - HIPPA
    - PCI
- GDPR and CCPA ready

## Instant Compute and Serverless

Configuring the data plane can be complicated and leads to over-provisioning of resouces
and higher administration costs. Databricks offers serverless compute or serverless data
plane which is available for Databricks SQL. Databricks Serverless SQL is provisioned and
managed by Databricks in the Databricks cloud account. Resources are provisioned on-demand
and destroyed when they are finished with. Serverless Compute relies on pre-configured 
database clusters which are assigned to customers as needed and supports elastic scaling
in response to demand. The compute has three layers of isolation:

- the container hosting the runtime
- the VM hosting the container
- the virtual network for the workspace

When finished the VM is terminated and not reused. Instead a new VM is deployed and allocated
to the pool. 

## Lakehouse Data Management Terminology

Delta Lake 

- provides a data storage format built for the Lakehouse and Unity Catalog. 

Unity Catalog

- provides a common governance model for data and AI assets

Metastore

- top-level logical construct for organising data and associated metadata. Functions as a reference for a collection of metadata and a link to the cloud storage container.

Catalog

- top-most container for data objects in Unity Catalog. Several Catalogs can exist in a Unity Catalog. Each Catalog represents the start of the data objects namespace, followed by the schema and tablename

Schema

- acts as a container for data assets like tables, views and functions

Tables

- defined by two distinct elements: metadata and data. Metadata here is the comments, tags and list of columns and data types. Tables can be either managed or external. External tables store their data in an external data store: with managed tables, data is stored in the Metastores defined storage location. 

Views

- read-only queries: are not able to modify the underlying data. 

Storage Credentials

- created by admins and used to authenticate to cloud storage containers

External Location

- used to provide access control at the file level

Shares and Recipients

- Used in Delta Sharing, an open source protocol for sharing data across organisations. Shares are read-only, logical collections of tables. 

## Supported Workloads

- Data Warehousing 
    - using Databricks SQL to support SQL Analytics and BI tasks (ETL, queries, dashboards, reporting). Data analysts can use the tools of their choice to interact with Databricks SQL. 
- Data Engineering 
    - ingesting, cleaning and orchestrating (delivering) data. Building data pipelines. 
    - Auto Loader is an optimised data ingestion tool that processes new data files as they arrive in the lakehouse cloud storage. Auto detects schemas and enforces it. The `COPY INTO SQL` command uses the 'lake-first' approach, loading data from a folder into a Delta Lake tables. 
    - Delta Live Tables (DLT) uses a declarative syntax to build reliable data pipelines. DLT supports both SQL and Python and works with streaming and batch workloads. With DLT, engineers treat their data as code, and can use common practices from software engineering: separate dev and prod environments, test before deploying, using parameters to define environments, unit testing and documentation. 
    - Databricks workflows can be built in the UI, using the Databricks Workflows API or external orchestrators such as Apache Airflow. 
- Data Streaming
    - handling real-time data 
    - supports real-time analysis, real-time ML, real-time applications
- Data Science and Machine Learning
    - Databricks ML Runtime
        - optimised and pre-configured ML Frameworks
        - distributed ML
        - built-in Auto-ML
        - GPU support
    - MLflow is an open-sourced ML platform created by Databricks
        - track model training sessions
        - package and re-use models
        - feature store to create new features or re-use existing features, to train or score models
        - serve models to production

## ACID transations

- Atomicity
    All changes to data are performed as if they are a single operation. That is, all the changes are performed, or none of them are.
    For example, in an application that transfers funds from one account to another, the atomicity property ensures that, if a debit is made successfully from one account, the corresponding credit is made to the other account.
- Consistency
    Data is in a consistent state when a transaction starts and when it ends.
    For example, in an application that transfers funds from one account to another, the consistency property ensures that the total value of funds in both the accounts is the same at the start and end of each transaction.
- Isolation
    The intermediate state of a transaction is invisible to other transactions. As a result, transactions that run concurrently appear to be serialized.
    For example, in an application that transfers funds from one account to another, the isolation property ensures that another transaction sees the transferred funds in one account or the other, but not in both, nor in neither.
- Durability
    After a transaction successfully completes, changes to data persist and are not undone, even in the event of a system failure.
    For example, in an application that transfers funds from one account to another, the durability property ensures that the changes made to each account will not be reversed. 
