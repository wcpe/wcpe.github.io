---
title: Tmux 终端新版本错误的键映射
author: WCPE
date: 2025-01-11 16:29:40
categories:
  - 记录
  - Linux
tags:
  - 'Linux'
  - 'Tmux'
---

### 起因

在 Tmux 内使用 [Jline](https://github.com/jline/jline3) 的应用程序会出现按键错误映射

### 寻找办法

在 Tmux 的 issues 里寻找了许久 直到看到

> 3.3a on macOS - wrong key mapping [#3436](https://github.com/tmux/tmux/issues/3436)
> > macOS 上的 3.3a - 错误的键映射

其中提到了解决方案

### 解决方法

编辑文件 `~/.tmux.conf` 填入 `set -g default-terminal screen-256color` 保存即可

