---
title: "Azure Container Registry Enumeration"
order: 101
description: Enumeration of Azure Container Registry
---

## Login

### Using EntraID

```bash
az acr login -n $ACR_NAME 
```

### Using admin user

```bash
az acr update -n $ACR_NAME --admin-enable
```

```bash
docker login -u $ACR_NAME -p <password> <login_server>
```

```bash
az acr login -n containerInstancesLab3ba59b400 --expose-token

docker login containerinstanceslab3ba59b400.azurecr.io \
  -u 00000000-0000-0000-0000-000000000000 \
  -p '<refreshToken>'
```

### Generate a token

```bash
az acr token create \
	--registry <container_name>
	--name token
	--scope-map _repositories_admin
```

```bash
docker login <login_server>.azurecr.io -u token -p <password>
```

### Enable anonymous pull

```bash
az acr update --name $ACR_NAME --anonymous-pull-enabled true
```

## List of all the registries

Check the network, managed identities, adminUserEnabled, softDeletePolicy, url...

```bash
az acr list
```

```bash
az acr list --query "[].{Name:name,loginServer:loginServer, AdminUser:adminUserEnabled, PublicAccess:publicNetworkAccess, AnonPull:anonymousPullEnabled, SKU:sku.tier, TrustPolicy:policies.trustPolicy.status, Encryption:encryption.status, resourceGroup:resourceGroup}" --output table
```

## List tokens of a registry

```bash
az acr token list --registry $ACR_NAME --resource-group $RESOURCE_GROUP
```

## List repositories in a registry

```bash
az acr repository list --name $ACR_NAME --resource-group $RESOURCE_GROUP
```

## List the tags of a repository

```bash
az acr repository show-tags --repository <repository-name> --name $ACR_NAME 
```

## List deleted repository tags

```bash
az acr repository list-deleted --name $ACR_NAME 
```

## List tasks

### Check the git URL or the command

```bash
az acr task list --registry $ACR_NAME 
```

```bash
az acr task list --output table \
  --query "[].{Name:name, Identity:identity.type, Source:step.contextPath, Schedule:trigger.timerTriggers[0].schedule, Status:status, imageName:step.imageNames}"  \
   --registry $ACR_NAME 
```

### List tasks runs

```bash
az acr task list-runs --registry $ACR_NAME 
```

### List connected registries

```bash
az acr connected-registry list --registry $ACR_NAME 
```

### List cache

```bash
az acr cache list --registry $ACR_NAME 
az acr cache show -r $ACR_NAME  -n <rule-name>
```

## Allow anonymous pull access

```bash
az acr update --name $ACR_NAME  --anonymous-pull-enabled true
```

