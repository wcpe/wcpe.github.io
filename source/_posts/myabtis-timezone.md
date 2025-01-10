---
title: MySQL 时区导致 Mybatis 插入 DateTime 的问题
author: WCPE
date: 2024-08-10 14:10:26
categories:
- 坑
tags:
- 'Mybatis'
- 'MySQL'
---
# 前言

使用 Mybatis 插入 Date 发现数据库中的日期与 Mybatis 查询出来的日期不一致


# 血案现场

查询表字段值：
```
mysql> SELECT * FROM `test`.`global_variables`;
+--------------+----------------+---------------------+---------------------+---------------------+
| variable_key | variable_value | created_at          | updated_at          | expires_at          |
+--------------+----------------+---------------------+---------------------+---------------------+
| a            | b              | 2024-08-10 13:19:52 | 2024-08-10 13:19:57 | 2024-12-31 17:01:01 |
+--------------+----------------+---------------------+---------------------+---------------------+
1 row in set (0.02 sec)
```


查询数据库中的时区变量

```
mysql> show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone |        |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.02 sec)
```

使用 Mybatis 查询出来的数据很奇怪

```
[13:31:58 INFO]: result: GlobalVariable(variableKey=a, variableValue=b, createdAt=2024-08-10 21:19:52.0, updatedAt=2024-08-10 21:19:57.0, expiresAt=2025-01-01 01:01:01.0)
```




# 问题根源

以下引用自通义千问
> UTC（Coordinated Universal Time）并不是一个特定的时区，而是一种世界通用的时间标准。它是基于原子时秒长的时间计量系统，在时刻上尽量接近于世界时（基于地球自转的时间计量系统）。UTC是为了克服世界时的不稳定性而制定的一种折衷方案。
>
> UTC 并不表示任何一个具体的时区偏移量，但它被用来定义世界各地的时区。例如，北京时间（UTC+8）表示比UTC时间晚8个小时。UTC本身没有时区偏移，可以认为是0时区的时间，即UTC+0或UTC-0。
>
> 简而言之，UTC是一个国际时间标准，用于定义全球时间同步的基础，而不是一个具体的时区。


# 解决方法

我在 MySQL 连接的 url 参数为 

> useSSL=false&characterEncoding=UTF-8&serverTimezone=**UTC**

尤其注意 serverTimezone=UTC 这配置的时区有问题 UTC 不是一个时区因改为当前所在时区 Asia/Shanghai


改为

> useSSL=false&characterEncoding=UTF-8&serverTimezone=**Asia/Shanghai**

即可解决这个问题
