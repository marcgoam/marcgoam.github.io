---
title: "Entra ID Enumeration"
order: 11
description: Enumeration of Entra ID 
---

## Get current user Object ID

```bash
USER_ID=$(az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/users/$EMAIL" --query "id" -o tsv)
```

## Get all Entra ID role assignments

```bash
ROLE_ASSIGNMENTS=$(az rest --method GET \
  --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments")
```

```bash
az rest --method get --query "value[].{RoleID:roleDefinitionId, Scope:directoryScopeId}" -o table --url "https://graph.microsoft.com/v1.0/roleManagement/directory/roleAssignments?\$filter=principalId eq '$MY_OID'"
```

**List only custom roles**

```bash
az rest --method GET \
  --uri "https://graph.microsoft.com/v1.0/roleManagement/directory/roleDefinitions" | jq '.value[] | select(.isBuiltIn == false)'
```

## Filter assignments for current user

```bash
echo "$ROLE_ASSIGNMENTS" | jq -r --arg uid "$USER_ID" \
  '.value[] | select(.principalId==$uid)'
```

## Users Enumeration

**Get users**

```bash
az ad user list --output table
az ad user list --query "[].userPrincipalName"
```

**Get info of 1 user**

```bash
az ad user show --id "test@corp.onmicrosoft.com"
```

**Search "admin" users**

```bash
az ad user list --query "[].displayName" | findstr /i "admin"
az ad user list --query "[?contains(displayName,'admin')].displayName"
```

**Search attributes containing the word "password"**

```bash
az ad user list | findstr /i "password" | findstr /v "null,"
```

**All users from Entra ID**

```bash
az ad user list --query "[].{osi:onPremisesSecurityIdentifier,upn:userPrincipalName}[?osi==null]"
az ad user list --query "[?onPremisesSecurityIdentifier==null].displayName"
```

**All users synced from on-prem**

```bash
az ad user list --query "[].{osi:onPremisesSecurityIdentifier,upn:userPrincipalName}[?osi!=null]"
az ad user list --query "[?onPremisesSecurityIdentifier!=null].displayName"
```

**Get groups where the user is a member**

```bash
az ad user get-member-groups --id <email>
```

**Get roles assigned to the user in Azure (NOT in Entra ID)**

```bash
az role assignment list --include-inherited --include-groups --include-classic-administrators true --assignee <email>
```

**Get ALL roles assigned in Azure in the current subscription (NOT in Entra ID)**

```bash
az role assignment list --include-inherited --include-groups --include-classic-administrators true --all
```

### Get EntraID roles assigned to a user

**Get Token**

```bash
export TOKEN=$(az account get-access-token --resource https://graph.microsoft.com/ --query accessToken -o tsv)
```

**Get users**


```bash
curl -X GET "https://graph.microsoft.com/v1.0/users" \
    -H "Authorization: Bearer $TOKEN" \ -H "Content-Type: application/json" | jq
```

**Get EntraID roles assigned to an user**

```bash
curl -X GET "https://graph.microsoft.com/beta/rolemanagement/directory/transitiveRoleAssignments?\$count=true&\$filter=principalId%20eq%20'86b10631-ff01-4e73-a031-29e505565caa'" \
-H "Authorization: Bearer $TOKEN" \
-H "ConsistencyLevel: eventual" \
-H "Content-Type: application/json" | jq
```

**Get role details**

```bash
curl -X GET "https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitions/cf1c38e5-3621-4004-a7cb-879624dced7c" \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" | jq
```

## Groups

**Enumerate groups**

```bash
az ad group list
az ad group list --query "[].[displayName]" -o table
```

**Get info of 1 group**

```bash
az ad group show --group <group>
```

**Get "admin" groups**

```bash
az ad group list --query "[].displayName" | findstr /i "admin"
az ad group list --query "[?contains(displayName,'admin')].displayName"
```

**All groups from Entra ID**

```bash
az ad group list --query "[].{osi:onPremisesSecurityIdentifier,displayName:displayName,description:description}[?osi==null]"
az ad group list --query "[?onPremisesSecurityIdentifier==null].displayName"
```

**All groups synced from on-prem**

