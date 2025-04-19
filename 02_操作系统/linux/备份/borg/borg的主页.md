---
title: borg 备份
description: 
published: true
date: 2023-04-16T03:32:44.255Z
tags: linux
editor: markdown
dateCreated: 2023-04-16T03:32:44.255Z
---

# borg 文件备份

在 Linux 中，有多种备份工具提供系统级备份和用户数据备份功能



## 背景

当我在工作中处理大量数据时，首先想到的显而易见的问题是：

- 如果我丢失了数据怎么办？
- 我的数据的安全性如何？

相同的场景将应用于个人机器，您应该始终根据数据的重要性备份您的数据，并保护您的数据免受非法访问。

无论是您的个人数据还是官方数据，您都应该始终规划良好的备份策略，并使用强大的备份工具来帮助您备份重要数据。

最受推荐和广泛使用的开源备份应用程序之一是“**Borg**”。



## 什么是`Borg`

**BorgBackup**，简称borg，是一种备份工具，旨在提供一种使用重复数据删除技术备份数据的有效方法

下面列出了Borg的一些独特功能

- **重复数据删除**- 重复数据删除技术仅存储数据的增量副本，非常适合进行日常备份

- **跨平台**- Borg 可以在 Linux、Mac OS X 和 FreeBSD 中安装和使用

- **安全**- 支持使用AES加密（256 位）的数据加密，以验证使用HMAC-SHA256的真实性

- **压缩**- 可以使用以下压缩方法压缩数据：

- - LZ4 -> 超快，低压缩
  - ZSTD -> 高速低压缩到低速高压缩
  - ZLIB -> 中等速度，中等压缩
  - LZMA -> 低速，高压缩

- **远程备份**- 数据可以通过 SSH 协议备份到远程机器

## `linux` 安装 `Borg`

Borg 在大多数 Linux 发行版的默认存储库中可用，因此，它可以使用特定于发行版的包管理器进行安装

在 rech linux 中安装：

```bash
dnf install borgbackup -y
```



## 使用 `Borg` 进行第一次备份

在进行第一次备份之前，您必须了解两个重要术语

- **档案**- 您的数据的备份副本（快照）将被称为档案
- **存储库**- 本地或远程文件系统中存储档案的目录

首先初始化存储档案的存储库（目录）

我在名为`source`的目录中有一个文件列表并创建了一个名为`backup`的新目录，它将作为我存储档案的存储库



## 初始化仓库

执行`borg init` 命令初始化目录， 备份目录可以在本地机器，也可以是远程机器

```bash
borg init --encryption=none /home/karthick/borg/backup
```

使用`repokey `加密

```bash
borg init --encryption=repokey  /home/karthick/borg/backup
```



使用`keyfile` 加密

```bash
borg init --encryption=keyfile  /home/karthick/borg/backup
```



初始化存储库时，您可以选择加密类型，当您使用加密类型为`None` 时，不会应用任何加密，当您使用`repokey`和`keyfile`作为加密类型时，它使用`AES-CTR-256`进行加密。

选择任何一种加密类型并运行该`init`命令，就我而言，出于演示目的，我选择加密类型为无



## 使用`borg` 备份文件

### 创建第一个备份

初始化备份文件夹之后， 可以通过下面命令来创建备份

```bash
borg create --stats --progress /home/cmzhu/backup::20230416 /home/cmzhu/kube-state-metrics
```

在这里，我以模拟每日备份的日期格式`20230416`给出存档名称，没有`--stats`和`--progress`标志，创建命令的输出将是安静的。

### 创建备份时显示文件

```bash
borg create --list -v /home/cmzhu/backup/::20230416001 /home/cmzhu/kube-state-metrics
```

