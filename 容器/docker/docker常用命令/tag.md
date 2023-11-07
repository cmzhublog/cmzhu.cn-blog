---
title: tag
description: 
published: true
date: 2022-10-30T05:53:14.557Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:53:14.557Z
---

# docker tag 命令

**docker tag** : 标记本地镜像，将其归入某一仓库。

## 语法

```
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
```

## 实例
将镜像ubuntu:15.10标记为 runoob/ubuntu:v3 镜像。

```
root@runoob:~# docker tag ubuntu:15.10 runoob/ubuntu:v3
root@runoob:~# docker images   runoob/ubuntu:v3
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/ubuntu       v3                  4e3b13c8a266        3 months ago        136.3 MB
```
