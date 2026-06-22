---
title: "Azure Container Instances Enumeration"
order: 122
description: Enumeration of Azure Container Instances
---

## Execute command inside container

> The following permissions are required:
> - **Microsoft.ContainerInstance/containerGroups/read**
> - **Microsoft.ContainerInstance/containerGroups/containers/exec/action**
>
> These permissions allow the user to execute a command in a running container. This can be used to escalate privileges in the container if it has any managed identity attached.

```bash
az container exec \
  --name $ACI_NAME \
  --resource-group $RESOURCE_GROUP \
  --exec-command '/bin/sh'
```

It’s also possible to read the output of the container with:

```bash
az container attach --name $ACI_NAME --resource-group $RESOURCE_GROUP
````

Or get the logs with:

``bash
az container logs --name $ACI_NAME --resource-group $RESOURCE_GROUP
```

## Restart Container

> The permission required is:
> - **Microsoft.ContainerInstance/containerGroups/restart/action**
>
> Allows restarting a specific container group within Azure Container Instances.

```bash
az container restart \
  --name $ACI_NAME \
  --resource-group $RESOURCE_GROUP
```

## Get token

```bash
curl -s -H "Metadata: true" \
  "http://169.254.169.254/metadata/identity/oauth2/token?\
api-version=2018-02-01&resource=https://vault.azure.net" \
  | jq -r '.access_token
```