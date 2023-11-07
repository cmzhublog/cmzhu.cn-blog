---
title: push
description: 
published: true
date: 2022-10-30T05:08:22.813Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:08:22.813Z
---

# docker push 命令

**docker push** : 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库

## 语法
```
docker push [OPTIONS] NAME[:TAG]
```

OPTIONS说明：

- --disable-content-trust :忽略镜像的校验,默认开启

## 实例
上传本地镜像myapache:v1到镜像仓库中。
```
docker push myapache:v1
```