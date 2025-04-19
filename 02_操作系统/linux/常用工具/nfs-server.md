---
title: nfs-server
description: 
published: true
date: 2022-12-05T02:44:32.451Z
tags: linux
editor: markdown
dateCreated: 2022-12-05T02:10:34.958Z
---

# nfs-server 安装 



操作系统 : Rocky linux 9

```bash
yum install -y nfs-utils
systemctl enable rpcbind
systemctl enable nfs-server
```



编辑文档

```yaml
> vim /etc/exports


/mnt/data1/nfs_bqdata     10.24.0.0/16(rw,sync,no_root_squash,no_all_squash)
```



1. `/data`: 共享目录位置。
2. `192.168.0.0/24`: 客户端 IP 范围，`*` 代表所有，即没有限制。
3. `rw`: 权限设置，可读可写。
4. `sync`: 同步共享目录。
5. `no_root_squash`: 可以使用 root 授权。
6. `no_all_squash`: 可以使用普通用户授权。

检查

```bash
 > showmount -e localhost


Export list for localhost:
/mnt/data1 10.24.0.0/16
```