```bash
[root@cmzhu cmzhu]# borg create --list -v /home/cmzhu/backup/::20230416004 /home/cmzhu/kube-state-metrics
Creating archive at "/home/cmzhu/backup/::20230416004"
U /home/cmzhu/kube-state-metrics/Chart.yaml
U /home/cmzhu/kube-state-metrics/.helmignore
U /home/cmzhu/kube-state-metrics/README.md
U /home/cmzhu/kube-state-metrics/values.yaml
A /home/cmzhu/kube-state-metrics/test
U /home/cmzhu/kube-state-metrics/templates/NOTES.txt
U /home/cmzhu/kube-state-metrics/templates/_helpers.tpl
U /home/cmzhu/kube-state-metrics/templates/clusterrolebinding.yaml
U /home/cmzhu/kube-state-metrics/templates/deployment.yaml
U /home/cmzhu/kube-state-metrics/templates/kubeconfig-secret.yaml
U /home/cmzhu/kube-state-metrics/templates/pdb.yaml
U /home/cmzhu/kube-state-metrics/templates/podsecuritypolicy.yaml
U /home/cmzhu/kube-state-metrics/templates/psp-clusterrole.yaml
U /home/cmzhu/kube-state-metrics/templates/psp-clusterrolebinding.yaml
U /home/cmzhu/kube-state-metrics/templates/rbac-configmap.yaml
U /home/cmzhu/kube-state-metrics/templates/role.yaml
U /home/cmzhu/kube-state-metrics/templates/rolebinding.yaml
U /home/cmzhu/kube-state-metrics/templates/service.yaml
U /home/cmzhu/kube-state-metrics/templates/serviceaccount.yaml
U /home/cmzhu/kube-state-metrics/templates/servicemonitor.yaml
U /home/cmzhu/kube-state-metrics/templates/stsdiscovery-role.yaml
U /home/cmzhu/kube-state-metrics/templates/stsdiscovery-rolebinding.yaml
U /home/cmzhu/kube-state-metrics/templates/verticalpodautoscaler.yaml
d /home/cmzhu/kube-state-metrics/templates
d /home/cmzhu/kube-state-metrics
[root@cmzhu cmzhu]# 

```



### 创建压缩备份

默认情况下，`borg` 使用**`lz4`压缩算法**，`lz4`压缩算法速度非常快，压缩比低，如果您想使用不同的压缩算法，您可以使用该`--compression`标志并将类型与压缩级别一起传递

例如，如果我想使用zstd算法，那么我的命令将如下所示

```bash
borg create --compression zstd,1 /home/cmzhu/backup/::20230416002 /home/cmzhu/kube-state-metrics

```

您可以从 Borg 官方文档中查看不同的压缩算法及其级别

### 列出备份

使用`Borg list` 您可以查询您的存储库以查找档案列表以及档案中包含哪些文件

要单独获取档案列表，请运行以下命令

```bash
 borg list /home/cmzhu/backup
```

```bash
[root@cmzhu cmzhu]#  borg list /home/cmzhu/backup
20230416000                          Sun, 2023-04-16 11:02:46 [ccd447e44333e52bbb301b6c6d39f59003ef266ab3ebb4a4fcf943e9d6621eb4]
20230416001                          Sun, 2023-04-16 11:05:35 [752b594a8a4819bd98855742fb452825da97e386b230f533aa70be90e0fceb68]
20230416002                          Sun, 2023-04-16 11:08:15 [7e5a6c06ab89682cf2bb2355d3f73c28fa488c056f9d6a81556c30faf172121d]
20230416003                          Sun, 2023-04-16 11:21:16 [d438e2e37dc98652082881b40b2cd1a1a843aa714d07d4cc8ccd648c827de94d]

```



您还可以使用该`--json`标志，它将以`json`格式提供有关存储库和档案列表的更多信息

```bash
 borg list --json /home/cmzhu/backup

```

