---
title: "Azure CosmosDB Explotation"
order: 82
description: Explotation of Azure CosmosDB
---

## Script for reading cosmosDB database

### With client secret

```python
from azure.identity import ClientSecretCredential
from azure.cosmos import CosmosClient

credential = ClientSecretCredential(
  tenant_id="$TENANT_ID",
  client_id="$CLIENT_ID",
  client_secret="$CLIENT_SECRET"
)

client = CosmosClient(
  url="https://$COSMOSDB_SERVER.documents.azure.com:443/",
  credential=credential
)

db = client.get_database_client("<database>")
container = db.get_container_client("<container>")

for item in container.read_all_items():
    print(item)
```

### With Access key

```python
from azure.cosmos import CosmosClient

client = CosmosClient(
  url="https://$COSMOSDB_SERVER.documents.azure.com:443/",
  credential="<access_key>"
)

db = client.get_database_client("<database>")
container = db.get_container_client("<container>")

for item in container.read_all_items():
    print(item)
```

## Create role

> The following permisions are required:
> - **Microsoft.DocumentDB/databaseAccounts/sqlRoleDefinitions/write**
> - **Microsoft.DocumentDB/databaseAccounts/sqlRoleDefinitions/read**
> - **Microsoft.DocumentDB/databaseAccounts/sqlRoleAssignments/write**
> - **Microsoft.DocumentDB/databaseAccounts/sqlRoleAssignments/read**
>
> With this permissions you can priviledgeescalate giving a user the pemrissions to execute queries and connect to the database. First a definition role is created giving the necesary permissions and scopes.


```bash
ROLE_ID=$(uuidgen | tr '[:upper:]' '[:lower:]')
```

```bash
az cosmosdb sql role definition create \
    --account-name <account_name> \
    --resource-group <resource_group_name> \
    --body '{
      "Id": "<Random-Unique-ID>", # For example 12345678-1234-1234-1234-123456789az
      "RoleName": "CustomReadRole",
      "Type": "CustomRole",
      "AssignableScopes": [
        "/subscriptions/<subscription_id>/resourceGroups/sqldatabase/providers/Microsoft.DocumentDB/databaseAccounts/<account_name>"
      ],
      "Permissions": [
        {
          "DataActions": [
            "Microsoft.DocumentDB/databaseAccounts/readMetadata",
            "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/items/read",
            "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers/*"
          ]
        }
      ]
    }'
```

### Assign custom role

Obtain oid

```bash
az account get-access-token --query accessToken -o tsv \
  | cut -d'.' -f2 | base64 -d 2>/dev/null | jq .oid
```

```bash
az cosmosdb sql role assignment create \
    --account-name <account_name> \
    --resource-group <resource_group_name> \
    --role-definition-id <Random-Unique-ID-used-in-definition> \
    --principal-id <principal_id-togive-perms> \
    --scope "/"
```
