---
title: IO 调度优先级设置
description: 
published: true
date: 2023-02-17T02:46:47.294Z
tags: linux
editor: markdown
dateCreated: 2023-02-17T02:46:47.294Z
---

# 磁盘IO 优化



## ionice

命令功能:

ionice – 获取或设置程序的IO调度与优先级。

```
参数说明：

-c class ：class表示调度策略，其中0 for none, 1 for real time, 2 for best-effort, 3 for idle。
-n classdata：classdata表示IO优先级级别，对于best effort和real time，classdata可以设置为0~7。
-p pid：指定要查看或设置的进程号或者线程号，如果没有指定pid参数，ionice will run the listed program with the given parameters。
-t ：忽视设置优先级时产生的错误。
```

```
命令格式：
ionice [[-c class] [-n classdata] [-t]] -p PID [PID]…
 
ionice [-c class] [-n classdata] [-t] COMMAND [ARG]…
```

###  例子

```
[root@linux ~]# ionice -c 3 -p 89  #设置进程号为89的进程的调度策略是idle

[root@linux ~]# ionice -c 2 -n 0 bash  #运行bash，调度策略是best-effort，最高优先级

[root@linux ~]# ionice -p 89 91  #打印进程号为89和91进程的调度策略和IO优先级

[root@linux ~]# ionice -c3 -p$$  #将当前的进程(就是shell)磁盘IO调度策略设置为idle类型
```