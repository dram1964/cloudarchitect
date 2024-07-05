# CosmosDB

Azure CosmosDB [documentation](https://learn.microsoft.com/en-gb/azure/cosmos-db/)

## CosmosDB for MongoDB

Create a MongoDB in CosmosDB:

```terraform
resource "azurerm_cosmosdb_account" "cosmosdb_example" {
  name                            = "cosmosdb_example"
  location                        = var.location
  resource_group_name             = var.rg_name
  offer_type                      = "Standard"
  kind                            = "MongoDB"
  enable_automatic_failover       = false
  enable_multiple_write_locations = false
  mongo_server_version            = "4.0"

  # Restrict access to chosen subnet in VNet
  # Requires service endpoint 'Microsoft.AzureCosmosDB' defined on the subnet

  is_virtual_network_filter_enabled = true
  virtual_network_rule {
    id                                   = azurerm_subnet.snet_example.id
    ignore_missing_vnet_service_endpoint = true
  }

  # Allow access from Azure Portal (https://docs.microsoft.com/azure/cosmos-db/how-to-configure-firewall#allow-requests-from-the-azure-portal)
  ip_range_filter = "104.42.195.92,40.76.54.131,52.176.6.30,52.169.50.45,52.187.184.26"

  tags = var.default_tags

  capabilities {
    name = "EnableServerless"
  }

  lifecycle {
    ignore_changes = [capabilities]
  }
  consistency_policy {
    consistency_level = "Session"
  }

  geo_location {
    location          = var.location
    failover_priority = 0
    zone_redundant    = false
  }
}

# ------------------------------------------------------------------------------------------------------
# Deploy cosmos mongo db and collections
# ------------------------------------------------------------------------------------------------------
resource "azurerm_cosmosdb_mongo_database" "mongodb" {
  name                = "Todo"
  resource_group_name = var.resource_group_name
  account_name        = azurerm_cosmosdb_account.cosmosdb_example.name
}

resource "azurerm_cosmosdb_mongo_collection" "list" {
  name                = "TodoList"
  resource_group_name = var.resource_group_name
  account_name        = azurerm_cosmosdb_account.cosmosdb_example.name
  database_name       = azurerm_cosmosdb_mongo_database.mongodb.name
  shard_key           = "_id"


  index {
    keys   = ["_id"]
    unique = true
  }
}

resource "azurerm_cosmosdb_mongo_collection" "item" {
  name                = "TodoItem"
  resource_group_name = var.resource_group_name
  account_name        = azurerm_cosmosdb_account.cosmosdb_example.name
  database_name       = azurerm_cosmosdb_mongo_database.mongodb.name
  shard_key           = "_id"

  index {
    keys   = ["_id"]
    unique = true
  }
}
```

Create the service endpoint in the subnet:

```terraform
resource "azurerm_subnet" "snet_example" {
  name                 = "snet_example"
  resource_group_name  = var.resource_group_name
  virtual_network_name = var.vnet_name
  address_prefixes     = ["10.0.1.0/24"]
  service_endpoints    = ["Microsoft.AzureCosmosDB"]
}

```

## Connect to CosmosDB for MongoDB from Python

See the [quickstart guide](https://learn.microsoft.com/en-gb/azure/cosmos-db/mongodb/quickstart-python?tabs=azure-cli%2Cvenv-windows%2Cwindows)

```python3
import os
import sys
from random import randint

import pymongo
from dotenv import load_dotenv

load_dotenv()
# use 'az cosmosdb keys list --type connection-strings --resource-group <resource-group-name> --name <cosmosdb-name>'
# then 'export COSMOS_CONNECTION_STRING=<primary_connection_string>'
CONNECTION_STRING = os.environ.get("COSMOS_CONNECTION_STRING")

DB_NAME = "adventureworks"

COLLECTION_NAME = "products"

client = pymongo.MongoClient(CONNECTION_STRING)

db = client[DB_NAME]
if DB_NAME not in client.list_database_names():
  db.command({"customAction": "CreateDatabase"})
  print("Created db '{}' with shared throughput.\n".format(DB_NAME))
else:
  print("Using database: '{}'.\n".format(DB_NAME))

# Create collection if it doesn't exist
collection = db[COLLECTION_NAME]
if COLLECTION_NAME not in db.list_collection_names():
  # Creates a unsharded collection that uses the DBs shared throughput
  db.command( {"customAction": "CreateCollection", "collection": COLLECTION_NAME})
  print("Created collection '{}'.\n".format(COLLECTION_NAME))
else:
  print("Using collection: '{}'.\n".format(COLLECTION_NAME))

indexes = [ {"key": {"_id": 1}, "name": "_id_1"}, {"key": {"name": 2}, "name": "_id_2"}, ]
db.command( { "customAction": "UpdateCollection", "collection": COLLECTION_NAME, "indexes": indexes, })
print("Indexes are: {}\n".format(sorted(collection.index_information())))

"""Create new document and upsert (create or replace) to collection"""
product = { "category": "gear-surf-surfboards", "name": "Yamba Surfboard-{}".format(randint(50, 5000)), "quantity": 1, "sale": False, }
result = collection.update_one( {"name": product["name"]}, {"$set": product}, upsert=True)
print("Upserted document with _id {}\n".format(result.upserted_id))

doc = collection.find_one({"_id": result.upserted_id})
print("Found a document with _id {}: {}\n".format(result.upserted_id, doc))

"""Query for documents in the collection"""
print("Products with category 'gear-surf-surfboards':\n")
allProductsQuery = {"category": "gear-surf-surfboards"}
for doc in collection.find(allProductsQuery).sort( "name", pymongo.ASCENDING):
  print("Found a product with _id {}: {}\n".format(doc["_id"], doc))
```                                                                                                        
