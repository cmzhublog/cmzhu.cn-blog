## iptables 规则管理

之前打过一个比方，每条”链”都是一个”关卡”，每个通过这个”关卡”的报文都要匹配这个关卡上的规则，如果匹配，则对报文进行对应的处理，比如说，你我二人此刻就好像两个”报文”，你我二人此刻都要入关，可是城主有命，只有器宇轩昂之人才能入关，不符合此条件的人不能入关，于是守关将士按照城主制定的”规则”，开始打量你我二人，最终，你顺利入关了，而我已被拒之门外，因为你符合”器宇轩昂”的标准，所以把你”放行”了，而我不符合标准，所以没有被放行，其实，”器宇轩昂”就是一种”匹配条件”，”放行”就是一种”动作”，”匹配条件”与”动作”组成了规则。

iptables 匹配方式分为 “ 源地址 ”, “ 目的地址 ”, “ 源端口 ”, “ 目的端口 ”等方式, 匹配完成后对常用的动作有 ACCEPT (接受), DROP(丢弃), REJECT(拒绝); ACCEPT 类似于放行; 这里并不是全部的匹配条件,只是日常维护中常用的条件.



实验: 研究iptables 对规则匹配和规则动作的验证;

实验基于 INPUT 链在filter 表中的实现 规则;

1、 首先查看INPUT 链

```bash
iptables -t filter -nvL INPUT
```

```bash
[root@cmzhu1 ~]# iptables -nL INPUT
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
INPUT_direct  all  --  0.0.0.0/0            0.0.0.0/0
INPUT_ZONES_SOURCE  all  --  0.0.0.0/0            0.0.0.0/0
INPUT_ZONES  all  --  0.0.0.0/0            0.0.0.0/0
DROP       all  --  0.0.0.0/0            0.0.0.0/0            ctstate INVALID
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
```

2、 清空iptables 规则, 好开始本次实验

```bash
iptables -F -t filter INPUT
```



3、 准备一台新的机器(192.168.122.108), 用来ping 前一台 测试机器, 可以发现目前是可通的;

```bash
[root@cmzhu2 ~]# ping 192.168.122.215
PING 192.168.122.215 (192.168.122.215) 56(84) bytes of data.
64 bytes from 192.168.122.215: icmp_seq=1 ttl=64 time=0.169 ms
64 bytes from 192.168.122.215: icmp_seq=2 ttl=64 time=0.171 ms
64 bytes from 192.168.122.215: icmp_seq=3 ttl=64 time=0.173 ms
64 bytes from 192.168.122.215: icmp_seq=4 ttl=64 time=0.180 ms
--- 192.168.122.215 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3000ms
rtt min/avg/max/mdev = 0.169/0.173/0.180/0.010 ms
[root@cmzhu2 ~]#
```

4、 加入一条置顶 DROP  ping 机器的规则

```bash
iptables -t filter -I INPUT -s 192.168.122.108 -j DROP
```

查看一下INPUT 表

```bash
[root@cmzhu1 ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 139 packets, 10016 bytes)
 pkts bytes target     prot opt in     out     source               destination
   50  4200 DROP       all  --  *      *       192.168.122.108      0.0.0.0/0
```

5、 在新机器(192.168.122.108) 对旧机器进行ping 测试;  可以发现 ping 包已经全部被丢弃了;

```bash
[root@cmzhu2 ~]# ping 192.168.122.215
PING 192.168.122.215 (192.168.122.215) 56(84) bytes of data.
```

实验到这里,基本完成了, 在INPUT链里面 添加DROP 规则, 是可以对所有包进行丢弃的;

6、 但是再深入一下, 我传入多条规则是怎么匹配的呢; 从上述文档中说是顺序匹配的, 这个下面就开始验证一下.

在上面的基础上, 再插入一条相同的ACCEPT规则

```
iptables -t filter -A INPUT  -s 192.168.122.108 -j ACCEPT
```

得出的防火墙结果为

```bash
[root@cmzhu1 ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 7 packets, 504 bytes)
 pkts bytes target     prot opt in     out     source               destination
  376 31584 DROP       all  --  *      *       192.168.122.108      0.0.0.0/0
    0     0 ACCEPT     all  --  *      *       192.168.122.108      0.0.0.0/0
```

从上图中很难看出顺序, 上文学过; 可以使用 `--line` 获得非常详细的信息.

```bash
[root@cmzhu1 ~]# iptables -t filter --line -nvL INPUT
Chain INPUT (policy ACCEPT 47 packets, 3508 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      534 44856 DROP       all  --  *      *       192.168.122.108      0.0.0.0/0
2        0     0 ACCEPT     all  --  *      *       192.168.122.108      0.0.0.0/0
```

这样添加链的结果如此, 并且在 规则1 的pkts 在不断增加, 有理由怀疑,数据包在匹配到第一条规则时被处理了,就没有进行下一步去匹配.

为此可以做以下验证, 

7、在规则的头部添加一个ACCEPT 的规则, 理论上结果应该是 能够ping 通,验证一下

添加规则

```bash
iptables -t filter -I INPUT -s 192.168.122.108 -j ACCEPT
```

添加后规则如下, 为此可以发现, 规则2 的pkts 没有变化, 并且ping 功能恢复正常

```bash
[root@cmzhu1 ~]# iptables -t filter --line -nvL INPUT
Chain INPUT (policy ACCEPT 7 packets, 504 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        3   252 ACCEPT     all  --  *      *       192.168.122.108      0.0.0.0/0
2      750 63000 DROP       all  --  *      *       192.168.122.108      0.0.0.0/0
3        0     0 ACCEPT     all  --  *      *       192.168.122.108      0.0.0.0/0
```



```bash
[root@cmzhu1 ~]# iptables -t filter --line -nvL INPUT
Chain INPUT (policy ACCEPT 13 packets, 904 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      379 31836 ACCEPT     all  --  *      *       192.168.122.108      0.0.0.0/0
2      750 63000 DROP       all  --  *      *       192.168.122.108      0.0.0.0/0
3        0     0 ACCEPT     all  --  *      *       192.168.122.108      0.0.0.0/0
```

```bash
[root@cmzhu2 ~]# ping 192.168.122.215
PING 192.168.122.215 (192.168.122.215) 56(84) bytes of data.
64 bytes from 192.168.122.215: icmp_seq=1 ttl=64 time=0.117 ms
64 bytes from 192.168.122.215: icmp_seq=2 ttl=64 time=0.177 ms
64 bytes from 192.168.122.215: icmp_seq=3 ttl=64 time=0.157 ms
--- 192.168.122.215 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.117/0.150/0.177/0.026 ms
```

由此可以推断, 规则链式匹配是正确的;

8、 在学习之后,应该如何删除规则呢? 下面指令表示删除第一条规则

```bash
iptables -t filter -D INPUT 1
```



9、修改规则危险, 勿用

```bash
iptables -t filter -R INPUT 1 -s 192.168.122.108 -j REJECT
```





 