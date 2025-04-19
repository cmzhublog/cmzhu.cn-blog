## containerd 支持pull 镜像

### 背景

- 部分镜像仓库没有配置 https 访问, 仅能用http 访问, 因此需要能够支持http的配置

### 配置方式

1、 查看containerd 版本

```bash
$ containerd --version
containerd github.com/containerd/containerd v1.6.24 61f9fd88f79f081d64d6fa3bb1a0dc71ec870523
```

2、 查看/etc/containerd/conf.toml 文件, 发现配置如下

```bash
$ cat /etc/containerd/config.toml
version = 2
root = "/var/lib/containerd"
state = "/run/containerd"
oom_score = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  uid = 0
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216

[debug]
  address = "/run/containerd/containerd-debug.sock"
  uid = 0
  gid = 0
  level = "warn"

[timeouts]
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "sealos.hub:5000/pause:3.8"
    max_container_log_line_size = -1
    max_concurrent_downloads = 20
    disable_apparmor = false
    [plugins."io.containerd.grpc.v1.cri".containerd]
      snapshotter = "overlayfs"
      default_runtime_name = "runc"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          runtime_engine = ""
          runtime_root = ""
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = "/etc/containerd/certs.d"
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
          [plugins."io.containerd.grpc.v1.cri".registry.configs."sealos.hub:5000".auth]
            username = "admin"
            password = "passw0rd"
```

发现配置如上, 其中最为重要的是如下这行配置, 指明了引入配置的路径

```toml
...
[plugins."io.containerd.grpc.v1.cri".registry]
      config_path = "/etc/containerd/certs.d"
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
...
```

3、 进入到/etc/containerd/certs.d 目录

```bash
$  cd /etc/containerd/certs.d &&  ls -la
total 16
drwxr-xr-x 4 root root 4096 Oct 16 04:39 .
drwxr-xr-x 3 root root 4096 Oct 16 04:40 ..
drwxr-xr-x 2 root root 4096 Oct 15 07:07 sealos.hub:5000
```



4、 会发现只有一个默认的sealos.hub:5000(因为集群是sealos 安装的, 这是sealos 自带的配置)

```bash
$ tree
.
└── sealos.hub:5000
    └── hosts.toml

2 directories, 1 files
```

5、 我们需要新建一个配置, 支持访问 dockerhub.bigquant.ai:5000 的访问, 首先按照当前目录结构拷贝目录

```bash
$ cd /etc/containerd/certs.d && mkdir -p dockerhub.bigquant.ai:5000

$ cat dockerhub.cmzhu.cn\:5000/hosts.toml
server = "http://dockerhub.cmzhu.cn:5000"

[host."http://dockerhub.cmzhu.cn:5000"]
  capabilities = ["pull", "resolve", "push"]
  skip_verify = true
```

将配置写入集群, 目录结构如下

```bash
$ tree
.
├── dockerhub.cmzhu.cn:5000
│   └── hosts.toml
└── sealos.hub:5000
    └── hosts.toml

3 directories, 2 files
```

4、 重启containerd 并且观察containerd 是否正常运行

```bash
$ systemctl restart containerd && systemctl status containerd
```

