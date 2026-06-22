---
title: "File Shares Enumeration"
order: 61
description: Enumeration of File Shares
---

## Azure File Share

### List file shares

```bash
az storage share list --account-name $STORAGE_ACCOUNT
```

To see the deleted ones too --include-deleted

```bash
az storage share-rm list \
  --storage-account filesharelab59874cdaa53 \
  --resource-group file_share-labs-rg-5 \
  --include-deleted
```

### Get network rules

```bash
az storage account list \
  --query "[?name=='share'].networkRuleSet"
```

### Get dirs/files inside the share

```bash
az storage file list --account-name $STORAGE_ACCOUNT --share-name <share-name>
```

If type is "dir", you can continue enumerating files inside of it

```bash
az storage file list --account-name $STORAGE_ACCOUNT --share-name <prev_dir/share-name>
```

### Download a complete share (with directories and files inside of them)

```bash
az storage file download-batch -d . --source <share-name> --account-name <name>
```

### Get snapshots/backups

```bash
az storage share snapshot --name <share-name>
az storage share list --account-name <name> --include-snapshots --query "[?snapshot != null]"
```

### List contents of a snapshot/backup

```bash
az storage file list --account-name <name> --share-name <share-name> --snapshot <snapshot-version> #e.g. "2024-11-25T11:26:59.0000000Z"
```

### Download snapshot/backup

```bash
az storage file download-batch -d . --account-name <name> --source <share-name> --snapshot <snapshot-version>
```

## Azure Table Storage

### List tables

```bash
az storage table list --account-name <name>
```

### Read Table

```bash
az storage entity query \
   --acount-name <name> \
   --table-name <t-name> \
   --num-results 10
```