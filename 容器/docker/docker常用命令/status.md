---
title: status
description: 
published: true
date: 2022-10-30T01:49:28.007Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:49:28.007Z
---

# docker status 命令
	
**docker stats** : 显示容器资源的使用情况，包括：CPU、内存、网络 I/O 等。

## 语法

```
docker stats [OPTIONS] [CONTAINER...]
```

OPTIONS 说明：

- --all , -a :显示所有的容器，包括未运行的。

- --format :指定返回值的模板文件。

- --no-stream :展示当前状态就直接退出了，不再实时更新。

- --no-trunc :不截断输出。

## 实例
列出所有在运行的容器信息。

```
runoob@runoob:~$  docker stats
CONTAINER ID        NAME                                    CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
b95a83497c91        awesome_brattain                        0.28%               5.629MiB / 1.952GiB   0.28%               916B / 0B           147kB / 0B          9
67b2525d8ad1        foobar                                  0.00%               1.727MiB / 1.952GiB   0.09%               2.48kB / 0B         4.11MB / 0B         2
e5c383697914        test-1951.1.kay7x1lh1twk9c0oig50sd5tr   0.00%               196KiB / 1.952GiB     0.01%               71.2kB / 0B         770kB / 0B          1
4bda148efbc0        random.1.vnc8on831idyr42slu578u3cr      0.00%               1.672MiB / 1.952GiB   0.08%               110kB / 0B          578kB / 0B          2
```


输出详情介绍：

- CONTAINER ID 与 NAME: 容器 ID 与名称。

- CPU % 与 MEM %: 容器使用的 CPU 和内存的百分比。

- MEM USAGE / LIMIT: 容器正在使用的总内存，以及允许使用的内存总量。

- NET I/O: 容器通过其网络接口发送和接收的数据量。

- BLOCK I/O: 容器从主机上的块设备读取和写入的数据量。

- PIDs: 容器创建的进程或线程数。

根据容器等 ID 或名称现实信息：

```
runoob@runoob:~$ docker stats awesome_brattain 67b2525d8ad1

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
b95a83497c91        awesome_brattain    0.28%               5.629MiB / 1.952GiB   0.28%               916B / 0B           147kB / 0B          9
67b2525d8ad1        foobar              0.00%               1.727MiB / 1.952GiB   0.09%               2.48kB / 0B         4.11MB / 0B         2
```

以 JSON 格式输出：
```
runoob@runoob:~$ docker stats nginx --no-stream --format "{{ json . }}"
  {"BlockIO":"0B / 13.3kB","CPUPerc":"0.03%","Container":"nginx","ID":"ed37317fbf42","MemPerc":"0.24%","MemUsage":"2.352MiB / 982.5MiB","Name":"nginx","NetIO":"539kB / 606kB","PIDs":"2"}
```

输出指定的信息：

```
runoob@runoob:~$ docker stats --all --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}" fervent_panini 5acfcb1b4fd1 drunk_visvesvaraya big_heisenberg
  {"BlockIO":"0B / 13.3kB","CPUPerc":"0.03%","Container":"nginx","ID":"ed37317fbf42","MemPerc":"0.24%","MemUsage":"2.352MiB / 982.5MiB","Name":"nginx","NetIO":"539kB / 606kB","PIDs":"2"}

CONTAINER                CPU %               MEM USAGE / LIMIT
fervent_panini           0.00%               56KiB / 15.57GiB
5acfcb1b4fd1             0.07%               32.86MiB / 15.57GiB
drunk_visvesvaraya       0.00%               0B / 0B
big_heisenberg           0.00%               0B / 0B
```