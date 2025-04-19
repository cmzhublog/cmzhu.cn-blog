## docker 离线安装步骤

参考官网文档： <https://docs.docker.com/engine/install/binaries/#install-daemon-and-client-binaries-on-linux>

下载 docker 二进制版本，请选择最新最稳定的CE版本

<https://download.docker.com/linux/static/stable/x86_64/>

```bash
export DOCKER_VERSION=v23.0.4

cd $(mktemp -d)

wget https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_VERSION/v/}.tgz -O docker-ce.tgz

tar zxvf docker-ce.tgz -C .
```



1、 开始安装docker, 将所有的二进制文件拷贝到`/usr/bin` 目录中

```bash
$ cp docker/* /usr/bin
```

2、配置docker 服务 /etc/systemd/system/docker.service

```service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target


[Service]
Type=notify
ExecStart=/usr/bin/dockerd
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



3、重新加载systemd

```bash
$ chmod +x /etc/systemd/system/docker.service
$ systemctl daemon-reload
```

4、启动docker 进程

```bash
# 开机启动
$ systemctl enable --now docker.service
# 启动docker
$ systemctl start docker
# docker状态
$ systemctl status docker
# 重启docker服务
$ systemctl restart docker
```

