---
title: Nginx性能指标
description: 
published: true
date: 2022-11-29T09:38:20.965Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:38:20.965Z
---

# Nginx性能指标

| 指标别名        | 指标含义解释                                  | 单位    |
| --------------- | --------------------------------------------- | ------- |
| 运行时长        | 该进程运行的时长                              | s       |
| CPU使用率       | 进程的CPU使用率                               | %       |
| 内存使用量      | 进程的内存使用大小                            | Bytes   |
| 内存使用率      | 进程内存使用量/主机内存总量                   | %       |
| 磁盘吞吐(Read)  | 进程每秒读取的磁盘流量                        | Bytes/s |
| 磁盘吞吐(Write) | 进程每秒写入的磁盘流量                        | Bytes/s |
| 吞吐率(QPS)     | 该Nginx服务每秒的请求数                       | 次/s    |
| 连接数详情      | Active、Reading、Waiting、Writing状态的连接数 |         |
| 可用性          | 该Apache服务的可用性，正常/异常/未监控        |         |

