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
./configure --prefix=/home/替换成你的用户名/local CFLAGS="-I/home/替换成你的用户名/local/include -I/home/替换成你的用户名/local/include/ncurses" LDFLAGS="-L/home/替换成你的用户名/local/lib -L/home/替换成你的用户名/local/include/ncurses -L/home/替换成你的用户名/local/include"
make && make install
```

如果没什么错误并且在当前目录下生成了 `tmux` 就成功构建了 将 `tmux` 文件复制出去使用即可

`configure` 可能会出现以下错误 原因缺少词法分析器 就我们去手动编译安装一下词法分析器

```
checking for yacc... no
configure: error: "yacc not found"
```

下载解压

```shell
wget https://ftp.gnu.org/gnu/bison/bison-3.2.1.tar.gz
tar -xzvf bison-3.8.tar.gz
```

进入目录配置编译安装一下

```shell
cd bison-3.8.tar.gz
./configure --prefix=/home/替换成你的用户名/local
make && make install
```

配置 `bison` 时可能会出现以下报错 原因是系统未安装 `M4` 可以去手动编译安装一下

```
checking for GNU M4 that supports accurate traces... configure: error: no acceptable m4 could be found in $PATH.
GNU M4 1.4.6 or later is required; 1.4.16 or newer is recommended.
GNU M4 1.4.15 uses a buggy replacement strstr on some systems.
Glibc 2.9 - 2.12 and GNU M4 1.4.11 - 1.4.15 have another strstr bug.
```

```shell
wget https://mirrors.kernel.org/gnu/m4/m4-1.4.19.tar.gz
tar -xzvf m4-1.4.19.tar.gz
```

继续进入目录配置编译安装一下

```shell
cd bison-3.8.tar.gz
./configure --prefix=/home/替换成你的用户名/local
make && make install
```

安装完成返回 `bison` 目录进行编译安装

### 配置用户运行目录

如果还是提示未找到 `M4` 可以配置一下用户的 `bin` 目录

```shell
vim ~/.bashrc
export PATH=$PATH:/home/替换成你的用户名/local/bin
source ~/.bashrc
```

然后再次执行配置编译 `bison` 然后到 `tmux` 目录执行编译安装

如果没有配置用户运行目录则需要将编译出来的 `tmux` 复制出去使用或者使用路径全称使用 `/home/替换成你的用户名/local/bin/tmux`





