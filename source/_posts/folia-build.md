---
title: 构建 Folia
date: 2024-12-01 18:57:10
tags:
---

准备构建 https://github.com/PaperMC/Folia 这个项目的 1.21.1 版本

clone 项目下来知道 gradle 换到 jdk21 即可同步项目

然后根据文档给出的构建方式
```
./gradlew applyPatches
```

```
./gradlew createMojmapPaperclipJar
```
直接报错

```
> Task :paper:patchCraftBukkit FAILED
Execution failed for task ':paper:patchCraftBukkit'.
> io.papermc.paperweight.PaperweightException: Command finished with 128 exit code: git -c commit.gpgsign=false -c core.safecrlf=false clone --no-hardlinks C:\XXX\Folia\.gradle\caches\paperweight\upstreams\paper\work\CraftBukkit C:\XXX\Folia\.gradle\caches\paperweight\upstreams\paper\.gradle\caches\paperweight\taskCache\patchCraftBukkit.repo

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.
> Get more help at https://help.gradle.org.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.
You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.
For more on this, please refer to https://docs.gradle.org/8.8/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation. 为什么
BUILD FAILED in 14s
2 actionable tasks: 2 executed
```

尝试了开全魔法环境, 重装 Git 等操作都没有用

于是我就把命令复制出来手动运行一遍 结果


git 最大只可以创建 4096 长度的文件名 然而在 Windows 只有 260 得改配置

```
git config --global core.longpaths true
```

直接再次运行 补丁成功

```
19:03:33: Executing 'applyPatches'…


> Configure project :
paperweight-patcher v1.7.1 (running on 'Windows 11')

> Task :clonePaperRepo

> Configure project :paper
paperweight-core v1.7.1 (running on 'Windows 11')

> Task :paper:initSubmodules
> Task :paper:patchSpigotApiPatches UP-TO-DATE
> Task :paper:patchSpigotApi UP-TO-DATE
> Task :paper:applyApiPatches UP-TO-DATE
> Task :paper:downloadMcManifest
> Task :paper:downloadMcVersionManifest UP-TO-DATE
> Task :paper:downloadServerJar UP-TO-DATE
> Task :paper:extractFromBundler UP-TO-DATE
> Task :paper:addAdditionalSpigotMappings UP-TO-DATE
> Task :paper:downloadMappings UP-TO-DATE
> Task :paper:filterVanillaJar UP-TO-DATE
> Task :paper:generateMappings UP-TO-DATE
> Task :paper:generateSpigotMappings UP-TO-DATE
> Task :paper:spigotRemapJar UP-TO-DATE
> Task :paper:cleanupMappings UP-TO-DATE
> Task :paper:patchMappings UP-TO-DATE
> Task :paper:cleanupSourceMappings UP-TO-DATE
> Task :paper:remapJar UP-TO-DATE
> Task :paper:fixJar UP-TO-DATE
> Task :paper:patchCraftBukkitPatches UP-TO-DATE
> Task :paper:filterSpigotExcludes UP-TO-DATE
> Task :paper:spigotDecompileJar UP-TO-DATE
> Task :paper:patchCraftBukkit UP-TO-DATE
> Task :paper:patchSpigotServerPatches UP-TO-DATE
> Task :paper:patchSpigotServer UP-TO-DATE
> Task :paper:patchSpigot UP-TO-DATE
> Task :paper:downloadSpigotDependencies UP-TO-DATE
> Task :paper:collectAtsFromPatches UP-TO-DATE
> Task :paper:mergePaperAts UP-TO-DATE
> Task :paper:remapSpigotSources UP-TO-DATE
> Task :paper:remapGeneratedAt UP-TO-DATE
> Task :paper:remapSpigotAt UP-TO-DATE
> Task :paper:mergeGeneratedAts UP-TO-DATE
> Task :paper:mergeAdditionalAts UP-TO-DATE
> Task :paper:applyMergedAt UP-TO-DATE
> Task :paper:copyResources UP-TO-DATE
> Task :paper:decompileJar UP-TO-DATE
> Task :paper:downloadMcLibrariesSources UP-TO-DATE
> Task :paper:applyServerPatches UP-TO-DATE
> Task :paper:applyPatches UP-TO-DATE
> Task :paper:lineMapJar UP-TO-DATE
> Task :paper:prepareForDownstream
> Task :getPaperUpstreamData

> Task :applyApiPatches
Creating Folia-API from patch source...
Applying patches to Folia-API...
5 patches applied cleanly to Folia-API

> Task :applyGeneratedApiPatches
Creating generated from patch source...
Applying patches to generated...
No patches found

> Task :applyServerPatches
Creating Folia-Server from patch source...
Importing 23 classes from vanilla...
Importing 0 data files from vanilla...
Importing 0 classes from library sources...
Applying patches to Folia-Server...
18 patches applied cleanly to Folia-Server

> Task :applyPatches

BUILD SUCCESSFUL in 1m 10s
5 actionable tasks: 5 executed
19:04:44: Execution finished 'applyPatches'.
```

直接运行 `createMojmapPaperclipJar` 开始构建 Jar 包