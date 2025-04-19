---
title: rocky9 命令
description: 
published: true
date: 2023-02-24T06:26:02.904Z
tags: linux
editor: markdown
dateCreated: 2023-02-23T02:10:06.214Z
---

# rocky9 常用命令

1、 查询某一个工具在哪一个安装包里面

背景: rocky9 系统中没有lspci 工具,查看需要下载什么包

```bash
dnf provides lspci

## 执行结果
Last metadata expiration check: 3:09:40 ago on Thu Feb 23 06:55:07 2023.
pciutils-3.7.0-5.el9.i686 : PCI bus related utilities
Repo        : appstream
Matched from:
Filename    : /usr/sbin/lspci
Provide    : /sbin/lspci

pciutils-3.7.0-5.el9.x86_64 : PCI bus related utilities
Repo        : baseos
Matched from:
Filename    : /usr/sbin/lspci
Provide    : /sbin/lspci
```

查询出安装包之后,直接安装

```bash
dnf install -y pciutils-3.7.0-5.el9.x86_64
```

2、 安装工具包

```bash
dnf groupinstall -y "Development Tools"
```

3、 dnf 搜索grouplist
```bash
dnf grouplist 
```
