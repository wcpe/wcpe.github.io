---
title: 群辉使用 fclones 进行文件去重
author: WCPE
date: 2024-07-09 16:17:54
categories:
- 记录
tags:
- '群晖'
- 'Synology'
---

## 前言

由于我的群辉 NAS 中的文件数量过于庞大，已经积累了许多重复的文件，我曾经尝试着手动去清理重复文件，刚开始还好，后面直接放弃了因为太浪费时间了，所以我开始寻找一款查询文件是否重复的软件

原先我在 Windows 中使用的查询重复的软件是 `AllDup` 现在他好像并不好使因为群辉系统是 `Linux` 所以我无法直接使用它，还得曲线救国以下将 Nas 中的文件使用 SMB 映射至 Windos 中使用 `AllDup` 进行扫描删除等操作 这速度十分的慢，所以我在网络上寻找实用的软件

### [fclones](https://github.com/pkolaczk/fclones)

> Efficient duplicate file finder and remover

发现他编译了 `Linux` `x86` 的版本，我就放到群辉上试了一下发现居然可以用

直接将编译好的 `fclones` 文件丢入群辉 `/bin` 目录下就可以直接使用了

几个简单的命令如下

```bash
fclones group . >dupes.txt
```

搜索当前路径

```bash
fclones group ./1 ./2 >dupes.txt
```

搜索 ./1 ./2 路径的重复文件 放入 dupes.txt

```bash
fclones link <dupes.txt
```

使用 dupes.txt 重复文件 将他们硬链接

```bash
fclones link --soft <dupes.txt
```

使用 dupes.txt 重复文件 将他们软链接

```bash
fclones remove --path '/path/**'  <dupes.txt
```

指定删除路径为 '/path/\*\*' 的

```bash
fclones link --soft --path '/path/**'  <dupes.txt
```

指定软链接路径为 '/path/\*\*'
