---
datitle: Zookeeper性能指标
description: 
published: true
date: 2022-11-30T02:07:58.634Z
tags: monitor
editor: markdown
dateCreated: 2022-11-30T02:07:58.634Z
---

# Zookeeper性能指标

## 集群指标

| 指标别名     | 指标含义解释                     | 单位 |
| :----------- | :------------------------------- | :--- |
| Quorum       | 参加选举的节点数                 | 个   |
| Standalone   | 单机模式                         |      |
| Leader数量   | Leader数量，sum                  | 个   |
| Follower数量 | Follower数量，sum                | 个   |
| ObServer数量 | ObServer数量，sum                | 个   |
| 活跃连接数   | 集群所有节点连接数之和           | 个   |
| 平均请求延迟 | 集群所有节点平均请求延迟的平均值 | tick |
| 最大请求延迟 | 集群所有节点最大请求延迟的最大值 | tick |
| 可用性       | 1-正常，2-警告，3-异常           |      |

## 节点指标

| 指标别名                        | 指标含义解释                                      | 单位    |
| :------------------------------ | :------------------------------------------------ | :------ |
| 发送数据包                      | 每秒钟发送的数据包数量                            | 个/s    |
| 接收数据包                      | 每秒钟接收的数据包数量                            | 个/s    |
| 排队请求数                      | 排队请求的个数                                    | 个      |
| 活跃连接数                      | 当前的活跃连接数量                                | 个      |
| 最大请求延迟                    | 该实例最大的请求延迟时间                          | tick    |
| 平均请求延迟                    | 该实例的平均请求延迟时间                          | tick    |
| watcher数量                     | Watcher的数量                                     | 个      |
| znode数量                       | Znode的数量                                       | 个      |
| 文件打开数占比                  | 已经打开的文件数/进程限制的打开文件数             | %       |
| CPU使用率                       | 进程的CPU使用率                                   | %       |
| 内存使用量                      | 进程的内存使用量                                  | Bytes   |
| 内存使用率                      | 进程的内存使用量/主机的内存总量                   | %       |
| 磁盘吞吐(Read)                  | 进程的磁盘每秒读取数据量                          | Bytes/s |
| 磁盘吞吐(Write)                 | 进程的磁盘每秒写入数据量                          | Bytes/s |
| 堆内存                          | 堆内存的init、used、commit、max值                 | Bytes   |
| 堆内存/Eden Space               | Eden Space的init、used、commit、max值             | Bytes   |
| 堆内存/Survival Space           | Survival Space 的init、used、commit、max值        | Bytes   |
| 堆内存/Old Gen                  | Old Gen的init、used、commit、max值                | Bytes   |
| 非堆内存                        | 非堆内存的init、used、commit、max值               | Bytes   |
| 非堆内存/Perm Gen               | Perm Gen的init、used、commit、max值               | Bytes   |
| 非堆内存/Metaspace              | Metaspace的init、used、commit、max值              | Bytes   |
| 非堆内存/Code Cache             | Code Cache的init、used、commit、max值             | Bytes   |
| 非堆内存/Compressed Class Space | Compressed Class Space的init、used、commit、max值 | Bytes   |
| GC 次数                         | Minor GC、Major GC平均每秒次数                    | 次/s    |
| GC 耗时                         | MinorGC、Major GC平均每秒耗时                     | ms      |
| JVM线程                         | current、daemon、deadlocked线程数                 | 个      |
| 堆内存使用率                    | 已用堆内存/堆内存最大值                           | %       |
| 非堆内存使用率                  | 已用非堆内存/非堆内存最大值                       | %       |

