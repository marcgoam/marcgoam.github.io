---
title: "Azure MySQL Enumeration"
order: 75
description: Enumeration of Azure MySQL
---

## List all flexible-servers

```bash
az mysql flexible-server list
```

```bash
az mysql flexible-server list --output table --resource-group $RESOURCE_GROUP --query "[].{Server:name, FQDN:fullyQualifiedDomainName, Port:databasePort, Version:version, Admin:administratorLogin, PublicAccess:network.publicNetworkAccess, EntraAuth:authConfig.activeDirectoryAuth, HA:highAvailability.mode, Tier:sku.tier}" 
```

## List databases in a flexible-server

```bash
az mysql flexible-server db list --resource-group <resource-group-name> --server-name <server_name>
```

```bash
az mysql flexible-server db list --output table\
  --query '[].{Name:name,collation:collation,resourceGroup:resourceGroup,systemData:systemData}' \
  --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

## Show specific details of a MySQL database

```bash
az mysql flexible-server db show --resource-group <resource-group-name> --server-name <server_name> --database-name <database_name>
```

## List firewall rules of the a server

```bash
az mysql flexible-server firewall-rule list --resource-group <resource-group-name> --name <server_name>
```

```bash
az mysql flexible-server firewall-rule list --query "[].{RuleName:name, Start:startIpAddress, End:endIpAddress}" --output table \
--resource-group $RESOURCE_GROUP --name $SERVER_NAME
```

## List all Entra ID admins in a server

```bash
az mysql flexible-server ad-admin list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

## List all managed identities from the server

```bash
az mysql flexible-server identity list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

## List the server backups

```bash
az mysql flexible-server backup list --resource-group $RESOURCE_GROUP --name $SERVER_NAME
```

## List all read replicas for a given server

```bash
az mysql flexible-server replica list --resource-group $RESOURCE_GROUP --name $SERVER_NAME
```

## Enumerate Monitoring Mechanisms

### Get the server's advanced threat protection setting

```bash
az mysql flexible-server advanced-threat-protection-setting show --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

### Audit Logging Enabled

```bash
az mysql flexible-server parameter show --name "audit_log_enabled" \
--query "{Parameter:name, Status:value}" \
--resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

### List all of the maintenances of a flexible server

```bash
az mysql flexible-server maintenance list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

### List log files for a server

```bash
az mysql flexible-server server-logs list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```