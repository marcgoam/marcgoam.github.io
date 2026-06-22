---
title: "IAM & RBAC Enumeration"
order: 21
description: Enumeration of IAM & RBAC
---

## List Role Assignments

**Roles assigned to current SP/user**

```bash
az role assignment list --assignee "$CLIENT_ID" --all --output table
```

**Inspect a custom role definition**

```bash
az role definition list --name "<role-name>" \
  --query "[].permissions[0].{actions:actions, dataActions:dataActions}"
```