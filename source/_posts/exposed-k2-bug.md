---
title: Exposed DSL 中 deleteWhere 无法识别 eq 操作符
author: WCPE
date: 2025-01-14 15:34:37
categories:
  - 记录
tags:
  - 'Exposed'
  - 'Java'
---

### 起因

代码中使用了 Exposed DSL 中的 deleteWhere 方法，但是在使用 eq 操作符时，编译器提示找不到 eq 方法。

> Unresolved reference 'eq'.

```kotlin
transaction {
    PetSoulInstanceTable.deleteWhere {
        PetSoulInstanceTable.identifier eq identifier
    }
}
```

只需要手动导入

```java
import org.jetbrains.exposed.sql.SqlExpressionBuilder.eq;
```
