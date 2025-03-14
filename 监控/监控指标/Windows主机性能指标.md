---
title: Windows主机性能指标
description: 
published: true
date: 2022-11-29T09:33:28.065Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:33:28.065Z
---

# Windows主机性能指标

## 主机指标

| 指标别名         | 指标含义解释                                                 | 单位      |
| ---------------- | ------------------------------------------------------------ | --------- |
| 运行时长         | 操作系统运行的秒数                                           | s         |
| 系统信息         | 操作系统名称和版本信息                                       |           |
| CPU核数          | CPU的逻辑核数                                                |           |
| CPU使用率        | 操作系统CPU已经使用的百分比                                  | %         |
| CPU使用详情      | CPU的user、interrupt、dpc、privileged等使用占比              | %         |
| 内存使用率       | 操作系统内存使用占比，公式：内存使用量/内存总量              | %         |
| 内存使用量       | 操作系统已经使用的内存容量                                   | Bytes     |
| 内存Cache        | Cache占用内存大小                                            | Bytes     |
| 剩余内存         | 操作系统剩余的内存大小                                       | Bytes     |
| 内存总量         | 主机内存的总容量                                             | Bytes     |
| Major Page Fault | 每秒发生缺页中断的次数                                       | 次/s      |
| Swap In          | 每秒页面从Swap转移到内存的数据量。                           | Bytes/s   |
| Swap Out         | 每秒页面从内存转移到Swap的数据量。                           | Bytes/s   |
| 磁盘分区使用率   | 磁盘分区的使用占比，公式:已用分区/分区总量                   | %         |
| 已用磁盘分区     | 挂载点已经使用的磁盘大小                                     | Bytes     |
| 剩余磁盘分区     | 挂载点还可以使用的磁盘大小                                   | Bytes     |
| 分区总量         | 挂载点的总大小                                               | Bytes     |
| 磁盘吞吐(Read)   | 磁盘设备每秒读取的流量                                       | Bytes/s   |
| 磁盘吞吐(Write)  | 磁盘设备每秒写入的流量                                       | Bytes/s   |
| 磁盘IOPS(Read)   | 磁盘设备每秒读取的次数                                       | iops      |
| 磁盘IOPS(Write)  | 磁盘设备每秒写入的次数                                       | iops      |
| 磁盘延迟(Read)   | 磁盘设备平均读取延迟                                         | ms        |
| 磁盘延迟(Write)  | 磁盘设备平均写入延迟                                         | ms        |
| 网络吞吐(In)     | 网卡每秒收到的网络流量                                       | bits/s    |
| 网络吞吐(Out)    | 网卡每发送的网络流量                                         | bit/s     |
| 网络包量(In)     | 网卡每秒收到的包数（Packets）                                | Packets/s |
| 网络包量(Out)    | 网卡每秒发送的包数（Packets）                                | Packets/s |
| 丢弃报文数(In)   | 丢弃的报文数，报文已经进入ring buffer，但是因为内存不足丢弃，网卡接收过程每秒drop的次数 | Packets/s |
| 丢弃报文数(Out)  | 丢弃的报文数，报文已经进入ring buffer，但是因为内存不足丢弃，，网卡发送过程每秒drop的次数drop的次数 | Packets/s |
| 错误报文数(In)   | 错误报文数，校验错误，帧错误，网卡接收过程每秒Error的次数    | Packets/s |
| 错误报文数(Out)  | 错误报文数，校验错误，帧错误，网卡接收过程每秒Error的次数    | Packets/s |
| TCP建连失败速率  | 每秒(SYN-SENt=>CLOSE + SYN-RECV=>CLOSE + SYN-RECV=>LISTEN)次数 | 次/s      |
| TCP主动连接速率  | 每秒CLOSE => SYN-SENT次数                                    | 次/s      |
| TCP被动连接速率  | 每秒LISTEN => SYN-RECV次数                                   | 次/s      |
| TCP连接重置速率  | 每秒连接重置的次数                                           | 次/s      |
| TCP重传率        | 公式:重传的TCP包/收发的所有TCP包                             | %         |
| TCP当前建连数    | TCP连接处于ESTABLISHED+CLOSE-WAIT状态的数量                  |           |
| 运行时长         | 系统运行的总时长                                             | s         |
| 可用性           | 系统的可用性，正常/异常                                      |           |

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
| 文件打开数Limit  | 进程限制的最大文件打开数                     |         |
| 文件数打开数占比 | 该进程的当前文件打开数/进程的文件打开数Limit | %       |
| 运行时长         | 进程的运行时长                               | s       |

