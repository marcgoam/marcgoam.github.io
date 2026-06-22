---
title: "Key Vaults Enumeration"
order: 31
description: Enumeration of Key Vaults
---

## List all Key Vaults in the subscription

```bash
az keyvault list
az keyvault list -o table
az keyvault list --resource-group $RESOURCE_GROUP --query "[].{Name:name, URI:properties.vaultUri, Public:properties.publicNetworkAccess, 
RBAC_Off:properties.enableRbacAuthorization, PolicyCount:length(properties.accessPolicies || '[]')}" --output table
```

**Using a token**

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.KeyVault/vaults?api-version=2023-07-01" \
  --headers "Authorization=Bearer $ARM_TOKEN" \
  --query "value[].name" -o tsv
```

## Check access policies (who has what permissions)

```bash
az keyvault show --name $KEYVAULT_NAME --query "properties.accessPolicies[].{OID:objectId, Permissions:permissions.secrets}"
```

**Using a token**

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.KeyVault/vaults/$VAULT_NAME?api-version=2023-07-01" \
  --headers "Authorization=Bearer $ARM_TOKEN" \
  --query "properties.accessPolicies[].{OID:objectId, Permissions:permissions.secrets}"
```

## List Key Vaults in a specific Resource Group

```bash
az keyvault list --resource-group $RESOURCE_GROUP
```

**Using a token**

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.KeyVault/vaults?api-version=2023-07-01" \
  --headers "Authorization=Bearer $ARM_TOKEN" \
  --query "value[].name" -o tsv
```

## Show details of a specific Key Vault

```bash
az keyvault show --name $VAULT_NAME
```

**Using a token**

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.KeyVault/vaults/$VAULT_NAME?api-version=2023-07-01" \
  --headers "Authorization=Bearer $ARM_TOKEN"
```

## List all keys in a Key Vault

```bash
az keyvault key list --vault-name $KEYVAULT_NAME
```

**Using a token**

```bash
az rest --method GET \
  --url "https://$VAULT_NAME.vault.azure.net/keys?api-version=7.4" \
  --headers "Authorization=Bearer $VAULT_TOKEN" \
  --query "value[].kid" -o tsv
```

## List all secrets in a Key Vault

```bash
az keyvault secret list --vault-name $KEYVAULT_NAME
```

**Using a token**

```bash
az rest --method GET \
  --url "https://$VAULT_NAME.vault.azure.net/secrets?api-version=7.4" \
  --headers "Authorization=Bearer $VAULT_TOKEN" \
  --query "value[].id" -o tsv
```

## List all certificates in a Key Vault

```bash
az keyvault certificate list --vault-name $KEYVAULT_NAME
```

**Using a token**

```bash
az rest --method GET \
  --url "https://$VAULT_NAME.vault.azure.net/certificates?api-version=7.4" \
  --headers "Authorization=Bearer $VAULT_TOKEN" \
  --query "value[].id" -o tsv
```

## List all deleted Key Vaults in the subscription

```bash
az keyvault list-deleted
```

**Using a token**

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.KeyVault/deletedVaults?api-version=2023-07-01" \
  --headers "Authorization=Bearer $ARM_TOKEN" \
  --query "value[].name" -o tsv
```

## Get properties of a deleted Key Vault

```bash
az keyvault show-deleted --name $KEYVAULT_NAME
```

**Using a token**

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.KeyVault/locations/$LOCATION/deletedVaults/$VAULT_NAME?api-version=2023-07-01" \
  --headers "Authorization=Bearer $ARM_TOKEN"
```

## Get assigned roles

```bash
az role assignment list --include-inherited --scope "/subscriptions/<subscription-uuid>/resourceGroups/$RESOURCE_GROUP$/providers/Microsoft.KeyVault/vaults/$KEYVAULT_NAME"
```

**Using a token**

```bash
az rest --method GET \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.KeyVault/vaults/$VAULT_NAME/providers/Microsoft.Authorization/roleAssignments?api-version=2022-04-01&\$filter=atScope()" \
  --headers "Authorization=Bearer $ARM_TOKEN"
```

## List deleted secrets

```bash
az keyvault secret list-deleted --vault-name $KEYVAULT_NAME
```

**Using a token**

```bash
az rest --method GET \
  --url "https://$VAULT_NAME.vault.azure.net/deletedsecrets?api-version=7.4" \
  --headers "Authorization=Bearer $VAULT_TOKEN" \
  --query "value[].id" -o tsv
```

