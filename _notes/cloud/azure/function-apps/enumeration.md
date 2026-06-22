---
title: "Azure Function Apps Enumeration"
order: 101
description: Enumeration of Azure Function Apps
---

## List all the functions

```bash
az functionapp list
```

```bash
az functionapp list --query "[].{Name:name,defaultHostName:defaultHostName,hostNames:hostNames,hostNamesDisabled:hostNamesDisabled, RG:resourceGroup, Runtime:functionAppConfig.runtime.name, Identity:identity.type, PrincipalID:identity.principalId, StorageURL:functionAppConfig.deployment.storage.value, HTTPS:httpsOnly}" -o json
```

## List functions in a function-app (endpoints)

```bash
az functionapp function list --name <app-name> --resource-group $RESOURCE_GROUP
```

```bash
az functionapp function list -o table \
  --query "[].{Name:name, Trigger:config.bindings[0].type, Queue:config.bindings[0].queueName, Script:config.scriptFile,functionDirectory:config.functionDirectory,Language:language,href:href}" \
  --name $FUNCTION_NAME --resource-group $RESOURCE_GROUP
```

## Get details about the source of the function code

```bash
az functionapp deployment source show \
	--name $FUNCTION_NAME \
	--resource-group $RESOURCE_GROUP
```

If error like "This is currently not supported.", then, this is probably using a container

### Get more info if a container is being used

```bash
az functionapp config container show \
	--name $FUNCTION_NAME \
	--resource-group $RESOURCE_GROUP
```

## Get settings (and privesc to the storage account)

```bash
az functionapp config appsettings list --name $FUNCTION_NAME --resource-group $RESOURCE_GROUP
```

## Get network restrictions

```bash
az functionapp config access-restriction show --name $FUNCTION_NAME --resource-group $RESOURCE_GROUP
```
  
## Get connection strings

```bash
az rest --url "https://management.azure.com/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/sites/$FUNCTION_NAME/hostruntime/admin/vfs/function_app.py?relativePath=1&api-version=2022-03-01"
```

## Get SCM credentials

```bash
az functionapp deployment list-publishing-profiles --name $FUNCTION_NAME --resource-group $RESOURCE_GROUP
```

```bash
az functionapp deployment list-publishing-credentials --name $FUNCTION_NAME --resource-group $RESOURCE_GROUP   
```

## Get function, system and master keys

```bash
az functionapp keys list --name $FUNCTION_NAME --resource-group $RESOURCE_GROUP 
```  
## Get Host key

```bash
az rest --method POST --uri "https://management.azure.com/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/sites/<app-name>/functions/<function-endpoint-name>/listKeys?api-version=2022-03-01"
```

## Get source code with Master Key of the function

```bash
curl "https://$FUNCTION_NAME.azurewebsites.net/admin/vfs/home/site/wwwroot/function_app.py?code=<master-key>" -v
```

## Get source code with Azure permissions

```bash
az rest --url "https://management.azure.com/$SUBSCRIPTION_ID/resourceGroups/<res-group>/providers/Microsoft.Web/sites/<app-name>/hostruntime/admin/vfs/function_app.py?relativePath=1&api-version=2022-03-01"
```