```bash
[root@cmzhu cmzhu]#  borg list --json /home/cmzhu/backup
{
    "archives": [
        {
            "archive": "20230416000",
            "barchive": "20230416000",
            "id": "ccd447e44333e52bbb301b6c6d39f59003ef266ab3ebb4a4fcf943e9d6621eb4",
            "name": "20230416000",
            "start": "2023-04-16T11:02:46.000000",
            "time": "2023-04-16T11:02:46.000000"
        },
        {
            "archive": "20230416001",
            "barchive": "20230416001",
            "id": "752b594a8a4819bd98855742fb452825da97e386b230f533aa70be90e0fceb68",
            "name": "20230416001",
            "start": "2023-04-16T11:05:35.000000",
            "time": "2023-04-16T11:05:35.000000"
        },
        {
            "archive": "20230416002",
            "barchive": "20230416002",
            "id": "7e5a6c06ab89682cf2bb2355d3f73c28fa488c056f9d6a81556c30faf172121d",
            "name": "20230416002",
            "start": "2023-04-16T11:08:15.000000",
            "time": "2023-04-16T11:08:15.000000"
        },
        {
            "archive": "20230416003",
            "barchive": "20230416003",
            "id": "d438e2e37dc98652082881b40b2cd1a1a843aa714d07d4cc8ccd648c827de94d",
            "name": "20230416003",
            "start": "2023-04-16T11:21:16.000000",
            "time": "2023-04-16T11:21:16.000000"
        }
    ],
    "encryption": {
        "mode": "none"
    },
    "repository": {
        "id": "469123c4118e0f016ec78d1dd213e0514b9c229d16f5d8c6088c5c797981d781",
        "last_modified": "2023-04-16T11:23:12.000000",
        "location": "/home/cmzhu/backup"
    }
}

```



### 列出存档中的文件

列出 `20230416` 存档中的文件

```bash
borg list /home/cmzhu/backup::20230416
```

```bash
[root@cmzhu cmzhu]# borg list /home/cmzhu/backup::20230416
drwxr-xr-x root   root          0 Wed, 2023-02-01 18:52:17 home/cmzhu/kube-state-metrics
-rw-r--r-- root   root        494 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/Chart.yaml
-rw-r--r-- root   root        333 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/.helmignore
-rw-r--r-- root   root       2725 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/README.md
-rw-r--r-- root   root      12810 Wed, 2023-02-01 18:52:17 home/cmzhu/kube-state-metrics/values.yaml
drwxr-xr-x root   root          0 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates
-rw-r--r-- root   root       1114 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/NOTES.txt
-rw-r--r-- root   root       3436 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/_helpers.tpl
-rw-r--r-- root   root        662 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/clusterrolebinding.yaml
-rw-r--r-- root   root      10489 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/deployment.yaml
-rw-r--r-- root   root        348 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/kubeconfig-secret.yaml
-rw-r--r-- root   root        579 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/pdb.yaml
-rw-r--r-- root   root       1039 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/podsecuritypolicy.yaml
-rw-r--r-- root   root        696 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/psp-clusterrole.yaml
-rw-r--r-- root   root        622 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/psp-clusterrolebinding.yaml
-rw-r--r-- root   root        477 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/rbac-configmap.yaml
-rw-r--r-- root   root       5203 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/role.yaml
-rw-r--r-- root   root        771 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/rolebinding.yaml
-rw-r--r-- root   root       1635 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/service.yaml
-rw-r--r-- root   root        592 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/serviceaccount.yaml
-rw-r--r-- root   root       3336 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/servicemonitor.yaml
-rw-r--r-- root   root        558 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/stsdiscovery-role.yaml
-rw-r--r-- root   root        636 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/stsdiscovery-rolebinding.yaml
-rw-r--r-- root   root       1348 Wed, 2023-02-01 18:04:55 home/cmzhu/kube-state-metrics/templates/verticalpodautoscaler.yaml
```

使用 `--json-lines`

```bash
borg list --json-lines /home/cmzhu/backup::20230416
```

```bash
[root@cmzhu cmzhu]# borg list --json-lines /home/cmzhu/backup::20230416 | jq
{
  "type": "d",
  "mode": "drwxr-xr-x",
  "user": "root",
  "group": "root",
  "uid": 0,
  "gid": 0,
  "path": "home/cmzhu/kube-state-metrics",
  "healthy": true,
  "source": "",
  "linktarget": "",
  "flags": 0,
  "mtime": "2023-02-01T18:52:17.180590",
  "size": 0
}
...
```

### 从备份中排除文件和目录

您可以使用`-e`或`--exclude`标志排除文件和目录

```bash
borg list /home/cmzhu/backup::25-11-2021 --exclude "hist"
```

### 找出档案之间的差异

可以使用 `diff` 命令来比对两个存档

