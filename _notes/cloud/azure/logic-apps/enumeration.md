---
title: "Azure Logic Apps Enumeration"
order: 171
description: Enumeration of Logic Apps
---

## List

```bash
az logic workflow list --resource-group <ResourceGroupName>
```

## Get info

```bash
az logic workflow show --name <LogicAppName> --resource-group <ResourceGroupName>
```

## Get details of a specific Logic App workflow, including its connections and parameters

```bash
az rest --method GET --uri "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{workflowName}?api-version=2016-10-01&$expand=connections.json,parameters.json"
```

## Get details about triggers for a specific Logic App

```bash
az rest --method GET --uri "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{logicAppName}/triggers?api-version=2016-06-01"
```

## Get workflow versions

```bash
az rest --method GET --uri "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Logic/workflows/<workflow-name>/versions?api-version=<api-version>"
```

## Get workflow version info

```bash
az rest --method GET --url 'https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Logic/workflows/<workflow-name>/versions/<version-id>?api-version=2016-10-01&$expand=connections.json,parameters.json'
```

## Get the callback URL for a specific trigger in a Logic App

```bash
az rest --method POST --uri "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{logicAppName}/triggers/{triggerName}/listCallbackUrl?api-version=2016-06-01"
```

## Get the history of a specific trigger in a Logic App

```bash
az rest --method GET --uri "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{logicAppName}/triggers/{triggerName}/histories?api-version=2016-06-01"
```

## List all runs of a specific Logic App workflow

```bash
az rest --method GET --uri "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{workflowName}/runs?api-version=2016-06-01"
```

## Get details of a specific workflow run

```bash
az rest --method GET --url 'https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{workflowName}/runs/{runId}?api-version=2016-10-01&$expand=properties/actions,workflow/properties'
```