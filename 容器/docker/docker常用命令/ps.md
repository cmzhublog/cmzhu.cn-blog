---
title: ps
description: 
published: true
date: 2022-10-30T01:21:11.179Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:21:11.179Z
---

# docker ps 命令

**docker ps** : 列出容器

## 语法

```
docker ps [OPTIONS]
```

OPTIONS说明：

- -a :显示所有的容器，包括未运行的。

- -f :根据条件过滤显示的内容。

- --format :指定返回值的模板文件。

- -l :显示最近创建的容器。

- -n :列出最近创建的n个容器。

- --no-trunc :不截断输出。

- -q :静默模式，只显示容器编号。

- -s :显示总的文件大小。

## 实例
列出所有在运行的容器信息。

```
runoob@runoob:~$ docker ps
CONTAINER ID   IMAGE          COMMAND                ...  PORTS                    NAMES
09b93464c2f7   nginx:latest   "nginx -g 'daemon off" ...  80/tcp, 443/tcp          myrunoob
96f7f14e99ab   mysql:5.6      "docker-entrypoint.sh" ...  0.0.0.0:3306->3306/tcp   mymysql

```
输出详情介绍：

**CONTAINER ID**: 容器 ID。

**IMAGE**: 使用的镜像。

**COMMAND**: 启动容器时运行的命令。

**CREATED**: 容器的创建时间。

**STATUS**: 容器状态。

状态有7种：

- created（已创建）
- restarting（重启中）
- running（运行中）
- removing（迁移中）
- paused（暂停）
- exited（停止）
- dead（死亡）
- PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。

NAMES: 自动分配的容器名称。

列出最近创建的5个容器信息。

```
runoob@runoob:~$ docker ps -n 5
CONTAINER ID        IMAGE               COMMAND                   CREATED           
09b93464c2f7        nginx:latest        "nginx -g 'daemon off"    2 days ago   ...     
b8573233d675        nginx:latest        "/bin/bash"               2 days ago   ...     
b1a0703e41e7        nginx:latest        "nginx -g 'daemon off"    2 days ago   ...    
f46fb1dec520        5c6e1090e771        "/bin/sh -c 'set -x \t"   2 days ago   ...   
a63b4a5597de        860c279d2fec        "bash"                    2 days ago   ...

```

列出所有创建的容器ID。

```
runoob@runoob:~$ docker ps -a -q
09b93464c2f7
b8573233d675
b1a0703e41e7
f46fb1dec520
a63b4a5597de
6a4aa42e947b
de7bb36e7968
43a432b73776
664a8ab1a585
ba52eb632bbd
...
```