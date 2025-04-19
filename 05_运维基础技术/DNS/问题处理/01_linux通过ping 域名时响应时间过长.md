## linux 服务器ping 域名时响应时长过长

### 问题描述

1、 linux ping 域名时需要10s 左右, 下面是测试ping 一次 gitlab.cmzhu.cn 的时长， 期间可以发现是10s 中才结束一次ping

```bash
$ time ping gitlab.cmzhu.cn -c 1
PING gitlab.netfang.cn (192.168.101.88) 56(84) bytes of data.
64 bytes from 192.168.101.88: icmp_seq=1 ttl=63 time=1.04 ms

--- gitlab.netfang.cn ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.042/1.042/1.042/0.000 ms

real    0m9.521s
user    0m0.002s
sys     0m0.001s

```

### 原因分析

1、 linux 在ping 的过程中会同时去找反向查找区域， 期间会使用ip 查询域名操作

2、windows 上没有此问题，windows 不会进行区域查询操作

### 解决思路

#### 方案1

指定 `-n` 参数， 用于让ping 命令不进行反向查询

```bash
$ ping -n gitlab.cmzhu.cn
```

#### 方案2

在域控配置反向查找区域，用于指定192 网段进行反向查找， 然后dns 可以自行去建立ptr 记录