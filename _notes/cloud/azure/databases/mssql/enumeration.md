---
title: "Azure MSSQL Enumeration"
order: 72
description: Enumeration of Azure MSSQL
---

## List servers

```bash
az sql server list --resource-group <resource-group-name> --output table
```

```bash
az sql server list --expand-ad-admin --query "[].{Name:name, FQDN:fullyQualifiedDomainName, PublicAccess:publicNetworkAccess,restrictOutboundNetworkAccess:restrictOutboundNetworkAccess, ResourceGroup:resourceGroup, Admin:administratorLogin, EntraAdmin:externalAdministrator.login, AdminType:externalAdministrator.principalType, AdminSID:externalAdministrator.sid}" --output table
```

### List Server Firewalls

```bash
az sql server firewall-rule list --output table \
  --query "[].{Rule:name, Start:startIpAddress, End:endIpAddress}" \
   --resource-group $RESOURCE_GROUP --server $SQL_SERVER
```

### List of Entra ID administrators in a server

```bash
az sql server ad-admin list --resource-group $RESOURCE_GROUP --server $SQL_SERVER --query "[].{Admin:login, Type:principalType, Tenant:tenantId, SID:sid}" --output table
```

### List Databases in a SQL server

```bash
az sql db list --server $SQL_SERVER \
   --resource-group $RESOURCE_GROUP \
   --query "[].{Name:name, Status:status, Tier:sku.tier, Ledger:ledgerOn, InfraEncrypt:isInfraEncryptionEnabled, Backup:currentBackupStorageRedundancy}" \
   --output table 
```

### Get details of a specific database

```bash
az sql db show --name <database_name> --server <server_name> --resource-group <resource_group>
```

### List of operations performed on the database.

```bash
az sql db op list --database <database_name> --server <server_name> --resource-group <resource_group>
```

### List deleted databases in a SQL server

```bash
az sql db list-deleted --server $SQL_SERVER --resource-group $RESOURCE_GROUP
```

### List all elastic pools in a SQL server

```bash
az sql elastic-pool list --server <server_name> --resource-group <resource_group> #--output table
```
### List all databases in a specific elastic pool

```bash
az sql elastic-pool show --name <elastic_pool_name>  --server <server_name> --resource-group <resource_group>
```

```bash
az sql elastic-pool list-dbs --name <elastic_pool_name>  --server <server_name> --resource-group <resource_group>
```

### List all managed Instances

```bash
az sql mi list
az sql mi show --resource-group <res-grp> --name <name>
az sql midb list
az sql midb show --resource-group <res-grp> --name <name>
```

### List all sql VM

```bash
az sql vm list
az sql vm show --resource-group <res-grp> --name <name>
```

### List schema by the database

```bash
az rest --method get --uri "https://management.azure.com/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Sql/servers/<serverName>/databases/<databaseName>/schemas?api-version=2021-11-01"
```
### Get tables of a database with the schema

```bash
az rest --method get --uri "https://management.azure.com/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Sql/servers/<serverName>/databases/<databaseName>/schemas/<schemaName>/tables?api-version=2021-11-01"
```

### Get columns of a database

```bash
az rest --method get --uri "https://management.azure.com/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Sql/servers/<serverName>/databases/<databaseName>/columns?api-version=2021-11-01"

```

### Get columns of a table

```bash
az rest --method get --uri https://management.azure.com/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Sql/servers/<serverName>/databases/<databaseName>/schemas/<schemaName>/tables/<tableName>/columns?api-version=2021-11-01"
```

### Get DataMaskingPolicies of a database

```bash
az rest --method get --uri "https://management.azure.com/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Sql/servers/<serverName>/databases/<databaseName>/dataMaskingPolicies/Default/rules?api-version=2021-11-01"
```

#### Get DataMasking for columns

```bash
az rest --method GET \
  --uri "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Sql/servers/$SQL_SERVER/databases/$DB_NAME/dataMaskingPolicies/Default/rules?api-version=2021-11-01"
```

### Get DataMaskingPolicies of a database using sql

```sql
-- Identificar security policies
SELECT * FROM sys.security_policies; 
GO

-- Ver qué tabla protege y el predicate function
SELECT 
    sp.name AS policy_name,
    spred.predicate_type_desc,
    spred.predicate_definition,
    OBJECT_NAME(spred.target_object_id) AS target_table
FROM sys.security_policies sp
JOIN sys.security_predicates spred ON sp.object_id = spred.object_id;
GO

-- Resolver el schema name
SELECT s.name AS schema_name, o.name AS policy_name
FROM sys.objects o
JOIN sys.schemas s ON o.schema_id = s.schema_id
WHERE o.object_id = 1941581955;
GO
```