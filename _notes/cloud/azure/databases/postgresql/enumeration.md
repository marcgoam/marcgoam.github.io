---
title: "Azure PostgreSQL Enumeration"
order: 78
description: Enumeration of Azure PostgreSQL
---

## List servers in a resource group

```bash
az postgres flexible-server list --resource-group $RESOURCE_GROUP
```

```bash
az postgres flexible-server list --output table \
  --query "[].{Server:name, FQDN:fullyQualifiedDomainName, Version:version, Admin:administratorLogin, PublicAccess:network.publicNetworkAccess, EntraAuth:authConfig.activeDirectoryAuth, PwdAuth:authConfig.passwordAuth, Tier:sku.tier}" \
  --resource-group $RESOURCE_GROUP 
```

## List databases in a flexible-server

```bash
az postgres flexible-server db list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

```bash
az postgres flexible-server db list \
  --resource-group $RESOURCE_GROUP \
  --server-name $SERVER_NAME \
  --query "[].{Name:name, ResourceGroup:resourceGroup, SystemData:systemData}" \
  --output table
```

## Show specific details of a Postgres database

```bash
az postgres flexible-server db show --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME --database-name $DB_NAME
```

## List firewall rules of the a server

```bash
az postgres flexible-server firewall-rule list --resource-group $RESOURCE_GROUP --name $SERVER_NAME
```

```bash
az postgres flexible-server firewall-rule list \
  --query "[].{Rule:name, Start:startIpAddress, End:endIpAddress}" \ 
  --output table \
   --resource-group $RESOURCE_GROUP --name $SERVER_NAME
```

## List parameter values for a flexible server

```bash
az postgres flexible-server parameter list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

## List private link

```bash
az postgres flexible-server private-link-resource list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

## List all Entra ID admins in a server

```bash
az postgres flexible-server ad-admin list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```

## List all assigned managed identities to the server

```bash
az postgres flexible-server identity list --resource-group $RESOURCE_GROUP --server-name $SERVER_NAME
```
