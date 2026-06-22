---
title: "Azure Service Bus Explotation"
order: 151
description: Explotation of Service Bus
---

## Get keys

> The following permission is needed
> - **Microsoft.ServiceBus/namespaces/authorizationrules/listKeys/action**

```bash
az servicebus namespace authorization-rule keys list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --authorization-rule-name RootManageSharedAccessKey [--authorization-rule-name RootManageSharedAccessKey]
```

## Renew keys

> The following permission is needed
> - **Microsoft.ServiceBus/namespaces/authorizationrules/regenerateKeys/action**

```bash
az servicebus namespace authorization-rule keys renew \
  --key PrimaryKey \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $BUS_NAME \
  --authorization-rule-name RootManageSharedAccessKey
```

## Script to read messages from topic

```bash
import asyncio
from azure.servicebus.aio import ServiceBusClient

CONN_STR = "Endpoint=sb://<namespace>.servicebus.windows.net/;SharedAccessKeyName=<rule>;SharedAccessKey=<key>="
TOPIC        = "<topic-name>"
SUBSCRIPTION = "<subscription-name>"

async def run():
    async with ServiceBusClient.from_connection_string(CONN_STR) as client:
        async with client.get_subscription_receiver(
            topic_name=TOPIC,
            subscription_name=SUBSCRIPTION,
            max_wait_time=5
        ) as receiver:
            async for msg in receiver:
                print(f"Body: {str(msg)}")
                print(f"Props: {msg.application_properties}")
                await receiver.complete_message(msg)  # removes from queue

asyncio.run(run())
```

## Create new authorization rule

> The permission needed is:
> - **Microsoft.ServiceBus/namespaces/AuthorizationRules/write**
>
> With this permission it’s possible to create a new authorization rule with all permissions and its own keys.

```bash
az servicebus namespace authorization-rule create \
  --authorization-rule-name "myRule" \
  --namespace-name $BUS_NAME \
  --resource-group $RESOURCE_GROUP \
  --rights Manage Listen Send
```