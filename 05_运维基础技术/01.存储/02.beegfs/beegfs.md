---
title: beegfs测试部署
description: 
published: true
date: 2023-04-12T02:02:21.066Z
tags: linux
editor: markdown
dateCreated: 2023-04-12T02:02:21.066Z
---

# beegfs 安装

本次实验通过 hyper-v 虚拟机来完成

| Hostname | IP              | 配置 | 服务                                      |
| -------- | --------------- | ---- | ----------------------------------------- |
| bfs001   | 192.168.101.201 | 4Gi  | beegfs-mgmtd, beegfs-meta, beegfs-storage |
| Bfs002   | 192.168.101.202 | 4Gi  | beegfs-storage                            |
| Bfs003   | 192.168.101.203 | 4Gi  | beegfs-storage                            |

## 前期准备

1、 配置三台服务器hosts(三台都执行)

```bash
echo "192.168.101.201 bfs001" >> /etc/hosts
echo "192.168.101.202 bfs002" >> /etc/hosts
echo "192.168.101.203 bfs003" >> /etc/hosts
```

2、 关闭所有机器防火墙和SELINUX(三台都执行)

```bash
systemctl stop firewalld
systemctl disable firewalld

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

3、 创建文件夹 `/mnt/beegfs-data`(三台都执行)

```bash
 mkdir /mnt/beegfs-data
```



## 开始安装

1、 下载beegfs 源(三台都执行)

```bash
dnf install wget -y && wget -O /etc/yum.repos.d/beegfs-rhel9.repo https://www.beegfs.io/release/beegfs_7.3.2/dists/beegfs-rhel9.repo
```

2、安装依赖包(三台都执行)

```bash
dnf install kernel-devel -y && dnf groupinstall -y "Development Tools"
```

3、 依照对应节点 安装服务

`bfs001`

3.1 安装 `beegfs-mgmtd `

```bash
dnf install beegfs-mgmtd -y
```

3.2 安装`beegfs-meta`

```bash
dnf install beegfs-meta -y
```

3.3 安装 `beegfs-storage`

```bash
dnf install beegfs-storage -y
```



bfs002

3.1 安装`beegfs-storage`

```bash
dnf install beegfs-storage -y
```



`bfs003`

3.1 安装`beegfs-storage`

```bash
dnf install beegfs-storage -y
```



4、 在所有需要挂载的client 节点安装对应的服务

```bash
dnf install -y beegfs-client beegfs-helperd beegfs-utils beegfs-common kernel-devel
```

## Management 节点配置

`bfs001`

```bash
/opt/beegfs/sbin/beegfs-setup-mgmtd -p /mnt/beegfs-data/beegfs_mgmtd
```

1、 配置文件位置

```bash
vi /etc/beegfs/beegfs-mgmtd.conf
```

| 配置项                                         | 功能           |
| ---------------------------------------------- | -------------- |
| connDisableAuthentication = true               | 配置不使用认证 |
| connInterfacesFile = /etc/beegfs/conn-inf.conf | 配置网卡       |
|                                                |                |

/etc/beegfs/conn-inf.conf 内容如下

```bash
# cat /etc/beegfs/conn-inf.conf
eth0
```



## Meta 节点配置

`bfs001`

```bash
/opt/beegfs/sbin/beegfs-setup-meta -p /mnt/beegfs-data/beegfs_meta -s 1 -m 192.168.101.201
```

1、 配置文件位置

```bash
vi /etc/beegfs/beegfs-meta.conf
```

| 配置项                                         | 功能           |
| ---------------------------------------------- | -------------- |
| connDisableAuthentication = true               | 配置不使用认证 |
| connInterfacesFile = /etc/beegfs/conn-inf.conf | 配置网卡       |
|                                                |                |

/etc/beegfs/conn-inf.conf 内容如下

```bash
# cat /etc/beegfs/conn-inf.conf
eth0
```

## Storage 节点配置

```bash
# bfs001
/opt/beegfs/sbin/beegfs-setup-storage -p /mnt/beegfs-data/beegfs_storage -s 1 -i 101 -m 192.168.101.201

