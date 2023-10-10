---
title: 4_iptables匹配条件
description: 
published: true
date: 2023-03-13T06:15:46.546Z
tags: linux
editor: markdown
dateCreated: 2023-03-13T06:15:46.546Z
---

# iptables 匹配条件

## 源地址匹配

1、在之前学习中, iptables 可以使用 -s 匹配指定IP; 

```bash
iptables -t filter -I INPUT -s 192.168.122.108 -j ACCEPT

```

2、 那么,如果需要匹配多个IP, 应该如何处理呢?

```bash
iptables -t filter -I INPUT -s 192.168.122.108,192.168.122.109 -j ACCEPT
```

可以看出上面指令, 再匹配多条IP的时候, 需要用`","` 隔开; 

结果如下:

```bash
[root@cmzhu1 ~]# iptables -t filter --line  -nvL INPUT
Chain INPUT (policy ACCEPT 50 packets, 3652 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 ACCEPT     all  --  *      *       192.168.122.109      0.0.0.0/0
2        0     0 ACCEPT     all  --  *      *       192.168.122.108      0.0.0.0/0
```

3、 除了对静态IP 匹配之外, 还可以对指定的网段进行匹配(<font color="red">执行指令时需要注意</font>)

```bash
iptables -t filter -I INPUT -s 192.168.0.0/16 -j DROP
```

4、 还可以对规则进行取反, 指令如下

```bash
iptables -t filter -I INPUT ! -s 192.168.122.108 -j DROP 
```

结果如下: 只有 192.168.122.108 能够访问;其余源地址IP 均不能访问;

```bash
[root@cmzhu1 ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
   19  1696 DROP       all  --  *      *      !192.168.122.108      0.0.0.0/0
```



## 目标地址匹配

除了通过 源地址 进行匹配之外, 还可以通过目标地址方式进行匹配(-d 目标地址); 为了完成实验, 我先在本地网卡上添加一个新的同段IP;

```bash
ip a a 192.168.122.100/24 dev eth0 
```

添加后IP 结果如下:

```bash
[root@cmzhu1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:3d:14:a2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.215/24 brd 192.168.122.255 scope global noprefixroute dynamic eth0
       valid_lft 2835sec preferred_lft 2835sec
    inet 192.168.122.100/24 scope global secondary eth0
       valid_lft forever preferred_lft forever

```

1、 现在需要实现, 访问 192.168.122.215 的请求全部拒绝访问, 请求 192.168.122.100 的请求可以通过, 设计的iptables 规则如下;

```bash
iptables -t filter -I INPUT -d 192.168.122.215 -j REJECT
iptables -t filter -I INPUT -d 192.168.122.100 -j ACCEPT
```



添加完成之后, 会发现` ssh 192.168.122.215` 登录失败, 而 `ssh 192.168.122.100 ` 登录比较缓慢; 

```bash
[root@cmzhu1 ~]# ssh 192.168.122.215
ssh: connect to host 192.168.122.215 port 22: Connection refused

[root@cmzhu1 ~]# ip r
default via 192.168.122.1 dev eth0 proto dhcp metric 100
192.168.122.0/24 dev eth0 proto kernel scope link src 192.168.122.215 metric 100
```

发现没有 指定`192.168.122.100 `的路由规则, 需要手动添加

```bash
ip r a 192.168.122.0/24 dev eth0 proto kernel scope link src 192.168.122.100
```

添加了这行路由之后,登录速度变快, 最后仔细检查发现,这个路由会自己重建; 

## 协议类型匹配

除了匹配源地址, 目的地址外, 还可以对使用`-p`对数据包的协议进行匹配	;

```bash
# 实验前先清空
iptables -t filter -F INPUT
iptables -t filter -I INPUT -s 192.168.122.108 -d 192.168.122.215 -p tcp -j REJECT
```

这时使用ssh 来连接, 会发现连接 `192.168.122.215 `会被拒绝; 

```bash
[root@cmzhu2 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:04:4e:71 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.108/24 brd 192.168.122.255 scope global noprefixroute dynamic eth0
       valid_lft 3532sec preferred_lft 3532sec
[root@cmzhu2 ~]# ssh 192.168.122.215
ssh: connect to host 192.168.122.215 port 22: Connection refused
```

使用` ssh 192.168.101.100` 是正常登录的

```bash
[root@cmzhu2 ~]# ssh 192.168.122.100
The authenticity of host '192.168.122.100 (192.168.122.100)' can't be established.
ECDSA key fingerprint is SHA256:ewWsA9SJMifMOMdp+nvRuyBuJH0pZtdcgmD1VZ9ueRY.
ECDSA key fingerprint is MD5:0f:09:73:1c:62:c5:e9:a7:c8:bb:65:c5:e0:76:84:98.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.122.100' (ECDSA) to the list of known hosts.
Last login: Sat Mar 11 14:32:29 2023 from gateway
```

在这个指令中 -p 参数主要支持以下协议

Centos6:

1. tcp
2. udp
3. udplite
4. icmp
5. esp
6. ah
7. sctp

centos7 :

1. tcp
2. udp
3. udplite
4. icmp
5. icmpv6
6. esp
7.  ah
8.  sctp
9.  mh

在不需要使用 -p 参数时默认会被匹配到所有的协议

## 网卡匹配

