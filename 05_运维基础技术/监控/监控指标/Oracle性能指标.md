---
title: Oracle性能指标
description: 
published: true
date: 2022-11-29T09:43:41.849Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:43:41.849Z
---

# Oracle性能指标

| 指标别名                | 指标含义解释                                     | 单位    |
| ----------------------- | ------------------------------------------------ | ------- |
| 运行时长                | 该组件的运行时长                                 | s       |
| 版本                    | MySQL版本                                        |         |
| 可用性                  | Oracle的可用性，正常/异常/未监控                 |         |
| CPU使用率               | 进程的CPU使用率                                  | %       |
| 内存使用量              | 进程的内存使用大小                               | Bytes   |
| 内存使用率              | 进程内存使用量/主机内存总量                      | %       |
| 磁盘吞吐(Read)          | 进程每秒读取的磁盘流量                           | Bytes/s |
| 磁盘吞吐(Write)         | 进程每秒写入的磁盘流量                           | Bytes/s |
| 吞吐率(TPS)             | 数据库每秒传输的事物数，等于Commit+Rollback      | 次/s    |
| 吞吐率(QPS)             | 数据库每秒执行的SQL数，等于execute_cout          | 次/s    |
| 吞吐率详情              | 每秒钟执行parse、execute、commit、rollback的次数 | 次/s    |
| 入队列超时数            | 数据库实例每秒入队列超时数                       | 次/s    |
| 活跃Session数           | 数据库实例当前活跃的Session数                    | 个      |
| 非活跃Session数         | 数据库实例当前非活跃的Session数                  | 个      |
| 系统Session数           | 数据库实例限制的Session的最大值                  | 个      |
| Session使用率           | (非活跃Session数+活跃Session数)/系统Session数    | %       |
| 等待时间占比            |                                                  | %       |
| 内存排序占比            |                                                  | %       |
| 平均响应时间            |                                                  | ms      |
| 归档日志大小            | 实例归档日志的大小                               | Bytes   |
| 数据库容量              | 实例的数据库容量                                 | Bytes   |
| 等待的事件数            | 每秒钟等待的事件数                               | 次/s    |
| SGA shared pool         | 共享池的大小                                     | Bytes   |
| SGA log buffer          | 日志高速缓存大小                                 | Bytes   |
| SGA large pool          | Large池的大小                                    | Bytes   |
| SGA java pool           | Java 池的大小                                    | Bytes   |
| SGA fixed               | 固定SGA区的大小                                  | Bytes   |
| SGA buffer cache        | 高速缓存的大小                                   | Bytes   |
| Share Pool sql area     | SQL语句缓存区的大小                              | Bytes   |
| Share Pool misc         | 混合使用区的大小                                 | Bytes   |
| Share Pool lib cache    | 库的高速缓存大小                                 | Bytes   |
| Share Pool free memory  | 空闲的缓存大小                                   | Bytes   |
| Share Pool dict cache   | 数据字典缓存区大小                               | Bytes   |
| Buffer Cache命中率      | 高速缓存命中率                                   | %       |
| Cursor Cache命中率      | 游标缓存命中率                                   | %       |
| Library Cache命中率     | Library                                          | %       |
| 已用PGA                 | 已经申请的PGA大小                                | Bytes   |
| PGA总容量               | PGA内存限制大小                                  | Bytes   |
| PGA使用率               | PGA                                              | %       |
| Memory                  | 每秒钟Redo log增长量                             | Bytes/s |
| 表空间使用量            | 当前表空间的使用字节数                           | Bytes   |
| 表空间最大量            | 当前表空间限制的最大字节数                       | Bytes   |
| 表空间使用率            | 表空间使用量/表空间最大量                        | %       |
| 临时表空间使用量        | 临时表空间的使用字节数                           | Bytes   |
| 磁盘排序                | 平均每秒钟磁盘排序的次数                         | 次/s    |
| 全表扫描                | 平均每秒钟全表扫描的次数                         | 次/s    |
| 物理读                  | 平均每秒钟物理读次数                             | 次/s    |
| 物理写                  | 平均每秒钟物理写次数                             | 次/s    |
| Redo写                  | 平均每秒钟Redo写入次数                           | 次/s    |
| 逻辑IO(Current Read)    | 平均每秒钟实时读的次数                           | 次/s    |
| 逻辑IO(Consistent Read) | 平均每秒钟一致性读的次数                         | 次/s    |
| 逻辑IO(Block Change)    | 平均每秒钟块修改的次数                           | 次/s    |

