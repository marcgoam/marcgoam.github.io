---
title: "Azure Container Instances Enumeration"
order: 121
description: Enumeration of Azure Container Instances
---

## List all container instances in the subscription

```bash
az container list
```

```bash
az container list --query "[].{Name:name, ResourceGroup:resourceGroup, FQDN:ipAddress.fqdn, PublicIP:ipAddress.ip, Port:ipAddress.ports[0].port, Identity:identity.type, Image:containers[0].image, OS:osType, Registry:imageRegistryCredentials[0].server}" --output json
```

## Show detailed information about a specific container instance

```bash
az container show --name $ACI_NAME --resource-group $RESOURCE_GROUP
```

## Fetch logs from a container

```bash
az container logs --name $ACI_NAME--resource-group $RESOURCE_GROUP
```

## Execute a command in a running container and get the output

```bash
az container exec --name $ACI_NAME --resource-group $RESOURCE_GROUP --exec-command "/bin/h"
```

## Get yaml configuration of the container group

```bash
az container export  --name $ACI_NAME --resource-group $RESOURCE_GROUP --file </path/local/file.yml>
```
