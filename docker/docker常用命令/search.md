---
title: search
description: 
published: true
date: 2022-10-30T05:47:03.439Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:47:03.439Z
---

# docker search 命令


**docker search** : 从Docker Hub查找镜像

## 语法
```
docker search [OPTIONS] TERM
```

OPTIONS说明：

- --automated :只列出 automated build类型的镜像；

- --no-trunc :显示完整的镜像描述；

- -f <过滤条件>:列出收藏数不小于指定值的镜像。

## 实例
从 Docker Hub 查找所有镜像名包含 java，并且收藏数大于 10 的镜像
```
runoob@runoob:~$ docker search -f stars=10 java
NAME                  DESCRIPTION                           STARS   OFFICIAL   AUTOMATED
java                  Java is a concurrent, class-based...   1037    [OK]       
anapsix/alpine-java   Oracle Java 8 (and 7) with GLIBC ...   115                [OK]
develar/java                                                 46                 [OK]
isuper/java-oracle    This repository contains all java...   38                 [OK]
lwieske/java-8        Oracle Java 8 Container - Full + ...   27                 [OK]
nimmis/java-centos    This is docker images of CentOS 7...   13                 [OK]
```
参数说明：

- NAME: 镜像仓库源的名称

- DESCRIPTION: 镜像的描述

- OFFICIAL: 是否 docker 官方发布

- stars: 类似 Github 里面的 star，表示点赞、喜欢的意思。

- AUTOMATED: 自动构建。
