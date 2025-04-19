## 单个 GPU 中的七个独立实例

多实例 GPU (MIG) 扩展了每个 [NVIDIA H100](https://www.nvidia.cn/data-center/h100/)、[A100](https://www.nvidia.cn/data-center/a100/) 及 [A30](https://www.nvidia.cn/data-center/products/a30-gpu/) Tensor Core GPU 的性能和价值。MIG 可将 GPU 划分为多达七个实例，每个实例均完全独立于各自的高带宽显存、缓存和计算核心。如此一来，管理员便能支持所有大小的工作负载，且服务质量 (QoS) 稳定可靠，让每位用户都能享用加速计算资源。

### MIG 规格

|                |                          H100                           | A100                                    |
| :------------: | :-----------------------------------------------------: | --------------------------------------- |
|  支持机密计算  |                          支持                           |                                         |
|    实例类型    | 7个 @10GB 4个 @20GB 2个 @40GB（计算能力更强） 1个 @80GB | 7个 @10GB 3个 @20GB 2个 @40GB 1个 @80GB |
| GPU 分析和监控 |                  在所有实例上并行运行                   | 一次仅一个实例                          |
|    安全租户    |                           7x                            | 1x                                      |
|   媒体解码器   |             每个实例专用的 NVJPEG 和 NVDEC              | 有限选项                                |

## GPU Operator

### 简介

![img](./01_NVIDIA GPU-Operator.assets/nvidia-gpu-operator-image9.jpg)

Kubernetes提供了通过设备插件框架访问特殊硬件资源，如NVIDIA GPU、NIC、Infiniband适配器和其他设备的功能。但是，配置和管理具有这些硬件资源的节点需要配置多个软件组件，如驱动程序、容器运行时或其他库，这很困难且容易出错。NVIDIA GPU Operator使用Kubernetes内的操作员框架自动化管理所有用于GPU提供的NVIDIA软件组件。这些组件包括NVIDIA驱动程序（以启用CUDA）、用于GPU的Kubernetes设备插件、NVIDIA Container Toolkit、使用GFD进行自动节点标记、基于DCGM的监控等。

### 管理组件

**NVIDIA-container-toolkit**

NVIDIA-container-toolkit 是 NVIDIA 提供的一款开源工具，用于在 Docker 和 Kubernetes 环境中支持 NVIDIA GPU。它包含了一系列工具和库，可以帮助用户轻松地为容器应用程序启用 NVIDIA GPU 加速。

NVIDIA-container-toolkit 的主要功能包括：

1. NVIDIA runtime：一款基于 NVIDIA 官方驱动程序的容器运行时，支持在容器内直接加载和使用 NVIDIA GPU 资源，避免了在容器外部安装和配置 GPU 驱动程序的麻烦；
2. NVIDIA Docker plugin：将 NVIDIA runtime 集成到 Docker 中，使得用户可以通过简单的 Docker 命令为容器添加 GPU 支持；
3. NVIDIA Container Runtime for Kubernetes：为 Kubernetes 集群提供 NVIDIA GPU 支持，支持自定义 GPU 配置、GPU 调度等高级特性。

使用 NVIDIA-container-toolkit，用户可以方便地在容器中使用 NVIDIA GPU，而无需手动安装和配置驱动程序和相关库。同时，其还支持多种操作系统和硬件平台，如 Linux、Windows 和 ARM 架构等，更好地满足不同场景下的需求。

**node-feature-discovery**

ode-feature-discovery 是 Kubernetes 的一个插件，用于自动探测节点上的硬件和软件特性，并将它们作为标签添加到节点上。这些标签可以用于调度 Pod、限制 Pod 调度位置等。

该插件使用各种探测器来识别节点上的功能，例如 CPU、GPU、网络、存储、操作系统等。通过将这些功能映射到 Kubernetes 资源的标签上，node-feature-discovery 可以帮助集群管理员更好地管理节点，优化资源利用率，并提高整个集群的性能。node-feature-discovery 还支持自定义探测器和标签，使其具有灵活性和可扩展性。

**gpu-feature-discovery**

gpu-feature-discovery 是基于 node-feature-discovery 的一个插件，用于自动发现 Kubernetes 节点上的 GPU 特性并添加相应的标签。它可以自动识别节点上的 NVIDIA GPU，并将其型号、驱动版本等信息作为标签添加到节点上，从而方便在 Kubernetes 中调度 GPU 工作负载。

gpu-feature-discovery 可以与 Kubernetes 的多个组件集成，例如 Device Plugin、DaemonSet 等，使 GPU 的资源能够被更好地管理和利用。此外，gpu-feature-discovery 还支持用户自定义的 GPU 标签和探测器，以满足不同的需求。

总之，gpu-feature-discovery 可以帮助用户更好地管理 Kubernetes 集群中的 GPU 资源，并提高 GPU 工作负载的性能和可靠性。

**mig-manager**

mig-manager 是 NVIDIA 的一个 Kubernetes 插件，用于管理 NVIDIA Multi-Instance GPU (MIG)。MIG 是 NVIDIA Ampere 架构中的一个新特性，可以将一块物理 GPU 划分为多个虚拟 GPU，从而提高 GPU 的利用率和灵活性。

mig-manager 可以自动检测节点上的 MIG 资源，并将其作为 Kubernetes 节点资源暴露给 Kubernetes 调度器。当需要运行 GPU 工作负载时，Kubernetes 调度器可以根据工作负载要求选择合适的 MIG 实例，并将其分配给工作负载使用。此外，mig-manager 还可以监控 MIG 实例的状态并自动重启故障实例，确保 GPU 工作负载的高可靠性和稳定性。

**nvidia-device-plugin**

nvidia Device Plugin 是一个 Kubernetes 插件，用于管理 NVIDIA GPU 资源。它可以在 Kubernetes 集群中自动发现节点上的 NVIDIA GPU，并将其作为 Kubernetes 节点资源暴露给 Kubernetes 调度器。

通过 nvidia Device Plugin，用户可以为 Kubernetes 中的 GPU 工作负载提供可靠和高效的资源管理。它可以自动检测 NVIDIA GPU 的数量和型号，并将其作为 Kubernetes 节点资源暴露出来，从而使 Kubernetes 调度器可以根据工作负载的需求选择合适的 GPU 资源来运行工作负载。此外，nvidia Device Plugin 还支持多租户模式，可以根据 Kubernetes 命名空间限制 GPU 资源的使用。

nvidia Device Plugin 还支持 Docker 和 CRI-O 等容器运行时，可以与 Kubernetes 中的其他插件集成，例如 kubelet、kube-proxy、kube-scheduler 等。同时，nvidia Device Plugin 还支持 Prometheus 监控和日志记录，以便用户更好地监控和优化 GPU 工作负载的性能和可靠性。

**dcgm-exporter**

dcgm exporter 是 NVIDIA 提供的一款开源软件，它可以收集并将 NVIDIA GPU 的监控数据暴露给 Prometheus 监控系统。dcgm exporter 使用 NVIDIA Data Center GPU Manager (DCGM) 库收集 GPU 监控数据，并将其转换为 Prometheus 可以理解的格式。

通过 dcgm exporter，用户可以方便地监控 NVIDIA GPU 的使用情况和性能指标，例如 GPU 温度、功耗、显存使用率等。dcgm exporter 还支持多种 GPU 监控指标，用户可以根据自己的需求选择需要监控的指标。

dcgm exporter 非常容易与 Prometheus 集成，只需要简单的配置即可将监控数据暴露给 Prometheus

### 部署

**部署环境**

操作系统：CentOS7.9

CPU架构：X86_64

kubernetes：v1.19.6

docker：20.10.9

**安装 gpu operator**

由于 gpu-operator 与 kubernetes ,操作系统绑定的版本较为严格，在其他环境不按照此文档操作不一定能够成功。GPU Operator Helm chart 提供了许多可自定义的选项，可以根据您的环境进行配置。以下是部分的参数，可用 --set 配置。

| 参数                     | 描述                                                         | 默认                                                       |
| ------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- |
| cdi.enabled              | 设置为 true 时，Operator 会安装两个额外的运行时类 nvidia-cdi 和 nvidia-legacy，并允许使用容器设备接口 (CDI) 使容器可以访问 GPU。 使用 CDI 使 Operator 与最近标准化 GPU 等复杂设备如何暴露于容器化环境的努力保持一致。 Pod 可以将 spec.runtimeClassName 指定为 nvidia-cdi 以使用该功能或指定 nvidia-legacy 以防止使用 CDI 执行设备注入。 | false                                                      |
| cdi.default              |                                                              | false                                                      |
| daemonsets.annotations   |                                                              | {}                                                         |
| daemonsets.labels        |                                                              | {}                                                         |
| driver.enabled           | 官方的驱动镜像会与操作系统进行强依赖关系绑定，建议手动安装驱动 | true                                                       |
| driver.repository        |                                                              | [dockerhub.bigquant.ai:5000/nvidia](http://nvcr.io/nvidia) |
| driver.rdma.enabled      |                                                              | false                                                      |
| driver.rdma.useHostMofed |                                                              | false                                                      |
| driver.usePrecompiled    | 当设置为true时，运算符将尝试部署具有预编译内核驱动程序的驱动程序容器。 | false                                                      |
| driver.version           | Operator 支持的 NVIDIA 数据中心驱动程序版本。 如果将 driver.usePrecompiled 设置为 true，则将此字段设置为驱动程序分支，例如 525 | 目前仅支持                                                 |
| mig.strategy             | 控制在受支持的 NVIDIA GPU 上与 MIG 一起使用的策略。选项是混合的或单一的。single 会以 http://nvidia.com/gpu 格式显示 gpu 的数量，mixed 以实际划分的规格展示资源。 | single                                                     |
| migManager.enabled       | MIG 管理器监视 MIG 几何结构的变化并根据需要应用重新配置。默认情况下，MIG 管理器仅在具有支持 MIG 的 GPU 的节点上运行（例如 A100）。 | true                                                       |
| nfd.enabled              | 将 Node Feature Discovery 插件部署为守护进程。如果 NFD 已在集群中运行，则将此变量设置为 false。 | true                                                       |
| toolkit.enabled          | 默认情况下，Operator 将 NVIDIA Container Toolkit（nvidia-docker2 堆栈）部署为系统上的容器。 在预装 NVIDIA 运行时的系统上使用 Operator 时，将此值设置为 false。 | true                                                       |
| toolkit.version          |                                                              |                                                            |
| operator.labels          | 将添加到所有 GPU operator 管理的 pod 的自定义标签的映射。    | {}                                                         |
| psp.enabled              | 如果启用，GPU Operator 会部署 PodSecurityPolicies            | false                                                      |
| toolkit.enabled          | 默认情况下，Operator 将 NVIDIA 容器工具包（nvidia-docker2 堆栈）部署为系统上的容器。在预装 NVIDIA 运行时的系统上使用 Operator 时，将此值设置为 false。 | true                                                       |

执行 安装命令

```
helm install --wait -n gpu-opeator --create-namespace \  --set nfd.enabled=true \  --set operator.defaultRuntime=docker \  --set operator.runtimeClass=nvidia \  --set mig.strategy=mixed \  --set driver.enabled=false \  --set driver.usePrecompiled=true \  --set driver.version=525.105.17 \  --set toolkit.enabled=false \  --set toolkit.repository=dockerhub.bigquant.ai:5000/nvidia/k8s \  --set toolkit.image=container-toolkit \  --set toolkit.version=v1.13.1 \  --set devicePlugin.enabled=true \  --set devicePlugin.repository=dockerhub.bigquant.ai:5000/nvidia \  --set devicePlugin.image=k8s-device-plugin \  --set devicePlugin.version=v0.12.2-ubi8 \  --set dcgmExporter.enabled=true \  --set dcgmExporter.repository=dockerhub.bigquant.ai:5000/nvidia/k8s \  --set dcgmExporter.image=dcgm-exporter \  --set dcgmExporter.version=3.1.7-3.1.4-ubuntu20.04 \  --set gfd.enabled=true \  --set gfd.repository=dockerhub.bigquant.ai:5000/nvidia \  --set gfd.version=v0.6.1-ubi8 \  --set migManager.enabled=true \  --set migManager.repository=dockerhub.bigquant.ai:5000/nvidia/cloud-native \  --set migManager.version=v0.4.2-ubuntu20.04 \  --set migManager.config.name=default-mig-parted-config \  --set sandboxDevicePlugin.enabled=false \  --set node-feature-discovery.image.repository=dockerhub.bigquant.ai:5000/nfd/node-feature-discovery \  --set node-feature-discovery.image.tag=v0.10.1 \  --set validator.image.repository=dockerhub.bigquant.ai:5000/nvidia/cloud-native/gpu-operator-validator \  --set validator.image.tag=v1.11.0 \  gpu-operator nvidia/gpu-operator --version v1.11.0
```

## 切分 GPU 为多个实例

有了 opeator 之后一起都变得轻松起来，主要给节点添加一个指定规格的标签就可以自动完成 mig 的划分，如果配置不生效或者切分失败，建议重启主机以解决问题。以下是指定 GPU 型号支持的规格。

|           | all-disabled | all-1g.10gb | all-2g.20gb | all-3g.40gb | all-7g.80gb | all-balanced                         |
| --------- | ------------ | ----------- | ----------- | ----------- | ----------- | ------------------------------------ |
| A100-40GB |              |             |             |             |             |                                      |
| A100-80GB | 0            | 7           | 3           | 2           | 1           | "1g.10gb": 2"2g.20gb": 1"3g.40gb": 1 |

为装配有 A100 节点划分 1g.10gb 规格的 MIG，总计可以切分为 7 等分，也就是 7 个卡。实际切分的工作是由 **[mig-parted](https://github.com/NVIDIA/mig-parted)** 完成 ，如切分失败可查看 mig-pared-manager 的日志。切分成功之后通过以下命令查看

```
kubernetes lable nodes <node> nvidia.com/mig.config=all-1g.10gb
```



```
nvidia-smi 
```

## 参考

- https://github.com/NVIDIA/mig-parted/blob/main/deployments/gpu-operator/nvidia-mig-manager-example.yaml#L175
- https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/gpu-operator-mig.html#disabling-mig