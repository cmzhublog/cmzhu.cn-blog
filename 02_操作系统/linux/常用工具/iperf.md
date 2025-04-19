---
title: iperf 测试节点之间网速
description: 
published: true
date: 2022-12-14T09:59:30.698Z
tags: linux
editor: markdown
dateCreated: 2022-12-14T09:59:30.698Z
---

# iperf 测试网速

## 软件下载

```bash
dnf install -y iperf3 
```

## 测速

### server 端
```bash
iperf3 -s
```

### 服务端
```bash
iperf3 -c 10.24.2.1 
```

### 结果
```yaml

Connecting to host 10.24.2.1, port 5201
[  5] local 10.24.105.246 port 40978 connected to 10.24.2.1 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   115 MBytes   961 Mbits/sec    0    519 KBytes
[  5]   1.00-2.00   sec   109 MBytes   913 Mbits/sec    0    573 KBytes
[  5]   2.00-3.00   sec   113 MBytes   944 Mbits/sec    0    634 KBytes
[  5]   3.00-4.00   sec   112 MBytes   942 Mbits/sec    0    634 KBytes
[  5]   4.00-5.00   sec   111 MBytes   931 Mbits/sec    0    634 KBytes
[  5]   5.00-6.00   sec   112 MBytes   943 Mbits/sec    0    667 KBytes
^C[  5]   6.00-6.31   sec  33.8 MBytes   927 Mbits/sec    0    667 KBytes

```

