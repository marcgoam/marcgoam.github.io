---
title: "Azure CosmosDB Enumeration"
order: 81
description: Enumeration of Azure CosmosDB
---

## Azure CosmosDB

### List Azure Cosmos DB database accounts.

```bash
az cosmosdb list --resource-group $RESOURCE_GROUP
az cosmosdb show --resource-group $RESOURCE_GROUP --name <AccountName>
```

```bash
az cosmosdb list -o table \
  --query "[].{Name:name, Kind:kind, Endpoint:documentEndpoint, PublicAccess:publicNetworkAccess, NetworkBypass:networkAclBypass, Location:location,NoVnet:isVirtualNetworkFilterEnabled, LocalAuth:disableLocalAuth}" \
  --resource-group $RESOURCE_GROUP
```

### Lists the virtual network accounts associated with a Cosmos DB account

```bash
az cosmosdb network-rule list --resource-group $RESOURCE_GROUP --name <AccountName>
```

### List the access keys or connection strings for a Azure Cosmos DB

```bash
az cosmosdb keys list --resource-group $RESOURCE_GROUP --name $SERVER_NAME
```

### List all the database accounts that can be restored.

```bash
az cosmosdb restorable-database-account list --account-name <AccountName>
```

### Show the identities for a Azure Cosmos DB database account.

```bash
az cosmosdb identity show --resource-group $RESOURCE_GROUP --name <AccountName>
```

## CosmoDB (NoSQL)

### List the NoSQL databases under an Azure Cosmos DB account.

```bash
az cosmosdb sql database list --resource-group $RESOURCE_GROUP --account-name <AccountName>
```

### List the NoSQL containers under an Azure Cosmos DB SQL database.

```bash
az cosmosdb sql container list --account-name <AccountName> --database-name <DatabaseName> --resource-group $RESOURCE_GROUP
```

### List all NoSQL role assignments under an Azure Cosmos DB

```bash
az cosmosdb sql role assignment list --resource-group $RESOURCE_GROUP --account-name <AccountName>
```

### List all NoSQL role definitions under an Azure Cosmos DB

```bash
az cosmosdb sql role definition list --resource-group $RESOURCE_GROUP --account-name <AccountName>
```

### Create database

```bash
az cosmosdb sql database create \
    --account-name <AccountName> \
    --resource-group $RESOURCE_GROUP \
    --name <name-database>
```

### Create container

```bash
az cosmosdb sql container create \
    --account-name <AccountName> \
    --resource-group $RESOURCE_GROUP \
    --database-name <name-database> \
    --name <container-name> \
    --partition-key-path /mypartitionkey
```

### Access database with key

```python
from azure.cosmos import CosmosClient

client = CosmosClient(
  url="https://<account-name>.documents.azure.com:443/",
  credential="<key>"
)

db = client.get_database_client("<database>")
container = db.get_container_client("<container>")

for item in container.read_all_items():
    print(item)
```

### Create custom role

Generate role uid



### Access using credentials

```python
from azure.identity import ClientSecretCredential
from azure.cosmos import CosmosClient

credential = ClientSecretCredential(
  tenant_id="$TENANT_ID",
  client_id="$CLIENT_ID",
  client_secret="$CLIENT_SECRET"
)

client = CosmosClient(
  url="https://<account-name>.documents.azure.com:443/",
  credential=credential
)

db = client.get_database_client("<database>")
container = db.get_container_client("<container>")

for item in container.read_all_items():
    print(item)
```