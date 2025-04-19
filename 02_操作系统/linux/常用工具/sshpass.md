## sshpass 命令

-  centos安装

```bash
$ yum install -y epel-release
$ yum install sshpass
```

如果没有该版本的rpm 包， 可以参考二进制安装

1、从[https://sourceforge.net/projects/sshpass/files/sshpass/](https://sourceforge.net/projects/sshpass/files/sshpass/) 下载指定版本的 sshpass， 这里以1.10 版本为例子

```bash
$ tar -zxvf sshpass-1.05.tar.gz
$ cd sshpass-1.05

#指定安装目录
$ ./configure --prefix=/opt/sshpass
$ make
$ make install
$ cp /opt/sshpass/bin/sshpass /usr/bin/
```

2、sshpass -V 验证安装完成

```bash
$ /opt/sshpass/bin/sshpass -V
sshpass 1.10
(C) 2006-2011 Lingnu Open Source Consulting Ltd.
(C) 2015-2016, 2021-2022 Shachar Shemesh
This program is free software, and can be distributed under the terms of the GPL
See the COPYING file for more information.

Using "assword" as the default password prompt indicator.
```



例子，使用sshpass 反向代理端口

1、创建service文件（/etc/systemd/system/nacos-sshpass.service）

```bash
[Unit]
Description=Nacos Application  Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target


[Service]
Type=simple
ExecStart=/opt/sshpass/bin/sshpass -p 'abc123!' ssh -L 0.0.0.0:32659:localhost:32659 -C -q -N root@192.168.101.100
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s


[Install]
WantedBy=multi-user.target

```

实现功能为将 192.168.101.100上的32659端口，正向代理到本地使用localhost:32659 进行访问。

使用下述命令启动服务`nacos-sshpass`

```bash
$ chmod +x /etc/systemd/system/nacos-sshpass.service
$ systemctl enable --now nacos-sshpass.service
```

