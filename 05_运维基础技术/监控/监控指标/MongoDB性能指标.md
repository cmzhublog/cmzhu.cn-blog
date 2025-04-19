---
title: MongoDB性能指标
description: 
published: true
date: 2022-11-29T10:07:11.319Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T10:07:11.319Z
---

# MongoDB性能指标

## 分片集群指标

| 指标别名                | 指标含义解释                                                 | 单位  |
| ----------------------- | ------------------------------------------------------------ | ----- |
| 连接数详情              | MongoDB集群连接数详情，mongos节点连接数需所有节点累加，configServer、shard都取Primary节点的连接数 |       |
| Cursor数量              | MongoDB集群Cursor数量，configServer、shard都取Primary节点的连接数 |       |
| 分配到分片上的表数量    | 集群中每个分片上的表数量，sum(metric_value) by shard         |       |
| 分配到分片上的表大小    | 集群中每个分片上的表大小，sum(metric_value) by shard         | Bytes |
| 分配到分片上的索引大小  | 集群中每个分片上的索引大小，sum(metric_value) by shard       | Bytes |
| 分配到分片上的索引数量  | 集群中每个分片上的索引数量，sum(metric_value) by shard       |       |
| 分配到分片上的文档数    | 集群中每个分片上的文档数量，sum(metric_value) by shard       |       |
| Balancer是否开启        | 集群中是否开启了Balancer，取所有实例中该指标的最大值(top1)   |       |
| 集群事件                | 集群中平均每秒触发的事件次数(metric_value>表示事件被触发)，event维度值表示不同的事件名称 |       |
| Chunk Balancers是否开启 | 是否开启Chunks Balancer, 取所有实例中该指标的最小值(降序top1)，当分片中的chunk数量触发迁移的阈值后，集群会自动在所有分片中进行chunk迁移和均衡 |       |
| Chunk数量               | 集群中Chunk的数量，sum(value) by ty_cluster                  |       |
| 分区表数量              | 集群中分区表的数量                                           |       |
| 分片数据库数量          | 集群中分区的数据库数量，取所有实例中该指标的最大值(top1)     |       |
| 分配到分片上的Chunk数量 | 集群中每个分片上的Chunk数量，sum(metric_value) by shard      |       |
| Draining分片数量        | 集群中处于Draining状态的分片数量，取所有实例中该指标的最大值(top1)，当运行removeShard后，分片会处于Draining状态直到balancer将该分片上的chunk移动到其他分片并将该分片删除 |       |
| 分片数量                | 集群中分片的数量，取所有实例中该指标的最大值(top1)           |       |
| 操作数量                | MongoDB集群操作详情。mongos节点连接数需所有节点累加，configServer、shard都取Primary节点的连接数；该指标type维度值insert、update、delete、query之和为QPS |       |
| 连接数占比              | current/total                                                | %     |

## 副本集指标

| 指标别名         | 指标含义解释                                                 | 单位 |
| ---------------- | ------------------------------------------------------------ | ---- |
| 最近一次选举时间 | 副本及内最后一次执行选举的时间，取副本集中所有实例该指标的最大值(top1) |      |
| 是否发生选举     | 副本集中是否发生了Leader选举, 0-未发生选举；>0则表示发生了选举 |      |
| 复制延迟         | 复制延迟的时间,secondary和primary的延迟                      |      |
| 副本状态         | 副本集中各实例角色状态，1-primary；2-secondary；7-arbiter;   |      |
| Ping 延时        | 副本集成员的ping延迟，取副本集中所有实例该指标的最大值(top1) |      |

## 实例指标

| 指标别名        | 指标含义解释                                                 | 单位    |
| --------------- | ------------------------------------------------------------ | ------- |
| 运行时长        | 该组件的运行时长                                             | s       |
| 角色            | MongoDB角色                                                  |         |
| 版本            | MongoDB版本                                                  |         |
| 可用性          | MongoDB的可用性                                              |         |
| CPU使用率       | 进程的CPU使用率                                              | %       |
| 内存使用量      | 进程的内存使用大小                                           | Bytes   |
| 内存使用率      | 进程内存使用量/主机内存总量                                  | %       |
| 磁盘吞吐(Read)  | 进程每秒读取的磁盘流量                                       | Bytes/s |
| 磁盘吞吐(Write) | 进程每秒写入的磁盘流量                                       | Bytes/s |
| 流量(Send)      | MongoDB每秒发送的字节数                                      | Bytes/s |
| 流量(Receive)   | MongoDB每秒收到的字节数                                      | Bytes/s |
| 吞吐率(QPS)     | 每秒执行的指令的次数                                         | 次/s    |
| 吞吐率详情      | 每秒钟insert、delete、update、return的次数                   | 次/s    |
| 平均响应时间    | 服务的平均响应时间                                           | ms      |
| 响应时间详情    | 具体操作的平均响应时间                                       | ms      |
| 游标总数        | 打开游标的总数量                                             |         |
| pinned游标数    | 打开pinned类型的游标数量                                     |         |
| noTimeout游标数 | 设置了DBQuery.Option.noTimeout参数后，打开的游标数量         |         |
| 连接数          | 当前的连接数                                                 |         |
| 活跃连接数      | 当前的活跃连接数                                             |         |
| 连接数占比      | 当前连接数/最大连接数                                        | %       |
| 排队锁数量      | 等待锁的请求数量                                             |         |
| 读锁数量        | 等待读锁的请求数量                                           |         |
| 写锁数量        | 等待写锁的请求数量                                           |         |
| 存储引擎        | 存储引擎类别                                                 |         |
| 数据容量        | 数据库占用的字节数                                           | Bytes   |
| 内存使用详情    | mapped、resident、virtual内存使用的字节数                    | Bytes   |
| 索引大小        | 数据库索引使用的字节数                                       | Bytes   |
| 主从同步延迟    | 主从同步延迟                                                 | s       |
| 缓存使用率      | WiredTiger缓存使用率，计算方式：当前缓存使用量/缓存配置最大值 | %       |
| 缓存使用量      | WiredTiger缓存中存储的数据量，数据格式为解压缩之后的格式     | Bytes   |
| 缓存最大值      | WiredTiger缓存设置的最大值                                   | Bytes   |
| 事务数          | WiredTiger引擎平均每秒产生的内部事务数量                     |         |
| 被驱逐页数      | WiredTiger引擎平均每秒由于缓存用尽而被驱逐的缓存页数量       |         |

