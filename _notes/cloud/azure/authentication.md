---
title: "Authentication & Basic Enumeration"
order: 1
description: Firsts steps attacking Azure
---

# Log in to Azure using az cli

## User

```bash
export EMAIL=''
export PASSWORD=''
az login -u "$EMAIL" -p "$PASSWORD" [--allow-no-subscriptions]
```

## ARM (Service Principal Authentication)

```bash
export ARM_CLIENT_ID=''
export ARM_SECRET=''
export TENANT_ID='fdd066e1-ee37-49bc-b08f-d0e152119b04'
az login --service-principal -u "$ARM_CLIENT_ID" -p "$ARM_SECRET" --tenant "$TENANT_ID" [--allow-no-subscriptions]
```

## Connection Information
Displays details of the currently authenticated user.

```bash
az ad signed-in-user show
```

## List all subscriptions accessible by the current user

```bash
az account list --output table
```

## Usfeul information

```bash
export SUBSCRIPTION_ID=$(az account show --query id --output tsv)
export TENANT_ID=$(az account show --query tenantId --output tsv)
export MY_OID=$(az ad sp show --id <ID> --query id -o tsv)
```

<br>

# Enumerate ARM Resources

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resources?api-version=2021-04-01" \
  | jq -r '.value[] | "\(.name) | \(.type) | \(.id)"'
```

<br>

# Check Effective Permissions

Check what actions you can perform at each scope (subscription → resource group → resource):

## At subscription level

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.Authorization/permissions?api-version=2022-04-01"
```

## At resource group level

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/<RG>/providers/Microsoft.Authorization/permissions?api-version=2022-04-01"
```

## At resource level

```bash
az rest --method GET \
  --url "https://management.azure.com/$RESOURCE_PATH/providers/Microsoft.Authorization/permissions?api-version=2022-04-01"
```
<br>

# Check RBAC Role Assignments

## All role assignments for current principal

```bash
az role assignment list --assignee "$CLIENT_ID" --all --output table
```

## Inspect a custom role definition

```bash
az role definition list --name "<role-name>" \
  --query "[].permissions[0].{actions:actions, dataActions:dataActions}"
```
