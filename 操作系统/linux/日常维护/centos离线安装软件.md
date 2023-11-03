---
title: linux离线安装软件
description: 
published: true
date: 2022-12-06T10:17:09.828Z
tags: linux
editor: markdown
dateCreated: 2022-12-06T10:16:42.077Z
---

# centos 离线安装软件(not install)



## 背景

有时在某些时候需要安装软件,但是系统上又没有办法访问公网,这时离线安装就派上用场.

## 步骤

1、 启动一个对应版本的容器, 目的是为了在外部营造一个纯净的centos 系统,不会过多的,可以将软件包的所有依赖都离线下载下来.

```bash
docker run -it --rm -v ${PWD}/soft:/root/soft centos:7 bash
```

2、 进入容器后,执行(后续步骤在容器中执行)

```bash
cd /root/soft
yum install --downloadonly --downloaddir=/root/soft nfs-utils
```

3、退出容器,当前目录下的 soft 文件夹下就是下载下来的软件包

4、 拷贝到无法联网机器上

5、 安装

```bash
cd soft 

rpm -Uvh *.rpm
```

6、 正常当前情况,就安装好了