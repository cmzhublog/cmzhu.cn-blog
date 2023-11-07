---
title: pull
description: 
published: true
date: 2022-10-30T05:06:49.657Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:06:49.657Z
---

# docker pull 命令


**docker pull** : 从镜像仓库中拉取或者更新指定镜像

## 语法
```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

OPTIONS说明：

- -a :拉取所有 tagged 镜像

- --disable-content-trust :忽略镜像的校验,默认开启

## 实例
从Docker Hub下载java最新版镜像。
```
docker pull java
```

从Docker Hub下载REPOSITORY为java的所有镜像。
```
docker pull -a java
```