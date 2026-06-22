---
title: "Azure Service Bus Enumeration"
order: 151
description: Enumeration of Service Bus
---

## Namespace Enumeration

```bash
az servicebus namespace list -o table --query "[].{name:name,publicNetworkAccess:publicNetworkAccess,serviceBusEndpoint:serviceBusEndpoint,disableLocalAuth:disableLocalAuth,resourceGroup:resourceGroup}"
```

```bash
az servicebus namespace network-rule-set list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME
```

```bash
az servicebus namespace show --resource-group $RESOURCE_GROUP --name $BUS_NAME
```

```bash
az servicebus namespace private-endpoint-connection list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME
```

```bash
az servicebus namespace exists --name $BUS_NAME
```

## Authorization Rule Enumeration

```bash
az servicebus namespace authorization-rule list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME
az servicebus namespace authorization-rule list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --query "[].{name:name,rights:rights}"
```

```bash
az servicebus queue authorization-rule list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --queue-name <MyQueue>
```

```bash
az servicebus topic authorization-rule list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --topic-name $TOPIC_NAME
```

## Get keys

```bash
az servicebus namespace authorization-rule keys list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME [--authorization-rule-name RootManageSharedAccessKey]
```

```bash
az servicebus topic authorization-rule keys list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --topic-name $TOPIC_NAME --name <auth-rule-name>

```

```bash
az servicebus queue authorization-rule keys list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --queue-name $TOPIC_NAME --name <auth-rule-name>
```

## Queue Enumeration

```bash
az servicebus queue list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME 
```

```bash
az servicebus queue show --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --name $TOPIC_NAME
```

## Topic Enumeration

```bash
az servicebus topic list \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $BUS_NAME \
  --query "[].{TopicName:name, Status:status, ActiveMsgs:countDetails.activeMessageCount, DeadLetterMsgs:countDetails.deadLetterMessageCount, Partitioning:enablePartitioning}" \
  --output table
```

```bash
az servicebus topic show --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --name $TOPIC_NAME
```

## Subscription Enumeration

```bash
az servicebus topic subscription list --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --topic-name $TOPIC_NAME
```

```bash
az servicebus topic subscription list \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $BUS_NAME \
  --topic-name labtopic \
  --query "[].{Subscription:name, Status:status, ActiveCount:countDetails.activeMessageCount, DeadLetterCount:countDetails.deadLetterMessageCount, RequiresSession:requiresSession}" \
  --output table
```

```bash
az servicebus topic subscription show --resource-group $RESOURCE_GROUP --namespace-name $BUS_NAME --topic-name $TOPIC_NAME --name <MySubscription>
```
