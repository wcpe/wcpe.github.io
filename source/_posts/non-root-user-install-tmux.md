---
title: 非 root 用户安装 Tmux 终端
author: WCPE
date: 2025-01-11 03:08:13
categories:
  - 记录
  - Linux
tags:
  - 'Linux'
  - 'Tmux'
---

Tmux 终端是一个很好用的终端复用器 他可以启动一系列的会话 窗口 窗格 每个都是独立的存在

## 安装

`root` 用户安装仅需一行

```shell
sodu apt-get install tmux
```

非 `root` 用户就不一样了 需要下载源码手动编译

### 下载

下载tmux及其依赖软件。

```shell
wget -c https://github.com/tmux/tmux/releases/download/3.5a/tmux-3.5a.tar.gz
wget -c https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz
wget -c https://ftp.gnu.org/gnu/ncurses/ncurses-6.4.tar.gz
```

### 解压

```shell
tar -xzvf tmux-3.5a.tar.gz
tar -xzvf libevent-2.1.12-stable.tar.gz
tar -xzvf ncurses-6.4.tar.gz
```

### 构建依赖

先安装 `libevent-2.1.12-stable` 依赖

```shell
cd libevent-2.1.12-stable
```


```shell
./configure --prefix=/home/替换成你的用户名/local --disable-shared
make && make install
```

接着退出当前目录

```shell
cd ..
```

再安装 `ncurses-6.4` 依赖

```shell
cd ncurses-6.4
```

```shell
./configure --prefix=/home/替换成你的用户名/local
make && make install
```

退出当前目录 依赖安装完成

```shell
cd ..
```

### 构建本体

```shell
cd tmux-3.5a
./configure CFLAGS="-I/home/替换成你的用户名/local/include -I/home/替换成你的用户名/local/include/ncurses" LDFLAGS="-L/home/替换成你的用户名/local/lib -L/home/替换成你的用户名/local/include/ncurses -L/home/替换成你的用户名/local/include"
make
```
没什么错误并且在当前目录下生成了 tmux 就成功构建了 将 tmux 文件复制出去使用即可