---
title: PostgreSQL性能指标
description: 
published: true
date: 2022-11-29T10:04:04.730Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T10:04:04.730Z
---

# PostgreSQL性能指标

| 指标别名                  | 指标含义解释                                                 | 单位    |
| ------------------------- | ------------------------------------------------------------ | ------- |
| 运行时长                  | 该组件的运行时长                                             | s       |
| 版本                      | PostgreSQL版本                                               |         |
| 角色                      | PostgreSQL角色，Master/Slave                                 |         |
| 可用性                    | PostgreSQL的可用性，正常/异常/未监控                         |         |
| CPU使用率                 | 进程的CPU使用率                                              | %       |
| 内存使用量                | 进程的内存使用大小                                           | Bytes   |
| 内存使用率                | 进程内存使用量/主机内存总量                                  | %       |
| 磁盘吞吐(Read)            | 进程每秒读取的磁盘流量                                       | Bytes/s |
| 磁盘吞吐(Write)           | 进程每秒写入的磁盘流量                                       | Bytes/s |
| 流量(Send)                | MySQL每秒发送的字节数                                        | Bytes/s |
| 流量(Receive)             | MySQL每秒收到的字节数                                        | Bytes/s |
| 吞吐率(QPS)               | 数据库每秒执行的SQL数(增删改查)                              | 次/s    |
| 吞吐率(TPS)               | 数据库每秒执行的事物数(commit+rollback)                      | 次/s    |
| 吞吐率详情                | 数据库每秒执行insert、delete、update、select、commit、rollback的次数 | 次/s    |
| 活跃连接数                | 数据库当前的活跃连接数                                       |         |
| 活跃连接数占比            | 活跃连接数/连接数最大值                                      | %       |
| 连接数详情                | 数据库当前的所有连接状态                                     |         |
| 数据库容量                | 数据库所占字节数，以库为维度                                 | Bytes   |
| 临时表数                  | 每秒钟临时表的数量，以库为维度                               | 个/s    |
| 死锁中断查询数            | 每秒钟因死锁中断的查询数                                     | 个/s    |
| 锁数量                    | 每秒钟产生锁的数量                                           | 个/s    |
| 死锁数量                  | 每秒钟产生死锁的数量                                         | 个/s    |
| 平均响应时间(Read)        | 读操作平均耗时                                               | ms      |
| 平均响应时间(Write)       | 写操作平均耗时                                               | ms      |
| Buffer Cache命中率        | 数据库高速缓冲区缓存命中率                                   | %       |
| 索引命中率                | 数据库索引命中率                                             | %       |
| Toast表Buffer Cache命中率 | Toast表Buffer Cache命中率                                    | %       |
| Toast表索引命中率         | Toast表索引命中率                                            | %       |
| 主从同步延迟（逻辑复制）  | 逻辑复制的主从延迟，值越小延迟越低                           | Bytes   |
| 主从同步延迟（物理复制）  | 物理复制的主从延迟，值越小延迟越低                           | Bytes   |

