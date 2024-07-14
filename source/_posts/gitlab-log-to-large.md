title: Docker 中 Gitlab 日志过大问题
author: WCPE
tags: []
categories:
- Gitlab
date: 2024-07-12 10:47:28
---

# 前言


突然发现 Gitlab 服务老挂很奇怪，于是进后台寻找问题，首先看一下 Gitlab 日志 里面有一段 Redis 本地化持久数据无法写入到磁盘，硬盘满了...... 


# 过程
使用 `df -h` 命令进行查询
```bash
[root@localhost ~]# df -h
文件系统             容量  已用  可用 已用% 挂载点
devtmpfs             4.0M     0  4.0M    0% /dev
tmpfs                 16G     0   16G    0% /dev/shm
tmpfs                6.3G   22M  6.2G    1% /run
/dev/mapper/rl-root   62G   47G   15G   76% /
/dev/nvme0n1p1       960M  531M  430M   56% /boot
/dev/mapper/rl-home   30G  664M   30G    3% /home
tmpfs                3.2G   52K  3.2G    1% /run/user/42
overlay               62G   47G   15G   76% /var/lib/docker/overlay2/3e003c0a9780a97f56039cca12c09458940b0d1f4b7b713598ca2bbd09a9eaf9/merged
overlay               62G   47G   15G   76% /var/lib/docker/overlay2/345b25c35b361220cee68f8c01c937b1afd8af528f6027489c1ad4b0ebada3ff/merged
overlay               62G   47G   15G   76% /var/lib/docker/overlay2/95b8fde65b7a6f9be728097f1b0c8d8230409e665f0cda3915709596eee0d8d1/merged
overlay               62G   47G   15G   76% /var/lib/docker/overlay2/272dbb01f0c892a48a77058f16736f184b256118e0c6434d4cb90df53783e260/merged
tmpfs                3.2G   36K  3.2G    1% /run/user/0
```
其中
```bash
/dev/mapper/rl-root   62G   47G   15G   76% /
```
发现快满了 但是不知道是哪个目录 接着使用 `find` 命令寻找大文件

```bash
find / -size +4000M
```
直接在根目录下寻找大于 4000M 的文件

```bash
[root@localhost ~]# find / -size +4000M
/proc/kcore
find: ‘/proc/767509/task/767509/fd/5’: 没有那个文件或目录
find: ‘/proc/767509/task/767509/fdinfo/5’: 没有那个文件或目录
find: ‘/proc/767509/fd/6’: 没有那个文件或目录
find: ‘/proc/767509/fdinfo/6’: 没有那个文件或目录
/var/lib/docker/containers/cd37e7e6668415a1b84ed97109bd785d1530f4662ef0caca45305a3b6ebf1b02/cd37e7e6668415a1b84ed97109bd785d1530f4662ef0caca45305a3b6ebf1b02-json.log
```

此时寻找一个 Docker 容器下的日志文件大于 4000M

到目录下 `ls -ll` 一看

```bash
[root@localhost ~]# cd /var/lib/docker/containers/cd37e7e6668415a1b84ed97109bd785d1530f4662ef0caca45305a3b6ebf1b02/
[root@localhost cd37e7e6668415a1b84ed97109bd785d1530f4662ef0caca45305a3b6ebf1b02]# ls -lh
总用量 11G
-rw-r-----. 1 root root  11G  7月 12 11:35 cd37e7e6668415a1b84ed97109bd785d1530f4662ef0caca45305a3b6ebf1b02-json.log
drwx------. 2 root root    6  6月 18 14:44 checkpoints
-rw-------. 1 root root  30K  7月 11 21:59 config.v2.json
-rw-------. 1 root root 1.9K  7月 11 21:59 hostconfig.json
-rw-r--r--. 1 root root   13  7月 11 21:59 hostname
-rw-r--r--. 1 root root  174  7月 11 21:59 hosts
drwx--x---. 2 root root    6  6月 18 14:44 mounts
-rw-r--r--. 1 root root  317  7月 11 21:59 resolv.conf
-rw-r--r--. 1 root root   71  7月 11 21:59 resolv.conf.hash
```
通过这个我们能看到 `cd37e7e6668415a1b84ed97109bd785d1530f4662ef0caca45305a3b6ebf1b02-json.log` 文件占用了 11G

接着使用 `` 命令查询该日志前 10 行

```bash
[root@localhost cd37e7e6668415a1b84ed97109bd785d1530f4662ef0caca45305a3b6ebf1b02]# head -n 10 cd37e7e6668415a1b84ed97109bd785d1530f4662ef0caca45305a3b6ebf1b02-json.log 
{"log":"Thank you for using GitLab Docker Image!\n","stream":"stdout","time":"2024-06-18T06:44:47.985803963Z"}
{"log":"Current version: gitlab-ce=17.0.2-ce.0\n","stream":"stdout","time":"2024-06-18T06:44:47.985835222Z"}
{"log":"\n","stream":"stdout","time":"2024-06-18T06:44:47.985839079Z"}
{"log":"Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file\n","stream":"stdout","time":"2024-06-18T06:44:47.985842175Z"}
{"log":"And restart this container to reload settings.\n","stream":"stdout","time":"2024-06-18T06:44:47.985845291Z"}
{"log":"To do it use docker exec:\n","stream":"stdout","time":"2024-06-18T06:44:47.985848156Z"}
{"log":"\n","stream":"stdout","time":"2024-06-18T06:44:47.985851212Z"}
{"log":"  docker exec -it gitlab editor /etc/gitlab/gitlab.rb\n","stream":"stdout","time":"2024-06-18T06:44:47.985876118Z"}
{"log":"  docker restart gitlab\n","stream":"stdout","time":"2024-06-18T06:44:47.98588256Z"}
{"log":"\n","stream":"stdout","time":"2024-06-18T06:44:47.985885686Z"}
```

观察发现这就是 GitLab Docker 镜像启动的日志文件

# 总结
至此赶紧删了也算找回一点磁盘容量了，从日志的日期不难看出这个容器才允许不到一个月时间，占用竟然来到了惊人的 11G 大小目前有两个解决方案，配置 Docker 容器日志上限或者配置 Gitlab 中日志等级

但是吧我是懒狗我选择先删除到时看心情再去配置