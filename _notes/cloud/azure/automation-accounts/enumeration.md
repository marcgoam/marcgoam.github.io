---
title: "Azure Automation Accounts Enumeration"
order: 161
description: Enumeration of Automation Accounts
---

## List Automation Accounts

```bash
az automation account list --output table
```

```bash
az automation account list --output json --query "[].{name:name,description:description,publicNetworkAccess:publicNetworkAccess,privateEndpointConnections:privateEndpointConnections,disableLocalAuth:disableLocalAuth,identity:identity}"
```

## Get keys of automation accounts

```bash
az automation account list-keys --automation-account-name $AUTOMATION_NAME --resource-group $RESOURCE_GROUP
```

## Get schedules of automation account

```bash
az automation schedule list --automation-account-name $AUTOMATION_NAME --resource-group $RESOURCE_GROUP
```

## Check the managed identities in `identity`

```bash
az automation account show --name <AUTOMATION-ACCOUNT> --resource-group <RG-NAME>
```

## Get connections of automation account

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Automation/automationAccounts/<automation-account-name>/connections?api-version=2023-11-01"
```

## Get credentials of automation account

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Automation/automationAccounts/$AUTOMATION_NAME/credentials?api-version=2023-11-01"
```

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Automation/automationAccounts/$AUTOMATION_NAME/credentials/<credential-name>?api-version=2023-11-01"
```

## Get certificates of automation account

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Automation/automationAccounts/<automation-account-name>/certificates?api-version=2023-11-01"
```
## Get variables of automation account

It's possible to get the value of unencrypted variables but not the encrypted ones

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Automation/automationAccounts/$AUTOMATION_NAME/variables?api-version=2023-11-01"
```
## Get runbooks of an automation account

```bash
az automation runbook list --automation-account-name $AUTOMATION_NAME --resource-group $RESOURCE_GROUP
```
## Get runbook content

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Automation/automationAccounts/$AUTOMATION_NAME/runbooks/$RUNBOOK_NAME/content?api-version=2023-11-01"
```

## Get jobs of an automation account

```bash
az automation job list --automation-account-name $AUTOMATION_NAME --resource-group $RESOURCE_GROUP
```

## Get job output

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Automation/automationAccounts/<automation-account-name>/jobs/<job-name>/output?api-version=2023-11-01"
```

## Get the Runbook content when the job was executed

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Automation/automationAccounts/<automation-account-name>/jobs/<job-name>/runbookContent?api-version=2023-11-01"
```
## Get webhooks inside an automation account

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Automation/automationAccounts/<automation-account-name>/webhooks?api-version=2018-06-30"
```

## Get the source control setting of an automation account (if any)

inside the output it's possible to see if the autoSync is enabled, if the publishRunbook is enabled and the repo URL

```bash
az automation source-control list --automation-account-name <AUTOMATION-ACCOUNT> --resource-group <RG-NAME>
```

## Get custom runtime environments

Check in defaultPackages for custom ones, by default Python envs won't have anything here and PS1 envs will have "az" and "azure cli"

```bash
az automation runtime-environment list \
    --resource-group <res-group>> \
    --automation-account-name <account-name> \
    --query "[?!(starts_with(description, 'System-generated'))]"
```

## Get hybrid worker groups for an automation account

```bash
az automation hrwg list --automation-account-name <AUTOMATION-ACCOUNT> --resource-group <RG-NAME>
```

## Get more details about a hybrid worker group (like VMs inside it)

```bash
az rest --method GET --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>>/providers/Microsoft.Automation/automationAccounts/<automation-account-name>/hybridRunbookWorkerGroups/<hybrid-worker-group-name>/hybridRunbookWorkers?&api-version=2021-06-22"
```
