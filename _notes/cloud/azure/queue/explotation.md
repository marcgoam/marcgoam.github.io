---
title: "Azure Queues Explotation"
order: 141
description: Explotation of Azure Queues
---

## Read messages

> The permission needed is:
> - **DataActions: Microsoft.Storage/storageAccounts/queueServices/queues/messages/read**
> 
> An attacker with this permission can peek messages from an Azure Storage Queue. This allows the attacker to view the content of messages without marking them as processed or altering their state.

```bash
az storage message peek \
  --queue-name $QUEUE_NAME \
  --account-name $STORAGE_ACCOUNT \
  --auth-mode login \
  --num-messages 32 \
  --output json | jq -r '.[].content'
```