---
title: exec
description:  docker exec 指令
published: true
date: 2022-10-30T01:13:16.145Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:13:16.145Z
---

# docker exec 命令

**docker exec** ：在运行的容器中执行命令

## 语法
```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

OPTIONS说明：

- -d :分离模式: 在后台运行

- -i :即使没有附加也保持STDIN 打开

- -t :分配一个伪终端

## 实例
在容器 mynginx 中以交互模式执行容器内 /root/runoob.sh 脚本:

```
runoob@runoob:~$ docker exec -it mynginx /bin/sh /root/runoob.sh
http://www.runoob.com/
```

在容器 mynginx 中开启一个交互模式的终端:

```
runoob@runoob:~$ docker exec -i -t  mynginx /bin/bash
root@b1a0703e41e7:/#
```

也可以通过 docker ps -a 命令查看已经在运行的容器，然后使用容器 ID 进入容器。

查看已经在运行的容器 ID：

```
# docker ps -a 
...
9df70f9a0714        openjdk             "/usercode/script.sh…" 
...
```

第一列的 9df70f9a0714 就是容器 ID。

通过 exec 命令对指定的容器执行 bash:

```
# docker exec -it 9df70f9a0714 /bin/bash
```
