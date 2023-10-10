---
title: linux 主机性能监控指标
description: 
published: true
date: 2022-11-29T09:31:41.344Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:31:41.344Z
---

# Linux主机性能指标

## 主机指标

| 指标别名               | 指标含义解释                                                 | 单位      |
| ---------------------- | ------------------------------------------------------------ | --------- |
| 操作系统               | Linux发行版和内核版本                                        |           |
| CPU核数                | CPU的逻辑核数                                                |           |
| CPU架构                | 操作系统内核信息，包含CPU架构                                |           |
| CPU使用率              | 操作系统CPU已经使用的百分比                                  | %         |
| CPU使用详情            | CPU的user、system、softirq、irq、nice、iowait、steal等使用占比 | %         |
| 1分钟平均负载          | CPU 1分钟的平均负载                                          |           |
| 5分钟平均负载          | CPU 5分钟的平均负载                                          |           |
| 15分钟平均负载         | CPU 15分钟的平均负载                                         |           |
| 1分钟平均每核负载      | 1分钟平均负载/CPU核数                                        |           |
| 内存使用率             | 操作系统内存使用占比，公式：内存使用量/内存总量              | %         |
| 内存使用量             | 操作系统已经使用的内存容量                                   | Bytes     |
| 内存Cache              | Cache占用内存大小，Cache指针对文件的缓存                     | Bytes     |
| 内存Buffer             | Buffer占用内存大小，Buffer指针对磁盘块的缓存                 | Bytes     |
| 剩余内存               | 操作系统剩余的内存大小                                       | Bytes     |
| 内存总量               | 主机内存的总容量                                             | Bytes     |
| Major page fault       | 每秒发生缺页中断的次数                                       | 次/s      |
| Swap in                | 每秒页面从Swap转移到内存的数据量。                           | Bytes/s   |
| Swap out               | 每秒页面从内存转移到Swap的数据量。                           | Bytes/s   |
| Swap使用率             | 系统已经使用的Swap大小/Swap总大小                            | Bytes     |
| 剩余Swap               | 系统剩余的Swap大小                                           | Bytes     |
| 磁盘分区使用率         | 磁盘分区的使用占比，公式：已用分区/分区总量                  | %         |
| 已用磁盘分区           | 挂载点已经使用的磁盘大小                                     | Bytes     |
| 剩余磁盘分区           | 挂载点还可以使用的磁盘大小                                   | Bytes     |
| 分区总量               | 挂载点的总大小                                               | Bytes     |
| 分区inode使用率        | 已用inode数/分区inode总数                                    | %         |
| 当前文件打开数         | 系统当前打开的文件数                                         |           |
| 文件打开数Limit        | 系统限制的打开文件数最大值，值为/proc/sys/fs/file-max        |           |
| 磁盘吞吐(Read)         | 磁盘设备每秒读取的流量                                       | Bytes/s   |
| 磁盘吞吐(Write)        | 磁盘设备每秒写入的流量                                       | Bytes/s   |
| 磁盘IOPS(Read)         | 磁盘设备每秒读取的次数                                       | iops      |
| 磁盘IOPS(Write)        | 磁盘设备每秒写入的次数                                       | iops      |
| 磁盘延迟(Read)         | 磁盘设备平均读取延迟                                         | ms        |
| 磁盘延迟(Write)        | 磁盘设备平均写入延迟                                         | ms        |
| IO平均队列             | IO操作的请求队列长度                                         |           |
| IOutil                 | 磁盘设备的繁忙比率                                           | %         |
| 网络吞吐(In)           | 网卡每秒收到的网络流量                                       | bits/s    |
| 网络吞吐(Out)          | 网卡每秒发送的网络流量                                       | bits/s    |
| 网络使用率(In)         | 网络吞吐(In)/网卡带宽                                        | %         |
| 网络使用率(Out)        | 网络吞吐(Out)/网卡带宽                                       | %         |
| 网络包量(In)           | 网卡每秒收到的包数（Packets）                                | Packets/s |
| 网络包量(Out)          | 网卡每秒发送的包数（Packets）                                | Packets/s |
| 丢弃报文数(In)         | 丢弃的报文数，报文已经进入ring buffer，但是因为内存不足丢弃，网卡接收过程每秒drop的次数 | Packets/s |
| 丢弃报文数(Out)        | 丢弃的报文数，报文已经进入ring buffer，但是因为内存不足丢弃，，网卡发送过程每秒drop的次数 | Packets/s |
| 错误报文数(In)         | 错误报文数，校验错误，帧错误，网卡接收过程每秒Error的次数    | Packets/s |
| 错误报文数(Out)        | 错误报文数，校验错误，帧错误，网卡发送过程每秒Error的次数    | Packets/s |
| TCP主动连接速率        | 每秒CLOSE => SYN-SENT次数                                    | 次/s      |
| TCP被动连接速率        | 每秒LISTEN => SYN-RECV次数                                   | 次/s      |
| TCP重传率              | 公式:重传的TCP包/收发的所有TCP包                             | %         |
| TCP当前建连数          | TCP连接处于ESTABLISHED+CLOSE-WAIT状态的数量                  |           |
| Forks                  | 每秒钟fork的进程数                                           |           |
| Runnable状态的进程个数 | 取自/proc/stat. Proc_running                                 |           |
| 等待I/O完成的进程个数  | 取自/proc/stat. Proc_blocked                                 |           |
| 上下文切换             | 平均每秒上下文切换次数                                       | 次/s      |
| 中断                   | 平均每秒发生中断次数                                         | 次/s      |
| 运行时长               | 系统运行的总时长                                             | s         |
| 可用性                 | 系统的可用性，正常/异常                                      |           |

## 进程指标

| 指标别名         | 指标含义解释                                 | 单位    |
| ---------------- | -------------------------------------------- | ------- |
| CPU使用率        | 进程的CPU使用率                              | %       |
| 内存使用量       | 进程的内存使用大小                           | Bytes   |
| 内存使用率       | 进程内存使用量/主机内存总量                  | %       |
| 磁盘吞吐(Read)   | 进程每秒读取的磁盘流量                       | Bytes/s |
| 磁盘吞吐(Write)  | 进程每秒写入的磁盘流量                       | Bytes/s |
| Swap使用         | 进程使用Swap的大小                           | Bytes   |
| 进程数           | 该类进程的进程总数                           |         |
| 当前文件打开数   | 进程当前的文件打开数                         |         |
| 文件数打开数占比 | 该进程的当前文件打开数/进程的文件打开数Limit | %       |
| 运行时长         | 进程的运行时长                               | s       |