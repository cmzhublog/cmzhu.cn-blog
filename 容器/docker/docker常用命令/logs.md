---
title: logs
description: 
published: true
date: 2022-10-30T01:35:49.110Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:35:49.110Z
---

# docker logs 命令

**docker logs** : 获取容器的日志

## 语法

```
docker logs [OPTIONS] CONTAINER
```

OPTIONS说明：

- -f : 跟踪日志输出

- --since :显示某个开始时间的所有日志

- -t : 显示时间戳

- --tail :仅列出最新N条容器日志

## 实例
跟踪查看容器mynginx的日志输出。
```
runoob@runoob:~$ docker logs -f mynginx
192.168.239.1 - - [10/Jul/2016:16:53:33 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
2016/07/10 16:53:33 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.239.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.239.130", referrer: "http://192.168.239.130/"
192.168.239.1 - - [10/Jul/2016:16:53:33 +0000] "GET /favicon.ico HTTP/1.1" 404 571 "http://192.168.239.130/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
192.168.239.1 - - [10/Jul/2016:16:53:59 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" "-"
...

```

查看容器mynginx从2016年7月1日后的最新10条日志。
```
docker logs --since="2016-07-01" --tail=10 mynginx
```
