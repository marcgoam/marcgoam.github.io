---
title: "Azure Queues Enumeration"
order: 141
description: Enumeration of Azure Queues
---

## Get storage accounts

```bash
az storage account list
```

```bash
az storage account list --query "[].{Name:name, ResourceGroup:resourceGroup, PublicBlob:allowBlobPublicAccess, SharedKey:allowSharedKeyAccess, NetDefaultAction:networkRuleSet.defaultAction, PublicNetwork:publicNetworkAccess, MinTLS:minimumTlsVersion, OAuthOnly:defaultToOAuthAuthentication}" -o table
```

## Get queues in storage account

```bash
az storage queue list --account-name $STORAGE_ACCOUNT_NAME [--auth-mode login]
```

## Queue Metadata

```bash
az storage queue metadata show --name $QUEUE_NAME --account-name $STORAGE_ACCOUNT_NAME
```

## Get Policy 

```bash
az storage queue policy list --queue-name $QUEUE_NAME --account-name $STORAGE_ACCOUNT_NAME
```

## Get Messages

```bash
az storage message get --queue-name $QUEUE_NAME --account-name $STORAGE_ACCOUNT_NAME
```

## Peek Messages (doesn’t affect msgs visibility)

```bash
az storage message peek --queue-name $QUEUE_NAME --account-name <$STORAGE_ACCOUNT_NAME
```
