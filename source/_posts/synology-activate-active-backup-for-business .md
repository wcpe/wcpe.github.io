---
title: 黑群晖激活 Active Backup for Business
date: 2024-08-18 02:45:53
tags:
---

两条命令

```
http://群晖地址:5000/webapi/auth.cgi?api=SYNO.API.Auth&version=3&method=login&account=管理员用户名&passwd=密码&format=cookie
```


```
http://群晖地址:5000/webapi/entry.cgi?api=SYNO.ActiveBackup.Activation&method=set&version=1&activated=true&serial_number="SN序列号"
```