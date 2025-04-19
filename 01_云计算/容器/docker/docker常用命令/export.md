---
title: export
description: 
published: true
date: 2022-10-30T01:44:03.033Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:44:03.033Z
---

# docker export 命令

**docker export** :将文件系统作为一个tar归档文件导出到STDOUT。

## 语法

```
docker export [OPTIONS] CONTAINER
```

OPTIONS说明：

- -o :将输入内容写到文件。

## 实例
将id为a404c6c174a2的容器按日期保存为tar文件。

```
runoob@runoob:~$ docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2
runoob@runoob:~$ ls mysql-`date +%Y%m%d`.tar
mysql-20160711.tar
```
