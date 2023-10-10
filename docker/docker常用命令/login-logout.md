---
title: login/logout
description: 
published: true
date: 2022-10-30T05:04:50.845Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:04:50.845Z
---

# docker login/logout 命令

**docker login **: 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

**docker logout** : 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

## 语法
```
docker login [OPTIONS] [SERVER]
```
```
docker logout [OPTIONS] [SERVER]
```

OPTIONS说明：

- -u :登陆的用户名

- -p :登陆的密码

## 实例
登陆到Docker Hub
```
docker login -u 用户名 -p 密码
```
登出Docker Hub
```
docker logout
```
