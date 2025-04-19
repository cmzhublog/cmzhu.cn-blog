---
title: create
description: docker create
published: true
date: 2022-10-30T01:09:55.937Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:09:55.937Z
---

# docker create 命令


**docker create** ：创建一个新的容器但不启动它

用法同 docker run

## 语法
```
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```
语法同 docker run

## 实例
使用docker镜像nginx:latest创建一个容器,并将容器命名为myrunoob

```
runoob@runoob:~$ docker create  --name myrunoob  nginx:latest      
09b93464c2f75b7b69f83d56a9cfc23ceb50a48a9db7652ee4c27e3e2cb1961f
```
