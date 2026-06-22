---
title: "Azure MySQL Explotation"
order: 76
description: Explotation of Azure MySQL
---

## Log in to MySQL

```bash
mysql -u <user> -p'<password>' \
  -h $SERVER_NAME.mysql.database.azure.com

mycli -u <user> -p'<password>' \
  -h $SERVER_NAME.mysql.database.azure.com
```