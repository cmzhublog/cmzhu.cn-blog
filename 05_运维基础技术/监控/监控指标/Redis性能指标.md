---
title: Redis性能指标
description: 
published: true
date: 2022-11-29T09:40:55.539Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:40:55.539Z
---

# Redis性能指标

| 指标别名                  | 指标含义解释                                                 | 单位    |
| ------------------------- | ------------------------------------------------------------ | ------- |
| 运行时长                  | 该组件的运行时长                                             | s       |
| 角色                      | Redis角色，Master/Slave                                      |         |
| 版本                      | Redis实例的版本                                              |         |
| 可用性                    | Redis的可用性，正常/异常/未监控                              |         |
| CPU使用率(User)           | 进程的CPU使用率                                              | %       |
| CPU使用率(Sys)            | Redis主进程内核态CPU使用率                                   | %       |
| CPU使用率(UserChildren)   | Redis后台进程用户态CPU使用率                                 | %       |
| CPU使用率(SysChildren)    | Redis后台进程内核态CPU使用率                                 | %       |
| CPU使用率                 | Reids组件的CPU使用率,公式:CPU使用率(User)+CPU使用率(Sys)+CPU使用率(UserChildren)+CPU使用率(SysChildren) | %       |
| 内存使用量                | Redis实例实际申请的内存大小                                  | Bytes   |
| 内存使用量(RSS)           | Redis实例实际占用的操作系统的内存大小                        | Bytes   |
| Redis内存最大值           | Redis实例设置的内存最大值                                    | Bytes   |
| 内存使用率                | Redis内存最大值未配置，=Redis内存使用量/主机内存总量；Redis内存最大值已配置，=Redis内存使用量/Redis内存最大值 | %       |
| 内存碎片率                | 内存使用量/内存使用量(RSS)                                   | %       |
| 磁盘吞吐(Read)            | 进程每秒读取的磁盘流量                                       | Bytes/s |
| 磁盘吞吐(Write)           | 进程每秒写入的磁盘流量                                       | Bytes/s |
| 流量(Send)                | Redis每秒发送的字节数                                        | Bytes/s |
| 流量(Receive)             | Redis每秒接收的字节数                                        | Bytes/s |
| 并发连接数                | Redis实例的并发连接数                                        |         |
| 阻塞连接数                | Redis实例阻塞的连接数                                        |         |
| Slave连接数               | Redis实例的Slave连接数                                       |         |
| 拒绝连接数                | Reids实例拒绝的连接数                                        |         |
| 拒绝连接速率              | Reids实例每秒的拒绝连接数                                    | 个/s    |
| 吞吐率(QPS)               | Redis实例每秒执行的命令数                                    | 次/s    |
| 吞吐率详情                | Redis实例每秒执行的命令数详情，包含set、get、del等           | 次/s    |
| 命中次数                  | 成功匹配key的次数                                            | 次/s    |
| 未命中次数                | 未成功匹配key的次数                                          | 次/s    |
| 缓存命中率                | 命中数次数/(命中次数+未命中次数)                             | %       |
| Key数量                   | Redis实例Key总个数                                           | 个      |
| Expiring Key数量          | Redis实例可过期Key的总个数                                   | 个      |
| Not-Expiring Key数量      | Redis实例不可过期Key的总个数                                 | 个      |
| Key过期数                 | Redis实例每秒过期个数，对应info输出的expired_keys            | 个/s    |
| Key驱逐数                 | Redis实例每秒驱逐的个数，对应Info输出的evicted_keys          | 个/s    |
| get平均响应时间           | get请求的平均响应时间                                        | ms      |
| set平均响应时间           | set请求的平均响应时间                                        | ms      |
| 最后一次RDB备份的时间点   | Redis实例最后一次RDB备份的时间点                             |         |
| 最后一次RDB备份的状态     | Redis实例最后一次RDB备份的状态                               |         |
| 最后一次RDB备份花费的时间 | Redis实例最后一次RDB备份花费的时间                           |         |
| 是否开启了AOF             | Redis实例是否开启了AOF                                       |         |
| 当前AOF重写花费的时间     | Redis实例正在进行AOF重写花费的时间                           |         |
| 最后一次AOF重写话费的时间 | Redis实例最后一次AOF重写花费的时间                           |         |

