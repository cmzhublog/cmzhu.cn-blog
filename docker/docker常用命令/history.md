---
title: history
description: 
published: true
date: 2022-10-30T05:57:46.518Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:57:46.518Z
---

# docker history 命令

**docker history** : 查看指定镜像的创建历史。

## 语法
```
docker history [OPTIONS] IMAGE
```

OPTIONS说明：

- -H :以可读的格式打印镜像大小和日期，默认为true；

- --no-trunc :显示完整的提交记录；

- -q :仅列出提交记录ID。

## 实例
查看本地镜像runoob/ubuntu:v3的创建历史。
```
root@runoob:~# docker history runoob/ubuntu:v3
IMAGE             CREATED           CREATED BY                                      SIZE      COMMENT
4e3b13c8a266      3 months ago      /bin/sh -c #(nop) CMD ["/bin/bash"]             0 B                 
<missing>         3 months ago      /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.863 kB            
<missing>         3 months ago      /bin/sh -c set -xe   && echo '#!/bin/sh' > /u   701 B               
<missing>         3 months ago      /bin/sh -c #(nop) ADD file:43cb048516c6b80f22   136.3 MB
```
