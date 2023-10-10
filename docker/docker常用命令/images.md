---
title: images
description: 
published: true
date: 2022-10-30T05:50:02.431Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:50:02.431Z
---

# docker images 命令

**docker images** : 列出本地镜像。

## 语法

```
docker images [OPTIONS] [REPOSITORY[:TAG]]
```
OPTIONS说明：

- -a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；

- --digests :显示镜像的摘要信息；

- -f :显示满足条件的镜像；

- --format :指定返回值的模板文件；

- --no-trunc :显示完整的镜像信息；

- -q :只显示镜像ID。

## 实例
查看本地镜像列表。
```
runoob@runoob:~$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
mymysql                 v1                  37af1236adef        5 minutes ago       329 MB
runoob/ubuntu           v4                  1c06aa18edee        2 days ago          142.1 MB
<none>                  <none>              5c6e1090e771        2 days ago          165.9 MB
httpd                   latest              ed38aaffef30        11 days ago         195.1 MB
alpine                  latest              4e38e38c8ce0        2 weeks ago         4.799 MB
mongo                   3.2                 282fd552add6        3 weeks ago         336.1 MB
redis                   latest              4465e4bcad80        3 weeks ago         185.7 MB
php                     5.6-fpm             025041cd3aa5        3 weeks ago         456.3 MB
python                  3.5                 045767ddf24a        3 weeks ago         684.1 MB
...
```

列出本地镜像中REPOSITORY为ubuntu的镜像列表。
```
root@runoob:~# docker images  ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               90d5884b1ee0        9 weeks ago         188 MB
ubuntu              15.10               4e3b13c8a266        3 months ago        136.3 MB
```
