---
title: kill 
description: docker kill 指令
published: true
date: 2022-10-30T00:58:57.843Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T00:58:00.413Z
---

# docker kill 指令


**docker kill** :杀掉一个运行中的容器。

## 语法
```
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```
OPTIONS说明：

- -s :向容器发送一个信号

## 实例
杀掉运行中的容器mynginx
```
runoob@runoob:~$ docker kill -s KILL mynginx
mynginx
```