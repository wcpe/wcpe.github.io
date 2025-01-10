---
title: Gradle 插件测试
author: WCPE
date: 2025-01-06 14:10:21
categories:
- 记录
tags:
- 'Gradle'
---



在项目的 `settings.gradle.kts` 可以直接引入插件发布的 `Maven` 仓库

```kotlin
pluginManagement {
    repositories {
        mavenLocal()
        maven("https://maven.wcpe.top/repository/maven-public/")
        gradlePluginPortal()
    }
}
```

在 `build.gradle.kts` 中可以直接引用插件

```kotlin
plugins {
    id("top.wcpe.corebridge") version "1.0.0"
}
```
