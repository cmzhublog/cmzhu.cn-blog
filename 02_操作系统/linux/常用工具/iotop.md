---
title: iotop
description: 
published: true
date: 2023-10-09T02:08:50.662Z
tags: linux
editor: markdown
dateCreated: 2023-10-09T02:08:50.662Z
---

## iotop 学习

用来监视磁盘`I/O`使用状况的工具

### 补充说明

**`iotop`命令** 是一个用来监视磁盘`I/O`使用状况的`top`类工具。`iotop`具有与`top`相似的`UI`，其中包括`PID`、用户、`I/O`、进程等相关信息。`Linux`下的`IO`统计工具如`iostat`，`nmon`等大多数是只能统计到`per`设备的读写情况，如果你想知道每个进程是如何使用IO的就比较麻烦，使用`iotop`命令可以很方便的查看。

`iotop`使用`Python`语言编写而成，要求`Python2.5`（及以上版本）和`Linux kernel2.6.20`（及以上版本）。`iotop`提供有源代码及`rpm`包，可从其官方主页下载。



### 安装

```bash
dnf install iotop -y 
```

#### 编译安装

```bash
wget http://guichaz.free.fr/iotop/files/iotop-0.4.4.tar.gz    
tar zxf iotop-0.4.4.tar.gz    
python setup.py build    
python setup.py install
```

### 语法

```bash
iotop [选项]
```

#### 选项

```bash
-o：只显示有io操作的进程
-b：批量显示，无交互，主要用作记录到文件。
-n NUM：显示NUM次，主要用于非交互式模式。
-d SEC：间隔SEC秒显示一次。
-P PID：监控的进程pid。
-u USER：监控的进程用户。
```

#### 快捷键

```bash
左右箭头：改变排序方式，默认是按IO排序。
r：改变排序顺序。
o：只显示有IO输出的进程。
p：进程/线程的显示方式的切换。
a：显示累积使用量。
q：退出。
```

