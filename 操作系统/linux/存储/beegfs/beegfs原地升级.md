---
title: beegfs原地升级
description: 
published: true
date: 2023-05-15T10:10:39.302Z
tags: linux
editor: markdown
dateCreated: 2023-05-15T10:10:25.962Z
---

## beegfs 升级

### 背景

beegfs 7.2.2  > beefs 7.3.3

软件包下载地址:


### 步骤

1、按照顺序停止所有的服务

```bash
## 停止所有 client
systemctl stop  beegfs-client
systemctl disable  beegfs-client
## 停止所有 helperd
systemctl stop  beegfs-helperd
systemctl disable  beegfs-helperd
## 停止所有 storage
systemctl stop  beegfs-storage
systemctl disable  beegfs-storage
## 停止所有meta
systemctl stop  beegfs-meta
systemctl disable  beegfs-meta
## 停止所有mgmtd
systemctl stop  beegfs-mgmtd
systemctl disable  beegfs-mgmtd
```

 2、 查看节点上安装了哪些包

```bash
[root@bqmgmtd ~]# yum list beegfs*
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
Installed Packages
beegfs-client.noarch                                                                  20:7.2.2-el7                                                           @beegfs
beegfs-common.noarch                                                                  20:7.2.2-el7                                                           @beegfs
beegfs-helperd.x86_64                                                                 20:7.2.2-el7                                                           @beegfs
beegfs-mgmtd.x86_64                                                                   20:7.2.2-el7                                                           @beegfs
beegfs-utils.x86_64                                                                   20:7.2.2-el7                                                           @beegfs


```

3、 升级包`mgmtd`

```bash
# 卸载源来的包
rpm -e beegfs-mgmtd beegfs-client beegfs-helperd beegfs-utils  beegfs-common
# 安装新包
rpm -ivh beegfs-common-7.3.3-el7.noarch.rpm 
rpm -ivh beegfs-mgmtd-7.3.3-el7.x86_64.rpm 
rpm -ivh beegfs-helperd-7.3.3-el7.x86_64.rpm 
rpm -ivh beegfs-client-7.3.3-el7.noarch.rpm 
rpm -ivh beegfs-utils-7.3.3-el7.x86_64.rpm
# 修改配置文件, 直接指向原来的配置文件所在地址
vim /etc/beegfs/beegfs-mgmtd.conf

- storeMgmtdDirectory      =
+ storeMgmtdDirectory      = /data/beegfs/beegfs_mgmtd

```

4、 升级`meta `节点

```bash
## 和升级`mgmtd`节点类似
rpm -e beegfs-meta beegfs-common 
## 安装新包
rpm -ivh beegfs-common-7.3.3-el7.noarch.rpm
rpm -ivh beegfs-meta-7.3.3-el7.x86_64.rpm
## 修改配置文件
vim /etc/beegfs/beegfs-meta.conf
- sysMgmtdHost                 =

- storeMetaDirectory           =
+ sysMgmtdHost                 = bqmgmtd
+ storeMetaDirectory           = /data/lun_meta
```



5、 `storage `节点升级

```bash
## 和升级`mgmtd`节点类似
rpm -e beegfs-storage beegfs-common 
## 安装新包
rpm -ivh beegfs-common-7.3.3-el7.noarch.rpm
rpm -ivh beegfs-storage-7.3.3-el7.x86_64.rpm
## 修改配置文件
vim /etc/beegfs/beegfs-storage.conf
- sysMgmtdHost                 =
+ sysMgmtdHost                 = bqmgmtd
- storeStorageDirectory        =
+ storeStorageDirectory        = ,/mnt/lun_storage_1 ,/mnt/lun_storage_2
- storeFsUUID                  =
+ storeFsUUID                  = ,1597effe-ed02-4323-b7b7-40059604690b ,40147f75-bba8-4c34-b263-049cecae493b

```

6、升级client

```bash
# 卸载源来的包
rpm -e  beegfs-client beegfs-helperd beegfs-utils  beegfs-common
# 安装新包
rpm -ivh beegfs-common-7.3.3-el7.noarch.rpm 
rpm -ivh beegfs-helperd-7.3.3-el7.x86_64.rpm 
rpm -ivh beegfs-client-7.3.3-el7.noarch.rpm 
rpm -ivh beegfs-utils-7.3.3-el7.x86_64.rpm
# 修改配置文件, 直接指向原来的配置文件所在地址
vim /etc/beegfs/beegfs-client.conf

- sysMgmtdHost                  =
+ sysMgmtdHost                  = bqmgmtd
```



7、beegfs-client 会启动失败, 需要升级内核

```bash
# 升级内核
yum localinstall *.rpm

grub2-set-default 0

reboot

```

8、`beegfs `修改编译文档

>  按照官方文档 需要使用`devtoolset-7`编译
>
> ![企业微信截图_fa9226ed-5d65-46bd-a07c-ef59ad00221d](.assets/06_BFS原地升级/企业微信截图_fa9226ed-5d65-46bd-a07c-ef59ad00221d.png)

```bash
## 安装 devtoolset-7
yum install centos-release-scl-rh
sudo yum install devtoolset-7

## 安装编译需要的依赖
yum install -y libelf-dev libelf-devel elfutils-libelf-devel  +124
## beegfs 修改编译文档

vim /etc/init.d/beegfs-client 

+ scl enable devtoolset-7 bash << __EOF__
+ gcc --version
set -e
make -C ${dir}/build auto_rebuild_configured ${TARGET_PARAM} --silent
make -C ${dir}/build clean --silent # always clean up
+ __EOF__
```



![image-20230515165243167](.assets/06_BFS原地升级/image-20230515165243167.png)

9、 处理完成之后, 按照顺序重启 `beegfs`

```bash
## 启动所有mgmtd
systemctl start  beegfs-mgmtd
systemctl enable  beegfs-mgmtd

## 启动所有meta
systemctl start  beegfs-meta
systemctl enable  beegfs-meta

## 启动所有 storage
systemctl start  beegfs-storage
systemctl enable  beegfs-storage

## 启动所有 helperd
systemctl start  beegfs-helperd
systemctl enable  beegfs-helperd

## 启动所有 client
systemctl start  beegfs-client
systemctl enable  beegfs-client

```





