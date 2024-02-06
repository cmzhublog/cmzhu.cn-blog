## k8s GPU 配置和使用

文档参考: [https://github.com/NVIDIA/k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)

`NVIDIA device plugin` 可以作为一个Daemonset , 可以自动做到:

- 公开在k8s 节点上所有的显卡数量
- 跟踪GPU 的运行状态
- 在k8s 容器中启动nvidia 显卡

依赖

- NVIDIA driver ~= 384.81
- nvidia-docker >= 2.0 || nvidia-container-toolkit >= 1.7.0 (>= 1.11.0 to use integrated GPUs on Tegra-based systems)
- nvidia-container-runtime configured as the default low-level runtime
- Kubernetes version >= 1.10

### 开始安装

####  配置GPU 节点

1、 配置nvidia 的GPU 节点

安装nvidia-container-toolkit, 参照: [https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

例子: 

在Rocky 9 或者Centos7 

```bash
## 1、配置源
$ curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | \
  sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo

## 2、 配置仓库使用nvidia-container-toolkit 的源
$ sudo yum-config-manager --enable nvidia-container-toolkit-experimental

## 3、 安装 nvidia-container-toolkit
$ sudo yum install -y nvidia-container-toolkit
```

2、配置容器运行时(runc: docker)

当容器运行时是docker 的时候, 需要配置 /etc/docker/daemon.json

```json
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

然后重启docker

```bash
$ systemctl restart docker
```

3、 配置容器运行时(runc: containerd)

当容器运行时是containerd 的时候, 需要修改/etc/containerd/config.toml 

```toml
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
```

然后重启containerd

```bash
$ sudo systemctl restart containerd
```

3、 在集群中的所有 GPU 节点上配置上述选项后，可以通过部署以下 Daemonset 来启用 GPU 支持

```bash
$ kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.14.4/nvidia-device-plugin.yml
```

4、 启动一个测试实例

```bash
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  restartPolicy: Never
  containers:
    - name: cuda-container
      image: nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda10.2
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
  tolerations:
  - key: nvidia.com/gpu
    operator: Exists
    effect: NoSchedule
EOF
```



5、 配置 NVIDIA device plugin binary

NVIDIA设备插件具有许多可以为其配置的选项。这些选项可以配置为命令行标志、环境变量，也可以在启动设备插件时通过配置文件进行配置。在这里，我们将解释这些选项中的每一个是什么，以及如何直接针对插件二进制文件配置它们。以下部分说明在通过 `helm` 部署插件时如何设置这些配置。

在命令中的配置

| Flag                     | Envvar                  | Default Value |
| ------------------------ | ----------------------- | ------------- |
| `--mig-strategy`         | `$MIG_STRATEGY`         | `"none"`      |
| `--fail-on-init-error`   | `$FAIL_ON_INIT_ERROR`   | `true`        |
| `--nvidia-driver-root`   | `$NVIDIA_DRIVER_ROOT`   | `"/"`         |
| `--pass-device-specs`    | `$PASS_DEVICE_SPECS`    | `false`       |
| `--device-list-strategy` | `$DEVICE_LIST_STRATEGY` | `"envvar"`    |
| `--device-id-strategy`   | `$DEVICE_ID_STRATEGY`   | `"uuid"`      |
| `--config-file`          | `$CONFIG_FILE`          | `""`          |

Helm 的configfile: 

```yaml
version: v1
flags:
  migStrategy: "none"
  failOnInitError: true
  nvidiaDriverRoot: "/"
  plugin:
    passDeviceSpecs: false
    deviceListStrategy: "envvar"
    deviceIDStrategy: "uuid"
```

#### 例子

1、 配置某一张网卡使用特定的显卡, 配置时可以在Pod 中通过环境变量配置`NVIDIA_VISIBLE_DEVICES`

```bash
# 表示使用节点上的pcie 编号为3 的那张gpu设备
NVIDIA_VISIBLE_DEVICES: 3
```

