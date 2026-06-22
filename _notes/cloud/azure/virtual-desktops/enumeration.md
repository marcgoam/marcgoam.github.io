---
title: "Azure Virtual Desktops Enumeration"
order: 181
description: Enumeration of Virtual Desktops
---

```bash
az extension add --name desktopvirtualization
```

## List HostPools

```bash
az desktopvirtualization hostpool list
```

## List Workspaces

```bash
az desktopvirtualization workspace list
```

## List Application Groups

```bash
az desktopvirtualization applicationgroup list 
```

## List Applications in a Application Group

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/applicationGroups/{applicationGroupName}/applications?api-version=2024-04-03"
```

## Check if Desktops are enabled

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/applicationGroups/{applicationGroupName}/desktops?api-version=2024-04-03"
```

## List hosts

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/hostPools/{hostPoolName}/sessionHosts?api-version=2024-04-03"
```

## List App Attach packages

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/appAttachPackages?api-version=2024-04-03"
```

## List user sessions

```bash
az rest --method GET --url "https://management.azure.com/ssubscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/hostpools/{hostPoolName}/sessionhosts/{hostPoolHostName}/userSessions?api-version=2024-04-03"
```


## List Desktops

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/applicationGroups/{applicationGroupName}/desktops?api-version=2024-04-03"
```

## List MSIX Packages

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/hostPools/{hostPoolName}/msixPackages?api-version=2024-04-03"
```

## List private endpoint connections associated with hostpool.

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/hostPools/{hostPoolName}/privateEndpointConnections?api-version=2024-04-03"
```

## List private endpoint connections associated By Workspace.

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/workspaces/{workspaceName}/privateEndpointConnections?api-version=2024-04-03"
```

## List the private link resources available for a hostpool.

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/hostPools/{hostPoolName}/privateLinkResources?api-version=2024-04-03"
```

## List the private link resources available for this workspace.

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DesktopVirtualization/workspaces/{workspaceName}/privateLinkResources?api-version=2024-04-03"
```