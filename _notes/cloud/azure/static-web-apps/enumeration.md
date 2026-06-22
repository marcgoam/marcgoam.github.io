---
title: "Azure Static Web Apps Enumeration"
order: 101
description: Enumeration of Azure Static Web Apps
---

## List Static Webapps

```bash
az staticwebapp list --output table
```

```bash
az staticwebapp list --query "[].{Name:name, ResourceGroup:resourceGroup, Hostname:defaultHostname, PublicAccess:publicNetworkAccess, SKU:sku.name, Identity:identity, KeyVaultId:keyVaultReferenceIdentity, ConfigUpdates:allowConfigFileUpdates, StagingPolicy:stagingEnvironmentPolicy, Backends:linkedBackends}" --output table
```

## Get Static Webapp details

```bash
az staticwebapp show --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP --output table
```

## Get appsettings

```bash
az staticwebapp appsettings list --name $WEBAPP_NAME
```

## Get env information

```bash
az staticwebapp environment list --name $WEBAPP_NAME
az staticwebapp environment functions --name $WEBAPP_NAME
az staticwebapp secrets list --name $WEBAPP_NAME
```

## Get current snippets

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/staticSites/$WEBAPP_NAME/snippets?api-version=2022-03-01"
```

## Get API key

```bash
az staticwebapp secrets list --name $WEBAPP_NAME
```

## Get invited users

```bash
az staticwebapp users list --name $WEBAPP_NAME
```

## Get database connections

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/staticSites/$WEBAPP_NAME/databaseConnections?api-version=2021-03-01"
```

### Once you have the database connection name ("default" by default) you can get the connection string with the credentials

```bash
az rest --method POST \
  --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Web/staticSites/<app-name>/databaseConnections/default/show?api-version=2021-03-01"
```

## Check connected backends

```bash
az staticwebapp backends show --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP
```