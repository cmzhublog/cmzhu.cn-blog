## SoftEther VPN 搭建和配置使用

### 背景

1、 家中搭建了nas 服务， 因为宽带没有公网IP，没办法直接访问

2、需要连接家庭中服务器，并对环境进行一些修改

### VPN 架构

#### 配置需求

1、docker

2、docker-compose

3、内网一台linux 服务器

4、公网一台linux服务器

5、frp

#### 架构图

```flow
st=>start: Start

op_home=>operation: SoftEther-Server 暴露5555 端口
op_frp_home=>operation: Frpc 客户端转发5555 端口
op_frp_server=>operation: Frps 服务端对公网暴露5555 端口
op_client=>operation: SoftEther-Client 连接Frps 的5555服务端

e=>end

st->op_home->op_frp_home->op_frp_server->op_client->e

```

### 搭建过程

以下步骤所有机器执行

1、 所有的服务器都安装docker

参考：[手动安装docker](https://wiki.cmzhu.cn/cmzhu.cn-blog/%E5%AE%B9%E5%99%A8/docker/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%8D%B8%E8%BD%BD/04_%E6%89%8B%E5%8A%A8%E5%AE%89%E8%A3%85docker/)

以下步骤在内网执行

2、在内网部署SoftEther-VPN的服务端， 并对外暴露5555 端口

```bash
$ docker run -d \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -p 1701:1701/tcp \
    -p 1194:1194/udp \
    -p 1443:443 \
    -p 5555:5555 \
    -e PSK=abc123! \
    -e USERNAME=root \
    -e PASSWORD=abc123 \
    -e SPW=123456 \
    -e HPW=1234567 \
    siomiz/softethervpn:4.43-opensuse

```

参数解释： 

- 500 （UDP） 端口、4500 （UDP）端口、 1701 （TCP）端口 用于`L2TP/IPSec` 协议的vpn， Windows 和Mac 的代理默认支持此协议
- 1194 （UDP） 端口 用于对OpenVPN 协议 的支持
- 443 （TCP）端口 用于对OpenVPN 的HTTPS 协议的支持
- 5555 （TCP）端口 用于对`SoftEther VPN` 的支持
- PSK 针对L2TP 有用， 用于设置L2TP 的预共享密钥
- USERNAME， PASSWORD 默认用户的用户名和密码
- SPW ： SoftEther VPN管理端的密码
- HPW ： ”DEFAULT“ 虚拟网络的管理端密码

3、在内网服务端安装frpc 服务， 用于向公网转发5555的TCP 端口,注意， 新版本是使用toml 格式文件进行配置， 但是大部分内容不变， 按照实际情况改写即可

```bash
$ docker run \
	--restart=always \
	--network host \
	-d \
	-v /etc/frp/frpc.ini:/etc/frp/frpc.ini \
	--name frpc \
	snowdreamtech/frpc
```

以下步骤在公网服务器操作

4、安装frps 服务端， 并暴露7000 转发端口

```bash
$ docker run \
	--restart=always \
	--network host \
	-d \
	-v /etc/frp/frps.ini:/etc/frp/frps.ini \
	--name frps \
	snowdreamtech/frps
```

5、安装`SoftEther Client` 并进行vpn 连接验证

