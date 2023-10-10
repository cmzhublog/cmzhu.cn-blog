---
title: top
description: 
published: true
date: 2022-10-30T01:26:38.550Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:26:38.550Z
---

# docker top 命令

**docker top** :查看容器中运行的进程信息，支持 ps 命令参数。

## 语法

```
docker top [OPTIONS] CONTAINER [ps OPTIONS]
```

容器运行时不一定有/bin/bash终端来交互执行top命令，而且容器还不一定有top命令，可以使用docker top来实现查看container中正在运行的进程。

## 实例
查看容器mymysql的进程信息。
```
runoob@runoob:~/mysql$ docker top mymysql
UID    PID    PPID    C      STIME   TTY  TIME       CMD
999    40347  40331   18     00:58   ?    00:00:02   mysqld
```

查看所有运行容器的进程信息。
```
for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top $i; done

```
