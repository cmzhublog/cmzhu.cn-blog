---
title: commit
description: 
published: true
date: 2022-10-30T04:58:12.095Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T04:58:12.095Z
---

# docker commit 语法

**docker commit** :从容器创建一个新的镜像。

## 语法

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

OPTIONS说明：

- -a :提交的镜像作者；

- -c :使用Dockerfile指令来创建镜像；

- -m :提交时的说明文字；

- -p :在commit时，将容器暂停。

## 实例
将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。

```
runoob@runoob:~$ docker commit -a "runoob.com" -m "my apache" a404c6c174a2  mymysql:v1 
sha256:37af1236adef1544e8886be23010b66577647a40bc02c0885a6600b33ee28057
runoob@runoob:~$ docker images mymysql:v1
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mymysql             v1                  37af1236adef        15 seconds ago      329 MB
```
