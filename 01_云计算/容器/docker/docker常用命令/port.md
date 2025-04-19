---
title: port
description: 
published: true
date: 2022-10-30T01:46:18.885Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:46:18.885Z
---

# docker port 用法

**docker port** :列出指定的容器的端口映射，或者查找将PRIVATE_PORT NAT到面向公众的端口。

## 语法

```
docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]
```

## 实例
查看容器mynginx的端口映射情况。

```
runoob@runoob:~$ docker port mymysql
3306/tcp -> 0.0.0.0:3306
```