```bash
az ad group list --query "[].{osi:onPremisesSecurityIdentifier,displayName:displayName,description:description}[?osi!=null]"
az ad group list --query "[?onPremisesSecurityIdentifier!=null].displayName"
```

**Get members of group**

```bash
az ad group member list --group <group> --query "[].userPrincipalName" -o table
```
**Check if member of group**

```bash
az ad group member check --group "VM Admins" --member-id <id>
```

**Get which groups a group is member of**

```bash
az ad group get-member-groups -g "VM Admins"
```

**Get roles assigned to the group in Azure (NOT in Entra ID)**

```bash
az role assignment list --include-groups --include-classic-administrators true --assignee <group-id>
```

## Service Principals

**Get Service Principals**

```bash
az ad sp list --all
az ad sp list --all --query "[].[displayName,appId]" -o table
```

**Get details of one SP**

```bash
az ad sp show --id 00000000-0000-0000-0000-000000000000
```

**Search SP by string**

```bash
az ad sp list --all --query "[?contains(displayName,'app')].displayName"
```

**Get owner of service principal**

```bash
az ad sp owner list --id <id> --query "[].[displayName]" -o table
```

**Get service principals owned by the current user**

```bash
az ad sp list --show-mine
```

**Get SPs with generated secret or certificate**

```bash
az ad sp list --query '[?length(keyCredentials) > `0` || length(passwordCredentials) > `0`].[displayName, appId, keyCredentials, passwordCredentials]' -o json
```

## Applications

**List Apps**

```bash
az ad app list
az ad app list --query "[].[displayName,appId]" -o table
```

**Get info of 1 App**
```bash
az ad app show --id 00000000-0000-0000-0000-000000000000
```

**Search App by string**

```bash
az ad app list --query "[?contains(displayName,'app')].displayName"
```

**Get the owner of an application**

```bash
az ad app owner list --id <id> --query "[].[displayName]" -o table
```

**Get SPs owned by current user**

```bash
az ad app list --show-mine
```

**Get apps with generated secret or certificate**

```bash
az ad app list --query '[?length(keyCredentials) > `0` || length(passwordCredentials) > `0`].[displayName, appId, keyCredentials, passwordCredentials]' -o json
```

**Get Global Administrators (full access over apps)**

```bash
az rest --method GET --url "https://graph.microsoft.com/v1.0/directoryRoles/1b2256f9-46c1-4fc2-a125-5b2f51bb43b7/members"
```

**Get Application Administrators (full access over apps)**

```bash
az rest --method GET --url "https://graph.microsoft.com/v1.0/directoryRoles/1e92c3b7-2363-4826-93a6-7f7a5b53e7f9/members"
```

**Get Cloud Applications Administrators (full access over apps)**

```bash
az rest --method GET --url "https://graph.microsoft.com/v1.0/directoryRoles/0d601d27-7b9c-476f-8134-8e7cd6744f02/members"
```

### Get "API Permissions" of an App

**Get the ResourceAppId**

```bash
az ad app show --id "<app-id>" --query "requiredResourceAccess" --output json
```

```json
[
  {
    "resourceAccess": [
      {
        "id": "e1fe6dd8-ba31-4d61-89e7-88639da4683d",
        "type": "Scope"
      },
      {
        "id": "d07a8cc0-3d51-4b77-b3b0-32704d1f69fa",
        "type": "Role"
      }
    ],
    "resourceAppId": "00000003-0000-0000-c000-000000000000"
  }
]
```

**For the perms of type "Scope"**

```bash
az ad sp show --id <ResourceAppId> --query "oauth2PermissionScopes[?id=='<id>'].value" -o tsv
az ad sp show --id "00000003-0000-0000-c000-000000000000" --query "oauth2PermissionScopes[?id=='e1fe6dd8-ba31-4d61-89e7-88639da4683d'].value" -o tsv
```

**For the perms of type "Role"**

```bash
az ad sp show --id <ResourceAppId> --query "appRoles[?id=='<id>'].value" -o tsv
az ad sp show --id 00000003-0000-0000-c000-000000000000 --query "appRoles[?id=='d07a8cc0-3d51-4b77-b3b0-32704d1f69fa'].value" -o tsv
```

## Managed Identities

**List all manged identities**

```bash
az identity list --output table
```