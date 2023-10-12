---
title: iptables 实际操作之规则查询
description: 
published: true
date: 2023-03-08T10:57:15.159Z
tags: linux
editor: markdown
dateCreated: 2023-03-08T10:57:15.159Z
---





# iptables 实际操作之规则查询

![wsktn.jpg](https://www.zsythink.net/wp-content/uploads/ueditor/php/upload/image/20170413/1492062771658218.jpg)

## 开始测试

<font color="red"> 在进行iptables实验时，请务必在测试机上进行。</font>

之前在iptables的概念中已经提到过，在实际操作iptables的过程中，是以”表”作为操作入口的，如果你经常操作关系型数据库，那么当你听到”表”这个词的时候，你可能会联想到另一个词—-“增删改查”，当我们定义iptables规则时，所做的操作其实类似于”增删改查”，那么，我们就先从最简单的”查”操作入手，开始实际操作iptables。

在之前的文章中，我们已经总结过，iptables为我们预定义了4张表，它们分别是raw表、mangle表、nat表、filter表，不同的表拥有不同的功能。

filter负责过滤功能，比如允许哪些IP地址访问，拒绝哪些IP地址访问，允许访问哪些端口，禁止访问哪些端口，filter表会根据我们定义的规则进行过滤，filter表应该是我们最常用到的表了，所以此处，我们以filter表为例，开始学习怎样实际操作iptables。

怎样查看filter表中的规则呢？使用如下命令即可查看。

```bash
iptables -t filter -L

-t: 指定表
-L: 列出表中规则
```

结果:

```te
[root@cmzhu cmzhu]# iptables -t filter -L
Chain INPUT (policy ACCEPT)       # 链 INPUT 
target     prot opt source               destination
KUBE-FIREWALL  all  --  anywhere             anywhere

Chain FORWARD (policy ACCEPT)      # 链 FORWARD
target     prot opt source               destination
DOCKER-USER  all  --  anywhere             anywhere
DOCKER-ISOLATION-STAGE-1  all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
DOCKER     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
DOCKER     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
DOCKER     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
DOCKER     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere

Chain OUTPUT (policy ACCEPT)          # 链OUTPUT
target     prot opt source               destination
KUBE-FIREWALL  all  --  anywhere             anywhere

Chain DOCKER (4 references)           # 自定义链
target     prot opt source               destination
ACCEPT     tcp  --  anywhere             172.20.0.2           tcp dpt:hbci
ACCEPT     tcp  --  anywhere             172.18.0.2           tcp dpt:http
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:ospf-lite
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:ddi-tcp-2
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:ddi-tcp-1
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:7891
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:7890
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:6170
ACCEPT     tcp  --  anywhere             172.19.0.2           tcp dpt:tproxy
ACCEPT     tcp  --  anywhere             172.19.0.2           tcp dpt:6881
ACCEPT     udp  --  anywhere             172.19.0.2           udp dpt:6881
ACCEPT     tcp  --  anywhere             172.17.0.2           tcp dpt:https
ACCEPT     tcp  --  anywhere             172.17.0.2           tcp dpt:http

Chain DOCKER-ISOLATION-STAGE-1 (1 references)    # 自定义链
target     prot opt source               destination
DOCKER-ISOLATION-STAGE-2  all  --  anywhere             anywhere
DOCKER-ISOLATION-STAGE-2  all  --  anywhere             anywhere
DOCKER-ISOLATION-STAGE-2  all  --  anywhere             anywhere
DOCKER-ISOLATION-STAGE-2  all  --  anywhere             anywhere
RETURN     all  --  anywhere             anywhere

Chain DOCKER-ISOLATION-STAGE-2 (4 references)       # 自定义链
target     prot opt source               destination
DROP       all  --  anywhere             anywhere
DROP       all  --  anywhere             anywhere
DROP       all  --  anywhere             anywhere
DROP       all  --  anywhere             anywhere
RETURN     all  --  anywhere             anywhere

Chain DOCKER-USER (1 references)           # 自定义链
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere

Chain KUBE-FIREWALL (2 references)         # 自定义链
target     prot opt source               destination
DROP       all  -- !127.0.0.0/8          127.0.0.0/8          /* block incoming localnet connections */ ! ctstate RELATED,ESTABLISHED,DNAT
DROP       all  --  anywhere             anywhere             /* kubernetes firewall for dropping marked packets */ mark match 0x8000/0x8000

Chain KUBE-KUBELET-CANARY (0 references)      # 自定义链
target     prot opt source               destination
```

其实, `-t filter` 可以省略, iptables 默认的表就是filter 表.

```bash
# 指定查询INPUT 链的内容.
iptables -L INPUT
```

```txt
[root@cmzhu cmzhu]# iptables -L INPUT
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
KUBE-FIREWALL  all  --  anywhere             anywhere
```

使用 -v 可以查看更加详细的信息

```bash
iptables -vL INPUT
```

```txt
[root@cmzhu cmzhu]# iptables -vL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
4646K  823M KUBE-FIREWALL  all  --  any    any     anywhere             anywhere
```

其实，这些字段就是规则对应的属性，说白了就是规则的各种信息，那么我们来总结一下这些字段的含义。

- **pkts**: 对应规则匹配到的报文的个数。

- **bytes**: 对应匹配到的报文包的大小总和。

- **target**: 规则对应的target，往往表示规则对应的”动作”，即规则匹配成功后需要采取的措施。

- **prot**: 表示规则对应的协议，是否只针对某些协议应用此规则。

- **opt**: 表示规则对应的选项。

- **in**: 表示数据包由哪个接口(网卡)流入，即从哪个网卡来。

- **out**: 表示数据包将由哪个接口(网卡)流出，即到哪个网卡去。

- **source**: 表示规则对应的源头地址，可以是一个IP，也可以是一个网段。

- **destination**: 表示规则对应的目标地址。可以是一个IP，也可以是一个网段。



看着source 和destination 都是anywhere ,iptables默认为我们进行了名称解析, 但是在规则非常多的情况下如果进行名称解析，效率会比较低，所以，在没有此需求的情况下，我们可以使用-n选项，表示不对IP地址进行名称反解，直接显示IP地址，示例如下。

```bash
iptables -nvL INPUT
```

```txt
[root@cmzhu cmzhu]# iptables -nvL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
4648K  824M KUBE-FIREWALL  all  --  *      *       0.0.0.0/0            0.0.0.0/0
```



如上所示, source 和destination 都显示为所有IP或者 IP 段;

如果你需要在显示前面加上列显示

```bash
iptables --line-number -nvL INPUT
```

```txt
[root@cmzhu cmzhu]# iptables --line-number -nvL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    4651K  824M KUBE-FIREWALL  all  --  *      *       0.0.0.0/0            0.0.0.0/0
```

在Chain INPUT 后面括号中数据 **(policy ACCEPT 0 packets, 0 bytes)** ;包含 `policy` `ACCEPT` ; 0 `packages`; 0 `bytes`; 

其中

`policy`: 表示当前链的默认策略

`packages` : 表示当前默认链匹配到的包的数量

`bytes` : 表示 当前默认策略匹配到的所有包的大小总和

```txt
[root@cmzhu cmzhu]# iptables --line-number -nvL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    4655K  825M KUBE-FIREWALL  all  --  *      *       0.0.0.0/0            0.0.0.0/0
[root@cmzhu cmzhu]# iptables --line-number -nvxL INPUT
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num      pkts      bytes target     prot opt in     out     source               destination
1     4655450 824568482 KUBE-FIREWALL  all  --  *      *       0.0.0.0/0            0.0.0.0/0
```


>  举一反三:查看其他表中的内容
>
> iptables -t nat -L
> Iptables -t raw -L
> Iptables -t mangle -L



