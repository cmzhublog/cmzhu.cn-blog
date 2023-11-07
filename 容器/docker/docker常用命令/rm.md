---
title: rm
description: docker rm
published: true
date: 2022-10-30T01:02:08.861Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:02:08.861Z
---

# docker rm 指令

**docker rm** ：删除一个或多个容器。

## 语法
```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

OPTIONS说明：

- -f :通过 SIGKILL 信号强制删除一个运行中的容器。

- -l :移除容器间的网络连接，而非容器本身。

- -v :删除与容器关联的卷。

## 实例
强制删除容器 db01、db02：
```
docker rm -f db01 db02
```

移除容器 nginx01 对容器 db01 的连接，连接名 db：
```
docker rm -l db 
```
删除容器 nginx01, 并删除容器挂载的数据卷：

```
docker rm -v nginx01
```
删除所有已经停止的容器：

```
docker rm $(docker ps -a -q)
```