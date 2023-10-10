---
title: 10_iptables自定义链
description: 
published: true
date: 2023-03-30T07:18:37.743Z
tags: linux
editor: markdown
dateCreated: 2023-03-30T07:18:37.743Z
---

# iptables 自定义链



前面都在学习iptables 默认链的柜子, 这一文来讲解一下自定义链



## 背景

iptables 默认链已经完全满足日常需求了, 为什么还要使用自定义链呢?

原因如下:

- 默认链规则非常多时, 不利于我们管理; 假设一下, 一个链中有`200 `条规则, 其中包含` httpd `服务的,有`sshd `服务的, 有针对私网`IP`的; 我们需要改一条规则, 还需要将所有的链都看一遍, 这样是不是不合理呢?

所以在`iptables` 中, 可以使用自定义链来解决以上的问题;

假设, 我们自定义一个链`IN_WEB` , 可以将所有`web` 相关的规则都放着里面, 当需要修改 web 相关规则的时候, 直接修改这个链就行;即使默认链中有再多的规则, 我们也不用感到害怕了, 因为我们知道, 所有关于`web` 的规则都在`IN_WEB`链中;

下面来手动添加一条自定义链:

```bash
# 添加一条自定义链:
iptables -t filter -N IN_WEB
```



如图所示:

- `-t filter`: 表示 `filter `表
- `-N IN_WEB`: 表示新建一个链叫`IN_WEB`链

创建完成之后, 可以查看filter 表中的链; 多了一个`IN_WEB` 的链; 当这个(0 references ) 的时候, 也就是说这个链没有被任何默认链引用,所以即使我们在`IN_WEB ` 配置链规则, 也不会生效;

```bash
[root@centos ~]# iptables -t filter -nvL  IN_WEB
Chain IN_WEB (0 references)
 pkts bytes target     prot opt in     out     source               destination
```

目前自定义链已经创建完毕了, 现在开始向其中添加规则了; 

```bash
# 向自定义链中添加规则
iptables -t filter -I IN_WEB -s 10.24.2.1 -j REJECT
iptables  -I IN_WEB -s 10.24.2.2 -j REJECT
```

查看`iptables`结果如下:

```bash
[root@centos ~]# iptables -t filter -nvL IN_WEB
Chain IN_WEB (0 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 REJECT     all  --  *      *       10.24.2.2            0.0.0.0/0            reject-with icmp-port-unreachable
    0     0 REJECT     all  --  *      *       10.24.2.1            0.0.0.0/0            reject-with icmp-port-unreachable
```

如图所示, 自定义链中的操作和默认链中的操作并没有什么不同;

现在, 自定义链中已经有了一些规则, 但是, 并不会匹配到一些报文, 因为我们没有在任何链中引用他们,既然`IN_WEB` 是针对入站规则创建的链, 那么 我们就在入站的`INPUT`链中引用他;

下面, 我们看看如何在`INPUT ` 链里面引用它

```bash
# INPUT 链中引用自定义链
iptables -t filter -I INPUT -p tcp --dport 22 -j IN_WEB
```

查看结果如下:

```bash
[root@centos ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 6616 packets, 1083K bytes)
 pkts bytes target     prot opt in     out     source               destination
   82  5664 IN_WEB     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22
 245M   41G CILIUM_INPUT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cilium-feeder: CILIUM_INPUT */
 245M   41G KUBE-PROXY-FIREWALL  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-proxy firewall rules */
 245M   41G KUBE-NODE-PORT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes health check rules */
 245M   41G KUBE-FIREWALL  all  --  *      *       0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:53
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:53
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:67
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:67
```

可以看到INPUT 链,的第一个规则就是指向了`IN_WEB`,可以查看 `IN_WEB` 链,里面 `(1 references)` 变成了`1`;

```bash
[root@centos ~]# iptables -t filter -nvL IN_WEB
Chain IN_WEB (1 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 REJECT     all  --  *      *       10.24.2.2            0.0.0.0/0            reject-with icmp-port-unreachable
    0     0 REJECT     all  --  *      *       10.24.2.1            0.0.0.0/0            reject-with icmp-port-unreachable
```



下面来验证规则是否有生效; 

```bash
[root@bqdev02 ~]# ssh 10.24.8.164
ssh: connect to host 10.24.8.164 port 22: Connection refused
[root@bqdev02 ~]# hostname -i
10.24.2.2
[root@bqdev02 ~]#
```



```bash
[root@bqdev01 ~]# ssh 10.24.8.164
ssh: connect to host 10.24.8.164 port 22: Connection refused
[root@bqdev01 ~]# hostname -i
10.24.2.1
[root@bqdev01 ~]#
```

使用其他的节点, 就正常访问; 

```bash
root at bqdev03 in ~
# ssh 10.24.8.164
root@10.24.8.164's password:
Last login: Thu Mar 30 14:40:12 2023 from 10.24.100.230
[root@centos ~]#

```



假如我觉得, 这个链的名字叫`IN_WEB` 不合适, 我们需要改成 `WEB`, 如何做呢?

```bash
# 修改自定义链的名字
iptables -t filter -E IN_WEB WEB
```

```bash
[root@centos ~]# iptables -t filter -E IN_WEB WEB
[root@centos ~]# iptables -t filter -nvL IN_WEB
iptables: No chain/target/match by that name.
[root@centos ~]#
[root@centos ~]# iptables -t filter -nvL WEB
Chain WEB (1 references)
 pkts bytes target     prot opt in     out     source               destination
    1    60 REJECT     all  --  *      *       10.24.2.2            0.0.0.0/0            reject-with icmp-port-unreachable
    2   120 REJECT     all  --  *      *       10.24.2.1            0.0.0.0/0            reject-with icmp-port-unreachable
[root@centos ~]#
```

如上所示, 已经全部完成修改; 使用 `- E` 可以修改自定义链的名字



那么, 应该如何删除自定义链呢? 

```bash
iptables -t filter -X WEB

```

```bash
[root@centos ~]# iptables -t filter -X WEB
iptables: Too many links.
```

发现删除不了, 原因是该链有被使用, 需要先将`INPUT` 链中的规则给删除掉

```bash
iptables -t filter -D INPUT 1
```

然后再来执行删除链, 发现

```bash
[root@centos ~]# iptables -t filter -X WEB
iptables: Directory not empty.
```



下面来尝试一下将`WEB` 链中的所有规则都清理掉, 如下所示, 自定义链可以正常删除了

```bash
[root@centos ~]# iptables -t filter -F WEB
[root@centos ~]#
[root@centos ~]# iptables -t filter -X WEB
[root@centos ~]#
```

因此,总结如下: 删除自定义链的方法是

- 清理掉所有其他链的引用 ,变成如下状态`(0 references)`  
- 清理掉该链中的所有规则; 
- 使用`- X ` 删除该链.

