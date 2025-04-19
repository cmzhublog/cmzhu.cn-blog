---
title: version
description: 
published: true
date: 2022-10-30T06:05:33.001Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T06:05:33.001Z
---

# docker version命令

**docker version** :显示 Docker 版本信息。

## 语法
```
docker version [OPTIONS]
```
OPTIONS说明：

- -f :指定返回值的模板文件。

# 实例
显示 Docker 版本信息。
```
$ docker version
Client:
 Version:      1.8.2
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0a8c2e3
 Built:        Thu Sep 10 19:19:00 UTC 2015
 OS/Arch:      linux/amd64

Server:
 Version:      1.8.2
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0a8c2e3
 Built:        Thu Sep 10 19:19:00 UTC 2015
 OS/Arch:      linux/amd64
 ```
 