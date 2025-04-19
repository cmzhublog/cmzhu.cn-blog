---
title: ActiveMQ性能指标
description: 
published: true
date: 2022-11-30T02:13:50.145Z
tags: monitor
editor: markdown
dateCreated: 2022-11-30T02:13:50.145Z
---

# ActiveMQ性能指标

| 指标别名               | 指标含义解释                          | 单位  |
| :--------------------- | :------------------------------------ | :---- |
| 运行时长               | 该组件的运行时长                      | s     |
| 版本                   | ActiveMQ版本                          |       |
| 可用性                 | ActiveMQ的可用性                      |       |
| CPU利用率              | 进程的CPU使用率                       | %     |
| 内存利用率             | 进程内存使用量/主机内存总量           | %     |
| 内存使用量             | 进程的内存使用大小                    | Bytes |
| 磁盘吞吐(Read)         | 进程每秒读取的磁盘流量                | Bytes |
| 磁盘吞吐(Write)        | 进程每秒写入的磁盘流量                | Bytes |
| 非持久性消息内存使用率 | 非持久性消息的内存限制使用占比        | %     |
| 存储使用率             | 持久性消息的存储限制使用百分比        | %     |
| temp使用率             | 临时消息的存储限制使用百分比          | %     |
| 当前连接数             | Broker的当前的连接数                  | 个    |
| 总连接数               | Broker启动以来总的连接数              | 个    |
| 总入队列数量           | Broker每秒入队列数量                  | 个/s  |
| 总出队列数量           | Broker每秒出队列数量                  | 个/s  |
| 总生产者数量           | Broker所有生产者的数量                | 个    |
| 总消费者数量           | Broker所有消费者的数量                | 个    |
| Queue数量              | Broker所有Queue的数量                 | 个    |
| Topic数量              | Broker所有Topic的数量                 | 个    |
| Queue平均响应时间      | Queue维度，消息在队列里保留的平均时间 | ms    |
| Queue未消费消息数      | Queue维度，消费者未消费的消息数       | 个    |
| Queue生产者数量        | Queue维度，生产者的数量               | 个    |
| Queue消费者数量        | Queue维度，消费者的数量               | 个    |
| Queue消息入队数        | Queue维度，每秒钟入队列的数量         | 个/s  |
| Queue消息出队数        | Queue维度，每秒钟出队列的数量         | 个/s  |
| Topic平均响应时间      | Topic维度，消息在队列里保留的平均时间 | ms    |
| Topic未消费消息数      | Topic维度，消费者未消费的消息数       | 个    |
| Topic生产者数量        | Topic维度，生产者的数量               | 个    |
| Topic消费者数量        | Topic维度，消费者的数量               | 个    |
| Topic消息入队数        | Topic维度，每秒钟入队列的数量         | 个/s  |
| Topic消息出队数        | Topic维度，每秒钟出队列的数量         | 个/s  |
| 堆内存使用量           | JVM中Heap区内存使用量                 | Bytes |
| 堆内存最大值           | JVM中Heap区设置的最大值               | Bytes |
| 堆内存利用率           | 堆内存使用量/堆内存最大值             | Bytes |
| GC 次数                | 每秒钟GC次数                          | 次/s  |
| GC 耗时                | 每秒钟GC耗时                          | ms    |

