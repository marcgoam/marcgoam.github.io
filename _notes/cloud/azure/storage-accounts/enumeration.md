---
title: "Storage Accounts Enumeration"
order: 51
description: Enumeration of Storage Accounts
---

## Get Storage accounts

```bash
az storage account list
```

```bash
az storage container list --account-name $STORAGE_ACCOUNT --auth-mode login
```

## Blob Storage

### List containers

```bash
az storage container list --account-name $STORAGE_ACCOUNT
```

### Check if public access is allowed

```bash
az storage container show-permission \
   --account-name $STORAGE_ACCOUNT \
   -n $CONTAINER_NAME
```

### Make a container public

```bash
az storage container set-permission \
--public-access container \
--account-name $STORAGE_ACCOUNT \
-n <container-name>
```

### List blobs in a container

```bash
az storage blob list \
   --container-name <container name> \
   --account-name $STORAGE_ACCOUNT
```

### Enum blobs with permissions

```bash
az storage blob list \
   --container-name <container name> \
   --account-name $STORAGE_ACCOUNT \
   --auth-mode login
```

### Download blob

```bash
az storage blob download \
   --account-name $STORAGE_ACCOUNT \
   --container-name <container name> \
   --name <blob name> \
   --file </path/to/local/file>
```

### Create container policy

```bash
az storage container policy create \
  --account-name $STORAGE_ACCOUNT \
  --container-name mycontainer \
  --name fullaccesspolicy \
  --permissions racwdl \
  --start 2023-11-22T00:00Z \
  --expiry 2024-11-22T00:00Z
```

## Access Keys

### Enumerate Access Keys

```bash
az storage account keys list --account-name $STORAGE_ACCOUNT
```
### Check key policies

```bash
az storage account show -n <name> --query "{KeyPolicy:keyPolicy}"
```

Once having the key, it's possible to use it with the argument --account-key

### Enum containers with account key

```bash
az storage container list \
   --account-name $STORAGE_ACCOUNT \
   --account-key "ZrF40pkVKvWPUr[...]v7LZw=="
```

### Enum blobs with account key

```bash
az storage blob list \
   --container-name <container name> \
   --account-name $STORAGE_ACCOUNT \
   --account-key "ZrF40pkVKvWPUr[...]v7LZw=="
```

### Download a file using an account key

```bash
az storage blob download \
  --account-name <account name> \
  --account-key "ZrF40pkVKvWPUr[...]v7LZw==" \
  --container-name <container name> \
  --name <blob name> \
  --file </path/to/local/file>
```

### Upload a file using an account key

```bash
az storage blob upload \
  --account-name <account name> \
  --account-key "ZrF40pkVKvWPUr[...]v7LZw==" \
  --container-name <container name> \
  --file </path/to/local/file>
```

## SAS
### List access policies

```bash
az storage <container|queue|share|table> policy list \
  --account-name $STORAGE_ACCOUNT \
  --container-name <container name>
```
### Generate SAS with all permissions using an access key

```bash
az storage <container|queue|share|table|blob> generate-sas \
  --permissions acdefilmrtwxy \
  --expiry 2024-12-31T23:59:00Z \
  --account-name $STORAGE_ACCOUNT \
  -n <container-name>
```
### Generate SAS with all permissions using via user delegation

```bash
az storage <container|queue|share|table|blob> generate-sas \
  --permissions acdefilmrtwxy \
  --expiry 2024-12-31T23:59:00Z \
  --account-name $STORAGE_ACCOUNT \
  --as-user --auth-mode login \
  -n <container-name>
```

### Generate account SAS

```bash
az storage account generate-sas \
--expiry 2024-12-31T23:59:00Z \
--account-name $STORAGE_ACCOUNT \
--services qt \
--resource-types sco \
--permissions acdfilrtuwxy
```

### Use the returned SAS key with the param --sas-token

```bash
az storage blob show \
  --account-name $STORAGE_ACCOUNT \
  --container-name <container name> \
  --sas-token 'se=2024-12-31T23%3A59%3A00Z&sp=racwdxyltfmei&sv=2022-11-02&sr=c&sig=ym%2Bu%2BQp5qqrPotIK5/rrm7EMMxZRwF/hMWLfK1VWy6E%3D' \
  --name 'asd.txt'
```

## Enum users

### List users

```bash
az storage account local-user list \
  --account-name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP
```

### Get user

```bash
az storage account local-user show \
  --account-name <storage-account-name> \
  --resource-group <resource-group-name> \
  --name <local-user-name>
```

### List keys

```bash
az storage account local-user list \
  --account-name <storage-account-name> \
  --resource-group <resource-group-name>
```



