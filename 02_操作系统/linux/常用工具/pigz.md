---
title: pigz用法
description: 
published: true
date: 2023-04-23T06:57:36.087Z
tags: linux
editor: markdown
dateCreated: 2023-04-23T06:57:36.087Z
---

# pigz

## 安装

```bash
dnf search pigz
dnf install -y pigz
```



## 用法

### 压缩文件

有下面几个常用参数

- `-p n` : 压缩使用的核心数,`n` 表示使用多少核; 默认使用所有核心
-  `-k` : 压缩后保留原文件
-  `-l`  : 列出压缩输入的内容
-  `-6` : 默认压缩级别
-  `-9` : 压缩率最高, 但是速度慢
-  `-1` : 压缩率最低, 但是速度快

例如, 压缩如果需要保留原文件, 可以使用; 会默认创建一个 `xxx.gz` 的文件

```bash
pigz -k NVIDIA-Linux-x86_64-525.89.02.run

[root@cmzhu ~]# ll | grep NVIDIA-Linux-x86_64-525.89.02
-rwxr--r--  1 root root 414116295 Feb  3 01:59 NVIDIA-Linux-x86_64-525.89.02.run
-rwxr--r--  1 root root 414167850 Feb  3 01:59 NVIDIA-Linux-x86_64-525.89.02.run.gz
[root@cmzhu ~]#
```

如果需要查看压缩了什么? 可以如下操作

```bash
[root@cmzhu ~]# pigz -l NVIDIA-Linux-x86_64-525.89.02.run.gz
compressed   original reduced  name
 414167798  414116295   -0.0%  NVIDIA-Linux-x86_64-525.89.02.run
[root@cmzhu ~]#
```



### 压缩目录

`pigz`没有压缩文件夹的功能, 只有压缩单个文件, pigz 可以和tar 命令一起使用, 来压缩文件夹;

```bash
tar -cvf - /var/log | pigz -k > var_log.tar.gz
```

```bash
[root@cmzhu ~]# pigz -l var_log.tar.gz
compressed   original reduced  name
 154655475 1785825280   91.3%  var_log.tar
[root@cmzhu ~]#
```

```bash
tar -cvf - /var/log | pigz -9  -k > var_log.tar.gz
[root@cmzhu ~]# pigz -l var_log.tar.gz
compressed   original reduced  name
 149322721 1786183680   91.6%  var_log.tar
[root@cmzhu ~]#
```

## 解压文件

### 解压单个文件

```bash
unpigz -d -k NVIDIA-Linux-x86_64-525.89.02.run.gz 

```



如果不需要保留压缩文件可以不用 `-k` 参数

```bash
unpigz -d NVIDIA-Linux-x86_64-525.89.02.run.gz 
```

### 解压一个目录

```bash
tar -xf var_log.tar.gz
```



