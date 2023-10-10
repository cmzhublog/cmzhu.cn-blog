---
title: nfs
description: 
published: true
date: 2022-12-07T08:25:15.540Z
tags: linux
editor: markdown
dateCreated: 2022-12-07T08:25:15.540Z
---

# nfs配置



## nfs 安装

安装nfs

```bash
yum install  nfs-utils
```

配置nfs

```bash
vim /etc/exports

/upload 192.168.1.0/24(rw,async,no_root_squash)
```



重启nfs

```bash
systemctl restart nfs-server
```



挂载

```bash
nfs3.0 挂载方式
file-system-id.region.nas.aliyuncs.com:/ /mnt nfs vers=3,nolock,proto=tcp,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,_netdev,noresvport 0 0
nfs 4.0 挂载方式
file-system-id.region.nas.aliyuncs.com:/ /mnt nfs vers=4,minorversion=0,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,_netdev,noresvport 0 0
极速行nas 挂载方式
file-system-id.region.extreme.nas.aliyuncs.com:/share /mnt nfs vers=3,nolock,noacl,proto=tcp,noresvport,_netdev 0 0
```





| 参数                    | 说明                                                         |
| :---------------------- | :----------------------------------------------------------- |
| _netdev                 | 防止客户端在网络就绪之前开始挂载文件系统。                   |
| 0（noresvport后第一项） | 非零值表示文件系统应由dump备份。对于NAS文件系统而言，此值默认为0。 |
| 0（noresvport后第二项） | 该值表示fsck在启动时检查文件系统的顺序。对于NAS文件系统而言，此值默认为0，表示fsck不应在启动时运行。 |



