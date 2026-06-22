---
title: "Azure Container Apps Enumeration"
order: 131
description: Enumeration of Azure Container Apps
---

## List all container apps in the subscription

```bash
az containerapp list
```

## Show detailed information about a specific container app

```bash
az containerapp show --name <app-name> --resource-group $RESOURCE_GROUP
```

## List application environments

```bash
az containerapp env list --resource-group $RESOURCE_GROUP
```

## Get configured secrets

```bash
az containerapp secret list --name <app-name> --resource-group $RESOURCE_GROUP

az containerapp secret show --name <app-name> --resource-group $RESOURCE_GROUP --secret-name <secret-name>
```

## Get authentication options

```bash
az containerapp auth  show --name <app-name> --resource-group $RESOURCE_GROUP
```

## Fetch logs from a container app

```bash
az containerapp logs show --name <app-name> --resource-group $RESOURCE_GROUP
```

## Get a shell

```bash
az containerapp exec --name <app-name> --resource-group $RESOURCE_GROUP --command "sh"
```

## Get debugging shell

```bash
az containerapp debug --name <app-name> --resource-group $RESOURCE_GROUP
```

