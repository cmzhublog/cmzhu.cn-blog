---
title: Apache HTTP Server性能指标
description: 
published: true
date: 2022-11-29T09:37:15.421Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:37:15.421Z
---

# Apache HTTP Server性能指标

| 指标别名        | 指标含义解释                                          | 单位    |
| --------------- | ----------------------------------------------------- | ------- |
| 运行时长        | 该组件的运行时长                                      | s       |
| CPU使用率       | 进程的CPU使用率                                       | %       |
| 内存使用量      | 进程的内存使用大小                                    | Bytes   |
| 内存使用率      | 进程内存使用量/主机内存总量                           | %       |
| 磁盘吞吐(Read)  | 进程每秒读取的磁盘流量                                | Bytes/s |
| 磁盘吞吐(Write) | 进程每秒写入的磁盘流量                                | Bytes/s |
| 流量(Sent)      | apache每秒传输的字节数                                | Bytes/s |
| 吞吐率(QPS)     | 该Apache服务每秒的请求数                              | 次/s    |
| 活跃线程数      | 该Apache服务当前活跃的线程数                          |         |
| 空闲线程数      | 该Apache服务当前空闲的线程数                          |         |
| 可用性          | 该Apache服务的可用性，正常/异常/未监控                |         |
| 版本            | Apache的版本                                          |         |
| 工作模式        | Apache服务的工作模式,Prefork MPM/Worker MPM/Event MPM |         |
| 连接数详情      | Apache处于Open、Sending、Reading、等状态的连接数      |         |