我们再来认识一个新的匹配条件，当本机有多个网卡时，我们可以使用 -i 选项去匹配报文是通过哪块网卡流入本机的

我们先动手做个小例子，对-i选项有一个初步的了解以后，再结合理论去看

当前主机的网卡名称为eth4，如下图

![img](https://www.zsythink.net/wp-content/uploads/2017/04/041917_0537_12.png)

假设想要拒绝由网卡eth4流入的ping请求报文，则可以进行如下设置

![img](https://www.zsythink.net/wp-content/uploads/2017/04/041917_0537_13.png)

从图中可以看到,使用 `-i eth4 ` 可以指定 对网卡进行匹配; 上例子中表示丢弃icmp 报文

1、 实际操作如下

首先,添加一个网卡 eth1, IP 配置如下

```bash
[root@cmzhu1 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:3d:14:a2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.215/24 brd 192.168.122.255 scope global noprefixroute dynamic eth0
       valid_lft 3528sec preferred_lft 3528sec
    inet 192.168.122.100/24 scope global secondary eth0
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:bf:0d:36 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.121/24 brd 192.168.122.255 scope global noprefixroute dynamic eth1
       valid_lft 3564sec preferred_lft 3564sec
```

按照上面指令,对网卡访问进行限制

```bash
iptables -t filter -I INPUT -i eth1 -p icmp -j DROP
```

```bash
[root@cmzhu1 ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 204 packets, 14788 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       icmp --  eth1   *       0.0.0.0/0            0.0.0.0/0
```

验证失败: 猜测原因应该是linux 内核 arp 转发的问题; 调整验证协议为TCP;

```bash
iptables -t filter -I INPUT -i eth1 -p tcp -j DROP
```

```bash
[root@cmzhu1 ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 57 packets, 7507 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       tcp  --  eth1   *       0.0.0.0/0            0.0.0.0/0
```

发现`ssh 192.168.122.121` 走eth1 的ssh 请求,失败了, 同时使用 eth0 网卡的IP连接,能够正常登录; 验证结果如下

```bash
[root@cmzhu2 ~]# ssh 192.168.122.121
ssh: connect to host 192.168.122.121 port 22: No route to host
[root@cmzhu2 ~]# ^C
[root@cmzhu2 ~]# ssh 192.168.122.100
Last login: Sat Mar 11 14:36:16 2023 from centos7-zero
[root@cmzhu1 ~]# exit
logout
Connection to 192.168.122.100 closed.
```



## 扩展匹配条件

除了 在使用基本匹配条件之外, 还有根据`源端口`和`目标端口`进行匹配 , 这个是和`源地址`和`目标地址` 不同, `源端口`和`目标端口` 属于扩展匹配条件;

那么, 什么是扩展匹配呢? 

基本匹配条件是直接可以使用的匹配条件, 而扩展匹配条件是需要依赖于一些扩展模块,才能用来使用的条件

例如: 
目前sshd 的默认访问端口为22, 当访问远程时, 服务器会默认链接22 端口, 假设, 我们使用一条 `iptables` 规则来拒绝来自`192.168.122.108` 的访问本机`192.168.122.215`  的请求,实现如下:

```bash
iptables -t filter -I INPUT -s 192.168.122.108 -p tcp -m tcp --dport 22 -j REJECT
```

```bash
[root@cmzhu1 ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 63 packets, 4392 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 REJECT     tcp  --  *      *       192.168.122.108      0.0.0.0/0            tcp dpt:22 reject-with icmp-port-unreachable
```

通过这一规则, 实现从`192.168.122.108` 访问`192.168.122.215` 的`22`端口所有请求,全部都拒绝;其他节点访问不受影响.

下面来解析一下这个命令:

- -t filter -I INPUT: 指定将规则插入到 filter 表的INPUT 链上
- -s 192.168.122.108: 指定访问的源地址
- -p tcp: 用于指定匹配报文的协议
- -m tcp: 用于指定扩展模块的名称

在上述例子中, 可以忽略`-m tcp` ; 因为iptables 在默认情况下, 会找和 -p 协议参数相同的模块; 

```bash
iptables -t filter -I INPUT -s 192.168.122.108 -p tcp --dport 22 -j ACCEPT
```

```bash
[root@cmzhu1 ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 76 packets, 5274 bytes)
 pkts bytes target     prot opt in     out     source               destination
   18  4609 ACCEPT     tcp  --  *      *       192.168.122.108      0.0.0.0/0            tcp dpt:22
    1    60 REJECT     tcp  --  *      *       192.168.122.108      0.0.0.0/0            tcp dpt:22 reject-with icmp-port-unreachable
```

由上,可知; 访问的`192.168.122.215` 的`22 `端口变得正常; 

另外,如果需要指定多个连续的端口, 可以直接使用`":"` 来进行写;如下命令,可以发现所有22:30 的端口的包都被丢弃

```bash
iptables -t filter -I INPUT -s 192.168.122.108 -p tcp  -m tcp --dport 22:30 -j DROP
```

 

如果需要指定多个端口匹配, 可以使用`","` 进行分割,例如,我要限制22, 80 等端口访问, 需要使用到 `multiport` 这个模块; 

```bash
iptables -t filter -I INPUT -s 192.168.122.108 -p tcp  -m multiport --dport 22,80 -j DROP
```

  









