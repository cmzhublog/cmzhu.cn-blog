---
title: diff
description: 
published: true
date: 2022-10-30T05:01:50.992Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:01:50.992Z
---

# docker diff 命令

**docker diff** : 检查容器里文件结构的更改。

## 语法
```
docker diff [OPTIONS] CONTAINER
```

## 实例
查看容器mymysql的文件结构更改。
```
runoob@runoob:~$ docker diff mymysql
A /logs
A /mysql_data
C /run
C /run/mysqld
A /run/mysqld/mysqld.pid
A /run/mysqld/mysqld.sock
C /tmp
```
