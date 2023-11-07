---
title: import
description: 
published: true
date: 2022-10-30T06:02:49.790Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T06:02:49.790Z
---

# docker import 命令
**docker import** : 从归档文件中创建镜像。

## 语法
```
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

OPTIONS说明：

- -c :应用docker 指令创建镜像；

- -m :提交时的说明文字；

## 实例
从镜像归档文件my_ubuntu_v3.tar创建镜像，命名为runoob/ubuntu:v4

```
runoob@runoob:~$ docker import  my_ubuntu_v3.tar runoob/ubuntu:v4  
sha256:63ce4a6d6bc3fabb95dbd6c561404a309b7bdfc4e21c1d59fe9fe4299cbfea39
runoob@runoob:~$ docker images runoob/ubuntu:v4
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/ubuntu       v4                  63ce4a6d6bc3        20 seconds ago      142.1 MB
```
