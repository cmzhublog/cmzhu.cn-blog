---
title: load
description: 
published: true
date: 2022-10-30T06:01:18.885Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T06:01:18.885Z
---

# docker load 命令

**docker load** : 导入使用 docker save 命令导出的镜像。

## 语法
```
docker load [OPTIONS]
```

OPTIONS 说明：

- --input , -i : 指定导入的文件，代替 STDIN。

- --quiet , -q : 精简输出信息。

## 实例
导入镜像：
```
$ docker image ls

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

$ docker load < busybox.tar.gz

Loaded image: busybox:latest
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              769b9341d937        7 weeks ago         2.489 MB

$ docker load --input fedora.tar

Loaded image: fedora:rawhide

Loaded image: fedora:20

$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              769b9341d937        7 weeks ago         2.489 MB
fedora              rawhide             0d20aec6529d        7 weeks ago         387 MB
fedora              20                  58394af37342        7 weeks ago         385.5 MB
fedora              heisenbug           58394af37342        7 weeks ago         385.5 MB
fedora              latest              58394af37342        7 weeks ago         385.5 MB
```
