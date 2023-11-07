---
title: docker start/stop/restart
description: 
published: true
date: 2022-10-30T00:55:38.576Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T00:53:25.773Z
---

# docker start/stop/restart 命令

**docker start** :启动一个或多个已经被停止的容器

**docker stop** :停止一个运行中的容器

**docker restart** :重启容器

## 语法
```
docker start [OPTIONS] CONTAINER [CONTAINER...]
```
```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```
```
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```
## 实例
启动已被停止的容器myrunoob
```
docker start myrunoob
```

停止运行中的容器myrunoob

```
docker stop myrunoob
```
重启容器myrunoob

```
docker restart myrunoob
```
