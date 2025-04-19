---
title: Docker监控指标
description: 
published: true
date: 2022-11-29T09:34:49.416Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:34:49.416Z
---

# Docker监控指标

## Docker指标

| 指标别名              | 指标含义解释                    | 单位 |
| --------------------- | ------------------------------- | ---- |
| Docker版本            | 主机上运行的Docker版本          |      |
| 运行容器数量          | 主机上运行的容器数量            |      |
| Image数量             | 主机上Image的数量               |      |
| Docker Storage Driver | Docker存储引擎                  |      |
| Docker Registry       | Docker默认镜像地址              |      |
| Docker运行状态        | Docker服务的运行状态，正常/异常 |      |

## 容器指标

| 指标别名        | 指标含义解释                       | 单位    |
| --------------- | ---------------------------------- | ------- |
| 运行时长        | 容器的运行时长                     | s       |
| CPU使用率       | 进程的CPU使用率                    | %       |
| 内存使用量      | 容器的内存使用大小                 | Bytes   |
| 磁盘吞吐(Read)  | 容器每秒读取的磁盘流量             | Bytes/s |
| 磁盘吞吐(Write) | 容器每秒写入的磁盘流量             | Bytes/s |
| 网络吞吐(In)    | 容器每秒收到的网络流量             | bits/s  |
| 网络吞吐(Out)   | 容器每秒发送的网络流量             | bits/s  |
| 镜像名称        | 容器的镜像名称                     |         |
| 端口            | 容器的端口映射关系                 |         |
| Command         | 容器的Command执行命令              |         |
| Pid             | 容器内进程在宿主机上的Pid          |         |
| Pod名称         | 容器所属的Pod名称，k8s环境下才会有 |         |