# bfs002
/opt/beegfs/sbin/beegfs-setup-storage -p /mnt/beegfs-data/beegfs_storage -s 2 -i 201 -m 192.168.101.202

# bfs003
/opt/beegfs/sbin/beegfs-setup-storage -p /mnt/beegfs-data/beegfs_storage -s 3 -i 301 -m 192.168.101.203
```

1、 修改配置文件

```bash
vi /etc/beegfs/beegfs-storage.conf
```



| 配置项                                         | 功能           |
| ---------------------------------------------- | -------------- |
| connDisableAuthentication = true               | 配置不使用认证 |
| connInterfacesFile = /etc/beegfs/conn-inf.conf | 配置网卡       |
|                                                |                |



### Client 节点配置

```bash
# bfs001 2 3 相同
/opt/beegfs/sbin/beegfs-setup-client -m 192.168.101.201
```

1、 修改配置文件

```bash
vi /etc/beegfs/beegfs-helperd.conf
vi /etc/beegfs/beegfs-client.conf
```

| 配置项                                         | 功能           |
| ---------------------------------------------- | -------------- |
| connDisableAuthentication = true               | 配置不使用认证 |
| connInterfacesFile = /etc/beegfs/conn-inf.conf | 配置网卡       |
|                                                |                |



## 启动服务

```bash
Management 节点:
   systemctl start beegfs-mgmtd
   systemctl enable beegfs-mgmtd
Metadata 节点:
   systemctl start beegfs-meta
   systemctl enable beegfs-meta
Storage 节点:
   systemctl start beegfs-storage
   systemctl enable beegfs-storage
Client 节点:
   systemctl start beegfs-helperd
   systemctl enable beegfs-helperd
   systemctl start beegfs-client
   systemctl enable beegfs-client
Mon 节点:
   systemctl start beegfs-mon
   systemctl enable beegfs-mon
```

## 异常问题处理

### 镜像组出现 Need-resync

出现如下状态（这里以 meta 为例）

```
> beegfs-ctl --listtargets --nodetype=meta --state

TargetID     Reachability  Consistency   NodeID
========     ============  ===========   ======
       2           Online Needs-resync        2
       3           Online Needs-resync        3
```

确定主从

```
beegfs-ctl --listmirrorgroups --nodetype=meta
```

重新同步

```
beegfs-ctl --setstate --nodetype=meta/storage --nodeid=<nodeid> --state=good --force

# 上一步设置 Good 后，就开始 resync 了
beegfs-ctl --startresync --nodeid=<nodeid> --nodetype=meta
beegfs-ctl --startresync --nodetype=storage --targetid=X --timestamp=0 --restart 
```

### 内核 5.14 无法使用 beegfs 7.3.2

修改：

```
sed -i 's|#include <stdarg.h>|//#include <linux/stdarg.h>|g' /opt/beegfs/src/client/client_module_7/source/common/Common.h

sed -i 's|PDE_DATA(const struct inode \*)|pde_data(const struct inode \*inode)|g' /opt/beegfs/src/client/client_module_7/build/KernelFeatureDetection.mk

sed -i 's|PDE_DATA(|pde_data(|g' /opt/beegfs/src/client/client_module_7/source/filesystem/ProcFs.c
```

其他修改处：

```
> vim /opt/beegfs/src/client/client_module_7/source/Makefile +186

diff --git a/client_module/source/Makefile b/client_module/source/Makefile
index 375f647..d8f612c 100644
--- a/client_module/source/Makefile
+++ b/client_module/source/Makefile
@@ -184,6 +184,8 @@ SOURCES := \
 
 ${TARGET}-y    := $(patsubst %.c,%.o,$(SOURCES))
 
+NOSTDINC_FLAGS+=-isystem $(shell $(CC) -print-file-name=include)
+
 ifneq ($(OFED_INCLUDE_PATH),)
 
```

