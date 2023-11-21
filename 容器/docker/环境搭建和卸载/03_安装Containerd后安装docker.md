## 安装Containerd 后安装docker

参考官网文档： <https://docs.docker.com/engine/install/binaries/#install-daemon-and-client-binaries-on-linux>

下载 docker 二进制版本，请选择最新最稳定的CE版本

<https://download.docker.com/linux/static/stable/x86_64/>

```bash
export DOCKER_VERSION=v23.0.4

cd $(mktemp -d)

wget https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_VERSION/v/}.tgz -O docker-ce.tgz

tar zxvf docker-ce.tgz -C .
```

移动所需二进制文件，建立软链

```bash
mv ./docker /usr/local/docker-${DOCKER_VERSION/v/}

ln -s /usr/local/docker-${DOCKER_VERSION/v/}/dockerd /usr/bin/dockerd
ln -s /usr/local/docker-${DOCKER_VERSION/v/}/docker-proxy /usr/bin/docker-proxy
ln -s /usr/local/docker-${DOCKER_VERSION/v/}/docker /usr/bin/docker
ln -s /usr/local/docker-${DOCKER_VERSION/v/}/docker-init /usr/bin/docker-init

ln -s /usr/local/docker-${DOCKER_VERSION/v/}/dockerd /usr/local/bin/dockerd
ln -s /usr/local/docker-${DOCKER_VERSION/v/}/docker-proxy /usr/local/bin/docker-proxy
ln -s /usr/local/docker-${DOCKER_VERSION/v/}/docker /usr/local/bin/docker
ln -s /usr/local/docker-${DOCKER_VERSION/v/}/docker-init /usr/local/bin/docker-init

```

查看二进制和软链

```bash
ls -al /usr/local/docker-${DOCKER_VERSION/v/}
ls -al /usr/bin/docker*
ls -al /usr/local/bin/docker*
```

创建一个 system 管理文件，参考

- <https://github.com/moby/moby/blob/v23.0.4/contrib/init/systemd/docker.servic>

```bash
wget https://raw.githubusercontent.com/moby/moby/v23.0.4/contrib/init/systemd/docker.service \
  -O /usr/lib/systemd/system/docker.service
```

配置文件内容如下：

```ini
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service containerd.service time-set.target
Wants=network-online.target containerd.service
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutStartSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target
```

创建 docker.socket，参考

- <https://github.com/moby/moby/blob/v23.0.4/contrib/init/systemd/docker.socket>

```bash
curl -L https://raw.githubusercontent.com/moby/moby/v20.10.19/contrib/init/systemd/docker.socket \
  -o /usr/lib/systemd/system/docker.socket
```

内容如下：

```bash
[Unit]
Description=Docker Socket for the API

[Socket]
# If /var/run is not implemented as a symlink to /run, you may need to
# specify ListenStream=/var/run/docker.sock instead.
ListenStream=/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
```

把上面的 group 改为 root

配置 docker 服务端

```bash
mkdir -p /etc/docker
vim /etc/docker/daemon.json 
```

内容如下：

```bash
{
    "debug": false,
    "insecure-registries": [
      "0.0.0.0/0"
    ],
    "ip-forward": true,
    "ipv6": false,
    "live-restore": true,
    "log-driver": "json-file",
    "log-level": "warn",
    "log-opts": {
      "max-size": "100m",
      "max-file": "2"
    },
    "selinux-enabled": false,
    "metrics-addr" : "0.0.0.0:9323",
    "experimental" : true,
    "storage-driver": "overlay2"
}
```

重启进程

```bash
systemctl daemon-reload
systemctl start docker.service

# 设置开机自动启动和启动docker 进程
systemctl enable --now docker
```