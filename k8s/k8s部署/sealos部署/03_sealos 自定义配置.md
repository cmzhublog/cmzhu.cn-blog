## Sealos 自定义配置

Sealos 安装操作系统时, 会有一些特殊的要求, 比如需要指定 Containerd 的主路径, 这些不可避免的会修改配置文件;

Sealos 的配置 都可通过配置文件来运行, 默认通过命令行生成的配置文件, 在~/.sealos/default/Clusterfile 中;也可以不安装, 直接通过命令生成Clusterfile

比如我的机器配置如下

master:

- 192.168.0.2

- 192.168.0.3

- 192.168.0.4

node:

- 192.168.0.5
- 192.168.0.6
- 192.168.0.7

需要安装如下安装包

- labring/kubernetes:v1.25.0 
- labring/helm:v3.8.2 
- labring/calico:v3.24.1

执行sealos gen 命令就可以生成对应的配置文件;

```bash

$ sealos gen labring/kubernetes:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1    --masters 192.168.0.2,192.168.0.3,192.168.0.4    --nodes 192.168.0.5,192.168.0.6,192.168.0.7 --passwd 'xxx' > Clusterfile
```



修改配置文件, 实现调整containerd 的 的存储主路径

```yaml
---
apiVersion: apps.sealos.io/v1beta1
kind: Config
metadata:
  name: containerd-config
spec:
  strategy: override
  # only applied to rootfs
  match: labring/kubernetes:v1.25.0
  path: etc/config.toml
  data: |
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

