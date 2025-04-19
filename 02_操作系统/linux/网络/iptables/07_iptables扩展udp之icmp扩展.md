## `iptables` 扩展`udp`之 `icmp`扩展

### udp 扩展

`udp` 扩展中使用的匹配条件较少, 只有两个 `--dport`和`--sport` ,即只匹配报文的源端口和目标端口; 例如:

放行`samba` 服务使用了`udp` 的`137`,`138` 端口, 

 ```bash
iptables -t filter -I INPUT -p udp -m udp --dport 137 -j ACCEPT
iptables -t filter -I INPUT -p udp -m udp --dport 138 -j ACCEPT
 ```

### `icmp` 扩展

现在聊聊`icmp`扩展，没错，看到`icmp`，你肯定就想到了`ping`命令，因为`ping`命令使用的就是`icmp`协议。

`ICMP`协议的全称为`Internet Control Message Protocol`，翻译为互联网控制报文协议，它主要用于探测网络上的主机是否可用，目标是否可达，网络是否通畅，路由是否可用等。

我们平常使用`ping`命令`ping`某主机时，如果主机可达，对应主机会对我们的`ping`请求做出回应（此处不考虑禁`ping`等情况），也就是说，我们发出`ping`请求，对方回应`ping`请求，虽然`ping`请求报文与`ping`回应报文都属于`ICMP`类型的报文，但是如果在概念上细分的话，它们所属的类型还是不同的，我们发出的`ping`请求属于类型`8`的`icmp`报文，而对方主机的`ping`回应报文则属于类型`0`的`icmp`报文，根据应用场景的不同，`icmp`报文被细分为如下各种类型。

![img](https://www.zsythink.net/wp-content/uploads/2017/05/050117_1112_4.png)

从上图可以看出，所有表示”目标不可达”的`icmp`报文的`type`码为`3`，而”目标不可达”又可以细分为多种情况，是网络不可达呢？还是主机不可达呢？再或者是端口不可达呢？所以，为了更加细化的区分它们，`icmp`对每种`type`又细分了对应的`code`，用不同的`code`对应具体的场景，  所以，我们可以使用`type/code`去匹配具体类型的`ICMP`报文，比如可以使用”3/1″表示主机不可达的icmp报文。

上图中的第一行就表示`ping`回应报文，它的`type`为`0`，`code`也为`0`，从上图可以看出，`ping`回应报文属于查询类（`query`）的`ICMP`报文，从大类上分，`ICMP`报文还能分为查询类与错误类两大类，目标不可达类的`icmp`报文则属于错误类报文。

而我们发出的ping请求报文对应的`type`为`8`，`code`为`0`。

了解完概念: 

来看看实际应用场景

```bash
# 禁止所有icmp 报文进入本机
iptables -t filter -I INPUT -p icmp -j DROP

```

例子中; 使用 `-p icmp ` 匹配了所有的icmp 报文; 如果进行了上述设置, 任何别的主机向我发送的 icmp 的报文,都会被丢弃;

假设, 我们目前需求有变; 只想让我们ping通别人, 但不想让别人ping 通我, 刚刚的配置就不再满足了; 可以进行如下配置

```bash
# 允许ping 别人, 别人不允许ping 我
iptables -t filter -I INPUT -p icmp -m icmp --icmp-type 8/0 -j REJECT
```

指令中;

`-m icmp `:  表示使用`icmp` 模块

`--icmp-type 8/0`:  表示具体的type 和code 去匹配对应的`icmp `报文;因为` type` 8 下只有一个`code 0`; 所以指令可以省略为

```bash
iptables -t filter -I INPUT -p icmp -m icmp --icmp-type 8 -j REJECT
```

除了使用对应的 `type/code` 外, 还可以使用对应的描述名称去匹配对应类型的报文; 

```bash
iptables -t filter -I INPUT -p icmp -m icmp --icmp-type "echo-request" -j REJECT
```

> 注意: 名称中的空格 需替换成 "-"