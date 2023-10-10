---
title: Kubernetes监控指标
description: 
published: true
date: 2022-11-29T09:36:00.937Z
tags: monitor
editor: markdown
dateCreated: 2022-11-29T09:36:00.937Z
---

# Kubernetes监控指标

针对整个Kubernetes集群的资源利用率，展示以下指标：

| 指标名称        | 指标含义解释                                                 | 单位  |
| --------------- | ------------------------------------------------------------ | ----- |
| CPU usage       | k8s集群内所有Container已经使用的CPU百分比，公式为container使用的CPU/Max CPUs。 | %     |
| Max CPUs        | k8s集群内所有Node节点允许使用的CPU总量。                     | Cores |
| Memory Usage    | k8s集群内所有container内存使用占比。公式为k8s集群内所有Container已经使用的内存总量/Max Memory。 | %     |
| Max Memory      | k8s集群内所有Node节点允许使用的内存总量。                    | Bytes |
| CPU requests    | k8s集群内所有Container配置的Requests CPU总和占比。公式为container request cpu/Max CPUs。 | %     |
| Memory Requests | k8s集群内所有Container配置的Requests 内存总和占比。公式为container request memory/Max Memory。 | %     |
| CPU limits      | k8s集群内集群内所有Container配置的limits CPU总和占比。公式为container limit cpu/Max CPUs。 | %     |
| Memory limits   | k8s集群内集群内所有Container配置的limits 内存总和占比。公式为container limit memory/Max CPUs。 | %     |
| CPU avalible    | 表示可允许配置的CPU资源占比，公式为1- CPU Requests。         | %     |
| Memory avalible | 表示可允许配置的内存资源占比，公式为1- Memory Requests。     | %     |

针对每个Node，展示以下指标：

| 指标名称        | 指标含义解释                                              | 单位            |
| --------------- | --------------------------------------------------------- | --------------- |
| Status          | 表示Node的状态。                                          | Ready/Not Ready |
| Problem         | 是否有CPU利用率过高，内存利用率过高，磁盘空间占满等异常。 |                 |
| CPU usage       | Node节点内所有Container的CPU使用量。                      | mCores          |
| CPU requests    | Node节点内所有Container 的CPU requests量。                | mCores          |
| CPU limits      | Node节点内所有Container的CPU limits量。                   | mCores          |
| Memory usage    | Node节点内所有Container的内存使用量。                     | Bytes           |
| Memory requests | Node节点内所有Container的内存 Requests量。                | Bytes           |
| Memory limits   | Node节点内所有Container的内存Limits量。                   | Bytes           |

针对每个Pod，展示以下指标：

| 指标名称               | 指标含义解释                     | 单位    |
| ---------------------- | -------------------------------- | ------- |
| 运行状态               | Pod的运行状态。                  |         |
| Namespace              | Pod所属的Namespace。             |         |
| CPU（mCores）          | Pod内所有Container的CPU使用量。  | mCores  |
| Memory Usage           | Pod内所有Container的内存使用量。 | Bytes   |
| 网络吞吐（In/Out）     | Pod内所有Container的网络吞吐量。 | bit/s   |
| 磁盘吞吐（Read/Write） | Pod内所有Container的磁盘吞吐量。 | Bytes/s |
| 重启次数               |                                  | 次      |
| Tag Name               | pod上标签的key值。               |         |
| Tag Value              | pod上标签的value值。             |         |
| Event Type             | Event类型。                      |         |
| Event Reason           | Event原因。                      |         |
| Event 时间             | Event时间。                      |         |
| Event Message          | Event内容。                      |         |

