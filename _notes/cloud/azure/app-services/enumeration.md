---
title: "Azure App Services Enumeration"
order: 91
description: Enumeration of Azure App Services
---

## List webapps

```bash
az webapp list
```

```bash
az webapp list --query "[].{Name:name,resourcegroup:resourceGroup, Host:defaultHostName, SCM:enabledHostNames[1], HTTPS_Only:httpsOnly, Runtime:siteConfig.linuxFxVersion || siteConfig.windowsFxVersion,State:state, Public_Access:publicNetworkAccess, Identity:identity.type, ClientCert:clientCertMode,CORS:cors,RemoteDebug:remoteDebuggingEnabled, WebSockets:webSocketsEnabled}" --output json
```

## Get instances of a webapp

```bash
az webapp list-instances --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP
```

## Get connection strings of a webapp

```bash
az webapp config connection-string list --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP
```

## Get appsettings of an app

```bash
az webapp config appsettings list --resource-group $RESOURCE_GROUP --name $WEBAPP_NAME
```

## Get backups of a webapp

```bash
az webapp config backup list --webapp-name $WEBAPP_NAME --resource-group $RESOURCE_GROUP
```

## Get snapshots

```bash
az webapp config snapshot list --resource-group $RESOURCE_GROUP -n $WEBAPP_NAME
```

## Get slots

```bash
az webapp deployment slot list --name <AppName> --resource-group $RESOURCE_GROUP --output table
az webapp show --slot <SlotName> --name <AppName> --resource-group $RESOURCE_GROUP
```

## Get traffic-routing

```bash
az webapp traffic-routing show --name <AppName> --resource-group $RESOURCE_GROUP
```

## Get storage account configurations of a webapp (contains access key)

```bash
az webapp config storage-account list --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP
```

## Get configured container (if any) in the webapp, it could contain credentials

```bash
az webapp config container show --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP
```

## Get Webjobs

```bash
az webapp webjob continuous list --resource-group $RESOURCE_GROUP --name <app-name>
az webapp webjob triggered list --resource-group $RESOURCE_GROUP --name <app-name>
```

## Retrieves the publishing profiles

```bash
az webapp deployment list-publishing-profiles \
  --name $WEBAPP_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "[].{Profile:profileName, Method:publishMethod, URL:publishUrl, User:userName, Password:userPWD,SQLServerDBConnectionString:SQLServerDBConnectionString,destinationAppUrl:destinationAppUrl}" \
  -o json
```