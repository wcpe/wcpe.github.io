---
title: Gitlab 迁移后的文件权限问题
author: WCPE
date: 2024-10-11 17:39:15
categories:
- 运维
tags:
- 'Gitlab'
---

# 起因

我的 `RockyLinux 9` 100G 硬盘分区不够用了我给他进行了扩容操作 然后经过一系列脑瘫操作之后成功的把 lvm 卷搞丢了 我就准备迁移数据重装一个 Linux 系统 于是选择了 `Debian 12`

经过我的不懈努力安装完成了 Debian12 装上了 1panel 面板之后 把数据通过 sync 迁移过去之后 发现 gitlab 仓库无法进行 pull push 等操作 都会提示
```
17:23:52.302: [ProjectName] git -c credential.helper= -c core.quotepath=false -c log.showSignature=false push --progress --porcelain origin refs/heads/master:master --tags
fatal: unable to access 'https://git.example.top/wcpeplugin/ProjectName.git/': Error while processing content unencoding: invalid stored block lengths
```

最后在 gitlab issue 中找到了解决方案 

> https://gitlab.com/gitlab-org/charts/gitlab/-/issues/5546

> codissimo1 @codissimo1
>
>I had the same problem Error while processing content unencoding: invalid stored block lengths when upgrading gitlab-ee (ubuntu repository package) from 16.11.2 to 17.2.1
>
>It turned out: some files in /var/opt/gitlab/git-data/repositories had owner = root.
>
>Solution:
>
>>find /var/opt/gitlab/git-data/repositories -exec chown git:git {} \;
>
>Thanks to @pks-gitlab

没错 只需要进入 gitlab-ce docker 容器中 输入 `find /var/opt/gitlab/git-data/repositories -exec chown git:git {} \;` 将权限修正即可