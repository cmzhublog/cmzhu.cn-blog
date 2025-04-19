---
title: info
description: 
published: true
date: 2022-10-30T06:06:39.936Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T06:04:19.783Z
---

# docker info 命令

**docker info** : 显示 Docker 系统信息，包括镜像和容器数。。

## 语法
```
docker info [OPTIONS]
```

## 实例
查看docker系统信息。

```
$ docker info

Containers: 12
Images: 41
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 66
 Dirperm1 Supported: false
Execution Driver: native-0.2
Logging Driver: json-file
Kernel Version: 3.13.0-32-generic
Operating System: Ubuntu 14.04.1 LTS
CPUs: 1
Total Memory: 1.954 GiB
Name: iZ23mtq8bs1Z
ID: M5N4:K6WN:PUNC:73ZN:AONJ:AUHL:KSYH:2JPI:CH3K:O4MK:6OCX:5OYW
```
