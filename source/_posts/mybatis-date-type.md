---
title: Mybatis 处理 MySQL 中 date 类型字段
date: 2024-08-09 15:40:27
tags:
---



# 前言

之前使用 Mybatis 都是直接存储的 `bigint` 时间戳即可主要为了避免时区问题和各种类型不兼容的问题，但是缺点也很明显不能直观的显示在数据库中，所以今天尝试使用 `Mybatis` 插入 Date 数据


在 sql 中有两个类 java.sql.Timestamp 和 java.sql.Date 我们直接使用 Timestamp 指定在 javaType 中

```kotlin
    @Select("SELECT * FROM `global_variables` WHERE variable_key = #{variableKey}")
    @Results(
        id = "GlobalVariableResult", value = [
            Result(column = "variable_key", property = "variableKey"),
            Result(column = "variable_value", property = "variableValue"),
            Result(column = "created_at", property = "createdAt", javaType = Timestamp::class),
            Result(column = "updated_at", property = "updatedAt", javaType = Timestamp::class),
            Result(column = "expires_at", property = "expiresAt", javaType = Timestamp::class)
        ]
    )
    fun getGlobalVariable(@Param("variableKey") variableKey: String): GlobalVariable?
```