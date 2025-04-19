---
title: save
description: 
published: true
date: 2022-10-30T05:59:45.288Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:59:45.288Z
---

# docker save 命令

**docker save** : 将指定镜像保存成 tar 归档文件。

## 语法
```
docker save [OPTIONS] IMAGE [IMAGE...]
```

OPTIONS 说明：

- -o :输出到的文件。

## 实例
将镜像 runoob/ubuntu:v3 生成 my_ubuntu_v3.tar 文档
```
runoob@runoob:~$ docker save -o my_ubuntu_v3.tar runoob/ubuntu:v3
runoob@runoob:~$ ll my_ubuntu_v3.tar
-rw------- 1 runoob runoob 142102016 Jul 11 01:37 my_ubuntu_v3.tar
```