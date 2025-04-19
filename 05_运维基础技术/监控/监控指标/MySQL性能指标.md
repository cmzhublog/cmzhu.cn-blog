---
title: MySQL性能指标
description: 
published: true
date: 2022-11-29T09:42:34.211Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:42:34.211Z
---

# MySQL性能指标

| 指标别名            | 指标含义解释                                         | 单位    |
| ------------------- | ---------------------------------------------------- | ------- |
| 运行时长            | 该组件的运行时长                                     | s       |
| 角色                | MySQL角色，Master/Slave                              |         |
| 版本                | MySQL版本                                            |         |
| 可用性              | Redis的可用性，正常/异常/未监控                      |         |
| CPU使用率           | 进程的CPU使用率                                      | %       |
| 内存使用量          | 进程的内存使用大小                                   | Bytes   |
| 内存使用率          | 进程内存使用量/主机内存总量                          | %       |
| 磁盘吞吐(Read)      | 进程每秒读取的磁盘流量                               | Bytes/s |
| 磁盘吞吐(Write)     | 进程每秒写入的磁盘流量                               | Bytes/s |
| 流量(Send)          | MySQL每秒发送的字节数                                | Bytes/s |
| 流量(Receive)       | MySQL每秒收到的字节数                                | Bytes/s |
| 吞吐率(TPS)         | 数据库每秒传输的事物数                               | 次/s    |
| 吞吐率(QPS)         | 数据库每秒执行的SQL数                                | 次/s    |
| 活跃连接数          | 数据库当前活跃的连接数                               |         |
| 当前连接数          | 数据库当前打开的连接数                               |         |
| 最大连接数          | 数据库最大的连接数                                   |         |
| 连接数占比          | 当前连接数/最大连接数                                | %       |
| 活跃连接数占比      | 活跃连接数/最大连接数                                |         |
| 线程状态            | 数据库当前线程状态的数量统计，类似于show processlist |         |
| 文件打开数占比      | 数据库当前打开文件数/数据库最大打开文件数            | %       |
| Aborted Connections | Aborted Connect，一般发生在网络不稳定的情况          | 次/s    |
| 连接超时            | Connect Timeout，连接超时                            | 次/s    |
| Aborted clients     | Aborted Clients，一般是连接数已超，或者密码不正确    | 次/s    |
| kill执行频率        | 数据库中kill命令的执行频率                           | 次/s    |
| 请求详情            | 数据库中各种请求的QPS，包括select，insert等          | 次/s    |
| 慢查询              | 每秒钟慢查询的次数                                   | 次/s    |
| 平均响应时间        | 数据库整体的平均响应时间                             | ms      |
| 响应时间详情        | insert、update、select、delete的平均响应时间         | ms      |
| 内部错误            | 每秒钟数据库内部所有错误的次数                       | 次/s    |
| 内部错误详情        | 每秒钟数据库内部错误的详情，包括accept、internal等   | 次/s    |
| 表大小              | 每个表的存储容量                                     | Bytes   |
| 表行数              | 每个表的行数                                         | 行      |
| 等待表锁次数        | 不能立刻获得表的锁的次数                             | 次/s    |
| 临时表数量          | 每秒创建临时表的次数                                 | 次/s    |
| 临时文件数量        | 每秒创建临时文件的次数                               | 次/s    |
| 磁盘临时表数量      | 每秒创建磁盘临时表的次数                             | 次/s    |
| InnoDB缓存命中率    | InnoDB引擎的缓存命中率                               | %       |
| InnoDB缓存使用率    | InnoDB引擎的缓存使用率                               | %       |
| InnoDB读磁盘次数    | InnoDB引擎每秒读磁盘文件的次数                       | 次/s    |
| InnoDB写磁盘次数    | InnoDB引擎每秒写入文件磁盘文件的次数                 | 次/s    |
| InnoDB fsync次数    | InnoDB引擎每秒调用fsync函数的次数                    | 次/s    |
| InnoDB打开表数      | InnoDB引擎当前打开表的数量                           | 个      |
| InnoDB空页数        | InnoDB引擎内存空页个数                               | 个      |
| InnoDB总页数        | InnoDB引擎占用内存总页数                             | 个      |
| InnoDB逻辑读        | InnoDB每秒完成逻辑读的次数                           | 次/s    |
| InnoDB物理读        | InnoDB每秒完成物理读的次数                           | 次/s    |
| InnoDB读取流量      | InnoDB每秒读取的字节数                               | Bytes/s |
| InnoDB读取量        | InnoDB每秒读取的次数                                 | 次/s    |
| InnoDB写入流量      | InnoDB每秒写入的字节数                               | Bytes/s |
| Innodb写入量        | InnoDB每秒写入的次数                                 | 次/s    |
| InnoDB行插入量      | InnoDB引擎每秒插入的行数                             | 次/s    |
| InnoDB 行更新量     | InnoDB引擎每秒更新的行数                             | 次/s    |
| InnoDB行删除量      | InnoDB引擎每秒删除的行数                             | 次/s    |
| InnoDB行读取量      | InnoDB引擎每秒读取的行数                             | 次/s    |
| InnoDB获取行锁时间  | InnoDB引擎锁定的平均时长                             | ms      |
| InnoDB等待行锁次数  | InnoDB引擎每秒等待行锁的次数                         | 次/s    |
| MyISAM缓存命中率    | MyISAM引擎的缓存命中率                               | %       |
| MyISAM缓存使用率    | MyISAM引擎的缓存使用率                               | %       |
| 主从延迟时间        | 主从延迟时间                                         | s       |
| 主从延迟距离        | 主从binlog差距                                       | MB      |
| IO线程状态          | IO线程运行状态(0-YES,1-No,2-Connecting)              |         |
| SQL线程状态         | SQL线程运行状态(0-Yes,1-No)                          |         |

