## docker 重启失败

### 报错

```bash
$ systemctl restart docker
Failed to restart docker.service: Unit docker.service is masked.
```

```bash
$ systemctl status docker
Warning: The unit file, source configuration file or drop-ins of docker.service changed on disk. Run 'systemctl daemon-reload' to >
○ docker.service
     Loaded: masked (Reason: Unit docker.service is masked.)
    Drop-In: /etc/systemd/system/docker.service.d
             └─proxy.conf
     Active: inactive (dead)
```



```bash

```





### 解决思路

1、 检查docker 服务配置是否正常, 发现docker.service 是空的

```bash
$ cat /usr/lib/systemd/system/docker.service
```

2、 补齐 /usr/lib/systemd/system/docker.service 的配置

```bash
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

3、 重启docker, 但是发现重启失败, 查看日志发现

```bash
$ systemctl restart docker 
```

```bash
$ journalctl -xe -u docker
...
Mar 27 10:18:03 dev-zyan systemd[1]: docker.socket: Failed to create listening socket (/run/docker.sock): Address already in use
Mar 27 10:18:03 dev-zyan systemd[1]: docker.socket: Failed to listen on sockets: Address already in use
Mar 27 10:18:03 dev-zyan systemd[1]: docker.socket: Failed with result 'resources'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: https://wiki.rockylinux.org/rocky/support
░░
░░ The unit docker.socket has entered the 'failed' state with result 'resources'.
Mar 27 10:18:03 dev-zyan systemd[1]: Failed to listen on Docker Socket for the API.
░░ Subject: A start job for unit docker.socket has failed
░░ Defined-By: systemd
░░ Support: https://wiki.rockylinux.org/rocky/support
░░
░░ A start job for unit docker.socket has finished with a failure.
░░
░░ The job identifier is 7771 and the job result is failed.
Mar 27 10:18:03 dev-zyan systemd[1]: Dependency failed for Docker Application Container Engine.
░░ Subject: A start job for unit docker.service has failed
░░ Defined-By: systemd
░░ Support: https://wiki.rockylinux.org/rocky/support
░░
░░ A start job for unit docker.service has finished with a failure.
░░
░░ The job identifier is 7684 and the job result is dependency.
...
```

4、 查看/run/docker.sock, 发现是一个空目录

```bash
$ ll /run/docker.sock
```

5、删除目录, 重新启动

```bash
$ rmdir /run/docker.sock
```

6、 重启docker 解决

```bash
$ systemctl enable --now docker
```

