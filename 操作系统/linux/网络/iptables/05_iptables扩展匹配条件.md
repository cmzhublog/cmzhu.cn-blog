

## iptables 扩展匹配条件

前文有讲到了扩展匹配条件, 使用到了tcp 模块; 除了这个模块之外, 还有很多扩展模块, 这里对一些常用的扩展模块进行一一讲解.

### iprange 模块

之前有讲过, 在不使用任何扩展模块情况下, 可以使用 -s he -d 指定匹配的报文的源地址和目标地址, 在指定IP 时可以指定一个IP, 也可以使用 `","` 指定多个IP, 但是<font color="red">不能连续指定多个IP</font>, 使用iprange 就可以实现;

iprange 扩展模块中提供了两个扩展匹配条件

--src-range

--dst-range

指定源地址的范围和目标地址的范围

例如: 指定`192.168.122.100-192.168.122.120` 范围中的所有IP 访问 `192.168.122.215` 的包全部丢弃;

```bash
iptables -t filter -I INPUT -m iprange --src-range 192.168.122.100-192.168.122.120 -j DROP
```

```bash
[root@cmzhu1 ~]# iptables -t filter -nvL INPUT
Chain INPUT (policy ACCEPT 111 packets, 7460 bytes)
 pkts bytes target     prot opt in     out     source               destination
   17  1428 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            source IP range 192.168.122.100-192.168.122.120
```



测试可以发现 192.168.122.100-192.168.122.120 的所有包都被丢弃掉了;

查看规则时 可以发现是`"source IP range 192.168.122.100-192.168.122.120" ` 在最后的描述信息中体现对应的IP限制范围

### string 模块

使用string 模块, 可以指定要匹配的字符串, 如果报文中存在符合条件的字符串, 则符合匹配条件

比如, 如果报文中存在 `"OOXX"`, 我们就丢弃该报文;

首先，我们在`IP`为 `108` 的主机上启动`http`服务，然后在默认的页面目录中添加两个页面，页面中的内容分别为`”OOXX”`和`”Hello World”`，如下图所示，在没有配置任何规则时，126主机可以正常访问146主机上的这两个页面。

```bash
iptables -A INPUT -p tcp --dport 8080 -m string --algo bm --string "OOXX" -j DROP

[root@centos ~]# iptables -t filter -nvL  INPUT
Chain INPUT (policy ACCEPT 391 packets, 45789 bytes)
 pkts bytes target     prot opt in     out     source               destination
  53M 9088M CILIUM_INPUT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cilium-feeder: CILIUM_INPUT */
  53M 9091M KUBE-PROXY-FIREWALL  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kube-proxy firewall rules */
  53M 9091M KUBE-NODE-PORT  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes health check rules */
  53M 9093M KUBE-FIREWALL  all  --  *      *       0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:53
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:53
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:67
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:67
    0     0 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 STRING match  "OOXX" ALGO name bm TO 65535

```

### time扩展模块

可以通过 time 模块, 根据时间段匹配报文, 如果报文在时间范围之类, 就符合匹配条件, 例如:
我需要自我约束, 早上9 点到晚上 6 点不能看网页? 







