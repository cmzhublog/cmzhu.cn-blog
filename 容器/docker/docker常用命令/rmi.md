---
title: rmi
description: 
published: true
date: 2022-10-30T05:51:46.432Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:51:46.432Z
---

# docker rmi 命令

**docker rmi** : 删除本地一个或多个镜像。

## 语法
```
docker rmi [OPTIONS] IMAGE [IMAGE...]
```
OPTIONS说明：

- -f :强制删除；

- --no-prune :不移除该镜像的过程镜像，默认移除；

## 实例
强制删除本地镜像 runoob/ubuntu:v4。
```
root@runoob:~# docker rmi -f runoob/ubuntu:v4
Untagged: runoob/ubuntu:v4
Deleted: sha256:1c06aa18edee44230f93a90a7d88139235de12cd4c089d41eed8419b503072be
Deleted: sha256:85feb446e89a28d58ee7d80ea5ce367eebb7cec70f0ec18aa4faa874cbd97c73
```