```bash
borg diff /home/cmzhu/backup/::27-11-2021 28-11-2021
```

```bash
[root@cmzhu cmzhu]# borg diff /home/cmzhu/backup/::20230416 20230416003
[ctime: Wed, 2023-02-01 18:52:17 -> Sun, 2023-04-16 11:20:51] [mtime: Wed, 2023-02-01 18:52:17 -> Sun, 2023-04-16 11:20:51] home/cmzhu/kube-state-metrics
added           0 B home/cmzhu/kube-state-metrics/test
[root@cmzhu cmzhu]# 
```

### 重命名档案

创建存档后，如果您想重命名它，您可以使用该borg rename命令来完成

```bash
borg rename /home/cmzhu/backup/::20230416 20230416000
```

```bash
[root@cmzhu cmzhu]# borg rename /home/cmzhu/backup/::20230416 20230416000
[root@cmzhu cmzhu]# borg list /home/cmzhu/backup/
20230416000                          Sun, 2023-04-16 11:02:46 [ccd447e44333e52bbb301b6c6d39f59003ef266ab3ebb4a4fcf943e9d6621eb4]
20230416001                          Sun, 2023-04-16 11:05:35 [752b594a8a4819bd98855742fb452825da97e386b230f533aa70be90e0fceb68]
20230416002                          Sun, 2023-04-16 11:08:15 [7e5a6c06ab89682cf2bb2355d3f73c28fa488c056f9d6a81556c30faf172121d]
20230416003                          Sun, 2023-04-16 11:21:16 [d438e2e37dc98652082881b40b2cd1a1a843aa714d07d4cc8ccd648c827de94d]
```

### 使用`borg` 恢复数据

备份数据的主要重点是**在需要时进行恢复**，因此，您可以使用该`borg extract`命令从档案中检索数据，当您运行该`extract`命令时，它会将数据提取到您从中运行提取命令的当前工作目录

运行以下命令将存档解压缩到当前工作目录。随着`-v`和`--list`标志补充说，它会告诉你提取的文件列表

```bash
borg extract -v --list backup/::20230416000



```

您还可以使用`--dry-run`标志，它只会显示将要提取的内容而不是提取它

```bash
borg extract --dry-run -v --list backup/::20230416000
```

```bash
[root@cmzhu cmzhu]# borg extract --dry-run -v --list backup/::20230416000
home/cmzhu/kube-state-metrics
home/cmzhu/kube-state-metrics/Chart.yaml
home/cmzhu/kube-state-metrics/.helmignore
home/cmzhu/kube-state-metrics/README.md
home/cmzhu/kube-state-metrics/values.yaml
home/cmzhu/kube-state-metrics/templates
home/cmzhu/kube-state-metrics/templates/NOTES.txt
home/cmzhu/kube-state-metrics/templates/_helpers.tpl
home/cmzhu/kube-state-metrics/templates/clusterrolebinding.yaml
home/cmzhu/kube-state-metrics/templates/deployment.yaml
home/cmzhu/kube-state-metrics/templates/kubeconfig-secret.yaml
home/cmzhu/kube-state-metrics/templates/pdb.yaml
home/cmzhu/kube-state-metrics/templates/podsecuritypolicy.yaml
home/cmzhu/kube-state-metrics/templates/psp-clusterrole.yaml
home/cmzhu/kube-state-metrics/templates/psp-clusterrolebinding.yaml
home/cmzhu/kube-state-metrics/templates/rbac-configmap.yaml
home/cmzhu/kube-state-metrics/templates/role.yaml
home/cmzhu/kube-state-metrics/templates/rolebinding.yaml
home/cmzhu/kube-state-metrics/templates/service.yaml
home/cmzhu/kube-state-metrics/templates/serviceaccount.yaml
home/cmzhu/kube-state-metrics/templates/servicemonitor.yaml
home/cmzhu/kube-state-metrics/templates/stsdiscovery-role.yaml
home/cmzhu/kube-state-metrics/templates/stsdiscovery-rolebinding.yaml
home/cmzhu/kube-state-metrics/templates/verticalpodautoscaler.yaml
```



