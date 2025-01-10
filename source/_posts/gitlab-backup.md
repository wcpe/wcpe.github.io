---
title: Gitlab 备份
author: WCPE
date: 2025-1-08 0:09:56
categories:
- 运维
tags:
- 'Gitlab'

---

### 数据备份

使用命令 `gitlab-backup create` 进行备份的创建

示例输出

```
Dumping database tables:
- Dumping table events... [DONE]
- Dumping table issues... [DONE]
- Dumping table keys... [DONE]
- Dumping table merge_requests... [DONE]
- Dumping table milestones... [DONE]
- Dumping table namespaces... [DONE]
- Dumping table notes... [DONE]
- Dumping table projects... [DONE]
- Dumping table protected_branches... [DONE]
- Dumping table schema_migrations... [DONE]
- Dumping table services... [DONE]
- Dumping table snippets... [DONE]
- Dumping table taggings... [DONE]
- Dumping table tags... [DONE]
- Dumping table users... [DONE]
- Dumping table users_projects... [DONE]
- Dumping table web_hooks... [DONE]
- Dumping table wikis... [DONE]
Dumping repositories:
- Dumping repository abcd... [DONE]
Creating backup archive: <backup-id>_gitlab_backup.tar [DONE]
Deleting tmp directories...[DONE]
Deleting old backups... [SKIPPING]
```

备份路径 `/var/opt/gitlab/backups` 会出现一个类似 `1736266336_2025_01_07_17.4.2_gitlab_backup.tar` 的文件



### 数据恢复

复制文件到 `/var/opt/gitlab/backups` 并修改文件权限 
```
sudo cp 1736266336_2025_01_07_17.4.2_gitlab_backup.tar /var/opt/gitlab/backups/
sudo chown git:git /var/opt/gitlab/backups/1736266336_2025_01_07_17.4.2_gitlab_backup.tar
```

使用命令先停止数据库连接

```
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
# Verify
sudo gitlab-ctl status
```

然后使用命令

```
sudo gitlab-backup restore BACKUP=1736266336_2025_01_07_17.4.2
```
其中 `1736266336_2025_01_07_17.4.2` 这段为备份文件名称前段
