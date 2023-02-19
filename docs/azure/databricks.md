# Azure Databricks

[Azure Databricks Documentation](https://learn.microsoft.com/en-us/azure/databricks/introduction/)

The Azure Databricks Lakehouse Platform provides a unified set of tools
for processing, storing and analysing data at scale. 
To use Azure Databricks you 
need to configure a workspace to integrate Azure Databricks infrastructure
with your cloud account. Azure Databricks then deploys ephemeral 
compute clusters using cloud resources in your account to process and 
store data. 

The Azure Databricks workspace provides user interfaces for many core data
tasks, such as:

- Interactive notebooks
- Workflows scheduler and manager
- SQL editor and dashboards
- Data ingestion and governance
- Data discovery, annotation and exploration
- Compute management
- ML experiment tracking
- ML model serving
- Feature store
- Source control with Git

You can also interact with Databricks using CLI, Terraform or REST. 

Common use cases for Azure Databricks:

- Enterprise [Data Lakehouse](https://learn.microsoft.com/en-us/azure/databricks/lakehouse/): combines data warehouse and data lake techniques for a unified enterprise data solution
- ETL and Data Engineering: combines Apache Spark with Delta Lake for ETL
- Machine Learning, AI and Data Science: expands core functionality with MLflow and Databricks Runtime for Machine Learning
- Data Warehousing, Analytics and BI: SQL warehousing can be queried using Notebooks supporting Python, R, Scala or SQL. 
- Data Governance: using [Unity Catalog](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/) to configure a unified data governance model. [Delta Sharing](https://learn.microsoft.com/en-us/azure/databricks/data-sharing/) can be leveraged for sharing outside of your secure environment. 
- [DevOps, CI/CD and Orchestration](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/)
- [Real-time and Streaming Analytics](https://learn.microsoft.com/en-us/azure/databricks/structured-streaming/): using Apache Spark


## Databricks Concepts

- Accounts and Workspaces
    - A Workspace is an Azure Databricks deployment in the cloud providing an environment for your team to access Databricks assets. An account is a single entity that can include multiple workspaces
- Billing
    - based on DBUs (DataBricks Units), or processing capability per hour
- Authentication and Authorisation
    - User: a unique individual represented by an email address
    - Service Principal: a service identity for use with jobs, automated tools and systems
    - Group: a collection of identities
    - ACL: list of permissions attached to the workspace, cluster, job, table or experiment. Specifies which users are granted access to the object and the operations they are allowed to perform
    - Personal Access Token: used to authenticate to the REST API and to connect to SQL warehouses
- Databricks Data Science and Engineering
    - Notebook: web-based interface to run commands, create visualisations and narrative text
    - Dashboard: collection of visualisations
    - Library: a package of code
    - Repo: folder synchronised to a remote Git repository
    - Experiment: a collection of MLflow runs for training an ML model
    - Interfaces: UI, REST API and a CLI
    - Databricks File System (DBFS): a filesystem abstraction layer over a blob store
    - Database: an organised collection of information
    - Table: a representation of structured data
    - Metastore: stores all the structural information of the various tables and partitions in the data warehouse
    - Visualisation: a graphical representation of query results
    - Cluster: a set of computation resources and configurations used to run notebooks and jobs. Job clusters are automatically created and destroyed by the job scheduler for each job run. All-purpose clusters can be manually created to allow sharing between multiple users.
    - Pool: a set of ready-to-use instances that are used to reduce cluster start and auto-scaling times.
    - Databricks runtime: a set of core components that run on the clusters. There are several types:
        - Databricks Runtime: includes Apache Spark
        - Databricks Runtime for Machine Learning: built on the Datbricks Runtime and adds popular libraries such as TensorFlow, Keras, PyTorch and XGBoost
        - Databricks Light: used for non-interactive job submission
    - Workflows: frameworks to develop and run data processing pipelines
    - Workload
        - Data Engineering: an automated workload running on a job cluster
        - Data Analytics: an interactive workload running on an all-purpose cluster
    - Execution Context: the state for a REPL (read-eval-print loop) environment (interactive shell) 
- Databrick Machine Learning
    - Experiments: main unit of organisation for tracking ML model develpment. Used to organise, display and control access to individual logged-runs of model training code
    - Feature Store: a centralised repository of features. 
    - Models: a trained machine learning or deep learning model that has been registered in the Model Registry
- Databricks SQL: geared towards data analysts who prefer to work with SQL queries and BI tools
    - REST API: an interface for task automation on Databricks SQL objects
    Dashboard: collection of visualisations
    Alert: notification of a threshold that was reached
    Query: a valid SQL statement
    SQL warehouse: computation resources used to execute SQL queries
    Query history: list of executed queries and their performance history

## [Create an Azure Databricks Workspace with Azure CLI](https://learn.microsoft.com/en-us/azure/databricks/getting-started/#azure-cli)

Sign-in to your Azure Subscription and install the Azure CLI extension for
databricks:

    az extension add --name databricks

Create the Azure Databricks workspace:

    RG_NAME=databricks-quickstart
    LOCATION=uksouth
    WS_NAME=mydatabricksws

    az group create --name $RG_NAME --location $LOCATION

    az databricks workspace create \
        --resource-group $RG_NAME \
        --name $WS_NAME \
        --location $LOCATION \
        --sku standard
