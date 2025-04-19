## iptables 扩展匹配条件之 -tcp-flags

前文之中已经对tcp 扩展模块进行过总结, 但是 只介绍了`--dport` 和 `--sport` 的匹配, 并没有对 `-tcp-flags`进行总结, 下面来学习一下

> 在学习此章需要对tcp 协议有一定了解, 如tcp 报头, TCP 三次握手

`-tcp-flags` : 就是`tcp` 头中的标志位

看来,在使用iptables 匹配的过程中, 我们可以使用扩展匹配条件匹配`tcp` 报文头部的标识位, 然后根据标识位实际情况实现控制功能

回顾一下tcp 报头的结构, 如下图所示:

![img](https://pic4.zhimg.com/80/v2-01f79cd44556d2c72edd474debc0bb53_1440w.webp)

在使用`iptables`时，使用`tcp`扩展模块的`–tcp-flags`选项，即可对上图中的标志位进行匹配，判断指定的标志位的值是否为`1`;

在tcp建立协议的时候, 需要进行三次握手, 而三次握手刚好就依靠 `tcp` 头部的标志位进行

回顾一下tcp 建立连接的步骤:
![img](https://pic2.zhimg.com/80/v2-148dec1af2412bc61873b383649bf609_1440w.webp)



第一次握手: 客户端向服务端 发送报文头 `SYN=1` 的报文

第二次握手: 服务端接受到之后, 返回 `SYN=1,ACK=1` 的报文

第三次握手: 客户端再向服务端发送 `ACK=1` 的报文

根据这些步骤: 可以通过 iptables 匹配tcp 传输的相应握手步骤

```bash
# 第一次握手
iptables -t filter -I INPUT -p tcp -m tcp --dport 22 --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN -j REJECT

# 第二次握手
iptables -t filter -I OUTPUT -p tcp -m tcp --dport 22 --tcp-flags SYN,ACK,FIN,RST,URG,PSH SYN,ACK -j REJECT

# 第三次握手
iptables -t filter -I INPUT -p tcp -m tcp --dport 22 --tcp-flags SYN,ACK,FIN,RST,URG,PSH ACK -j REJECT
```

可以修改一下上述写法:

```bash
# 第一次握手
iptables -t filter -I INPUT -p tcp -m tcp --dport 22 --tcp-flags ALL SYN -j REJECT

# 第二次握手
iptables -t filter -I OUTPUT -p tcp -m tcp --dport 22 --tcp-flags ALL SYN,ACK -j REJECT

# 第三次握手
iptables -t filter -I INPUT -p tcp -m tcp --dport 22 --tcp-flags ALL ACK -j REJECT
```

针对匹配第一次握手, `tcp` 模块中也提供了一个参数来匹配

```bash
# 第一次握手
iptables -t filter -I INPUT -p tcp -m tcp --dport 22 --syn -j REJECT
```

### 用途

使用`-tcp-flags` 匹配时, 可以有效防止 DDOS 攻击, 使用以下规则进行处理

```bash
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j RETURN
iptables -A INPUT -p tcp --syn -j DROP
```

这些规则的作用如下：

- 第一个规则将限制每秒钟只允许一个 SYN 数据包的到达，但在限制突发进程方面，允许最多三个数据包突发
- 第二个规则丢弃所有剩余的 SYN 数据包，从而避免 DDOS 攻击