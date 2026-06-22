---
title: "Azure PostgreSQL Explotation"
order: 79
description: Explotation of Azure PostgreSQL
---

## Log in

```bash
psql "host=$SERVER_NAME.postgres.database.azure.com \
  dbname=<database> \
  user=<user> \
  password=<password> \
  sslmode=require"
```

## Modify configurations of the server

> The following permission is required:
> - Microsoft.DBforPostgreSQL/flexibleServers/read
> - Microsoft.DBforPostgreSQL/flexibleServers/write
> 
> With this permission, you can create, update, or delete PostgreSQL Flexible Server instances on Azure. This includes provisioning new servers, modifying existing server configurations, decommissioning servers, or change the admin user’s password.

### Change firewall rules

```bash
az postgres flexible-server firewall-rule create \
  --name $SERVER_NAME \
  --resource-group $RG \
  --start-ip-address $MYIP \
  --end-ip-address $MYIP
```

### Change admin's password

```bash
az rest --method PATCH \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RG/providers/Microsoft.DBforPostgreSQL/flexibleServers/$SERVER_NAME?api-version=2022-12-01" \
  --body '{
    "properties": {
      "administratorLoginPassword": "NewP@ssword123!"
    }
  }'
```

### Enable MI

Furthermore, with the permissions you can enable the assigned identity, and operate with the managed identity attached to the server. Here you can find all the extensions that Azure PostgreSQL flexible server supports https://learn.microsoft.com/en-us/azure/cosmos-db/postgresql/reference-extensions. To be able to use these extensions some server parameters (azure.extensions) need to be changed. For example here with a managed identity that can access Azure Storage:

```bash
az postgres flexible-server parameter set \
  --server-name postgresqlserverlab2-9874cdaa \
  --resource-group $RG \
  --name azure.extensions \
  --value AZURE_STORAGE
```

```sql
-- Make sure the extension is installed
CREATE EXTENSION IF NOT EXISTS azure_storage;

-- Login using storage keys
SELECT azure_storage.account_add('<storage-account>', '<storage-key>');
-- Login using managed identity
SELECT azure_storage.account_add(azure_storage.account_options_managed_identity('<storage-account>', 'blob'));

-- List configured accounts
SELECT * FROM azure_storage.account_list();

-- List all the files in the storage account
SELECT *
FROM azure_storage.blob_list(
    '<storage-account>',
    '<container>'
);

-- Access one file inside the storage account
SELECT *
FROM azure_storage.blob_get(
    '<storage-account>',
    '<container>',
    'message.txt',
    decoder := 'text'
) AS t(content text)
LIMIT 1;
```
