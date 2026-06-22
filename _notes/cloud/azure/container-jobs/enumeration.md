---
title: "Azure Container Jobs Enumeration"
order: 131
description: Enumeration of Azure Container Jobs
---

## List all container apps jobs in a resource group

```bash
az containerapp job list --resource-group $RESOURCE_GROUP
```

## Show detailed information about a specific container app job

```bash
az containerapp job show --name <job-name> --resource-group $RESOURCE_GROUP
```

## Fetch logs from a container app job

```bash
az containerapp job logs show --name <job-name> --resource-group $RESOURCE_GROUP
```

## Fetch executions from a container app job

```bash
az containerapp job execution list --name <job-name> --resource-group $RESOURCE_GROUP

az containerapp job execution show --name <job-name> --resource-group $RESOURCE_GROUP --job-execution-name <job-execution>
```

## Lost and get secrets from a container app job

```bash
az containerapp job secret list --name <job-name> --resource-group $RESOURCE_GROUP

az containerapp job secret show --name <job-name> --resource-group $RESOURCE_GROUP --secret-name <secret-name>
```

## Start a job execution (for manual jobs)

```bash
az containerapp job start --name <job-name> --resource-group $RESOURCE_GROUP
```
