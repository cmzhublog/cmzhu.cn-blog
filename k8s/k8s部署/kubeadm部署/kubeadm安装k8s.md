# kubeadm 安装 k8s



## 安装前检查

1、 **检查设备唯一性**

- 你可以使用命令 `ip link` 或 `ifconfig -a` 来获取网络接口的 MAC 地址
- 可以使用 `sudo cat /sys/class/dmi/id/product_uuid` 命令对 product_uuid 校验

一般来讲，硬件设备会拥有唯一的地址，但是有些虚拟机的地址可能会重复。 Kubernetes 使用这些值来唯一确定集群中的节点。 如果这些值在每个节点上不唯一，可能会导致安装[失败](https://github.com/kubernetes/kubeadm/issues/31)。



2、 **检查网络适配器**

如果你有一个以上的网络适配器，同时你的 Kubernetes 组件通过默认路由不可达，我们建议你预先添加 IP 路由规则， 这样 Kubernetes 集群就可以通过对应的适配器完成连接。

3、 检查所需要的端口

启用这些[必要的端口](https://kubernetes.io/zh-cn/docs/reference/networking/ports-and-protocols/)后才能使 Kubernetes 的各组件相互通信。 可以使用 netcat 之类的工具来检查端口是否启用，例如：

```shell
nc 127.0.0.1 6443
```

你使用的 Pod 网络插件 (详见后续章节) 也可能需要开启某些特定端口。 由于各个 Pod 网络插件的功能都有所不同，请参阅他们各自文档中对端口的要求。



## 安装前准备

1、 安装容器运行时

为了在 Pod 中运行容器，Kubernetes 使用 [容器运行时（Container Runtime）](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes)。

默认情况下，Kubernetes 使用 [容器运行时接口（Container Runtime Interface，CRI）](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#container-runtime) 来与你所选择的容器运行时交互。

如果你不指定运行时，kubeadm 会自动尝试通过扫描已知的端点列表来检测已安装的容器运行时。

如果检测到有多个或者没有容器运行时，kubeadm 将抛出一个错误并要求你指定一个想要使用的运行时。

参阅[容器运行时](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/) 以了解更多信息。

**说明：**

Docker Engine 没有实现 [CRI](https://kubernetes.io/zh-cn/docs/concepts/architecture/cri/)， 而这是容器运行时在 Kubernetes 中工作所需要的。 为此，必须安装一个额外的服务 [cri-dockerd](https://github.com/Mirantis/cri-dockerd)。 cri-dockerd 是一个基于传统的内置 Docker 引擎支持的项目， 它在 1.24 版本从 kubelet 中[移除](https://kubernetes.io/zh-cn/dockershim)。



下面的表格包括被支持的操作系统的已知端点。

| 运行时                            | Unix 域套接字                                |
| --------------------------------- | -------------------------------------------- |
| containerd                        | `unix:///var/run/containerd/containerd.sock` |
| CRI-O                             | `unix:///var/run/crio/crio.sock`             |
| Docker Engine（使用 cri-dockerd） | `unix:///var/run/cri-dockerd.sock`           |



## 安装容器运行时

### 背景

> **说明：** 自 1.24 版起，Dockershim 已从 Kubernetes 项目中移除。阅读 [Dockershim 移除的常见问题](https://kubernetes.io/zh-cn/dockershim)了解更多详情。

你需要在集群内每个节点上安装一个 [容器运行时](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes) 以使 Pod 可以运行在上面。

> **说明：**v1.24 之前的 Kubernetes 版本直接集成了 Docker Engine 的一个组件，名为 **dockershim**。 这种特殊的直接整合不再是 Kubernetes 的一部分 （这次删除被作为 v1.20 发行版本的一部分[宣布](https://kubernetes.io/zh-cn/blog/2020/12/08/kubernetes-1-20-release-announcement/#dockershim-deprecation)）。
>
> 你可以阅读[检查 Dockershim 移除是否会影响你](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/migrating-from-dockershim/check-if-dockershim-removal-affects-you/)以了解此删除可能会如何影响你。 要了解如何使用 dockershim 进行迁移， 请参阅[从 dockershim 迁移](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/migrating-from-dockershim/)。
>
> 如果你正在运行 v1.26 以外的 Kubernetes 版本，查看对应版本的文档。

### 安装和配置先决条件

1、转发 IPv4 并让 iptables 看到桥接流量

执行以下指令

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system
```

通过运行以下指令确认 `br_netfilter` 和 `overlay` 模块被加载：

```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```

通过运行以下指令确认 `net.bridge.bridge-nf-call-iptables`、`net.bridge.bridge-nf-call-ip6tables` 和 `net.ipv4.ip_forward` 系统变量在你的 `sysctl` 配置中被设置为 1：

```bash
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

### cgroup 驱动

在 Linux 上，[控制组（CGroup）](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-cgroup)用于限制分配给进程的资源。

[kubelet](https://kubernetes.io/docs/reference/generated/kubelet) 和底层容器运行时都需要对接控制组来强制执行 [为 Pod 和容器管理资源](https://kubernetes.io/zh-cn/docs/concepts/configuration/manage-resources-containers/) 并为诸如 CPU、内存这类资源设置请求和限制。若要对接控制组，kubelet 和容器运行时需要使用一个 **cgroup 驱动**。 关键的一点是 kubelet 和容器运行时需使用相同的 cgroup 驱动并且采用相同的配置。

可用的 cgroup 驱动有两个：

- [`cgroupfs`](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#cgroupfs-cgroup-driver)
- [`systemd`](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#systemd-cgroup-driver)

### cgroupfs 驱动

`cgroupfs` 驱动是 kubelet 中默认的 cgroup 驱动。当使用 `cgroupfs` 驱动时， kubelet 和容器运行时将直接对接 cgroup 文件系统来配置 cgroup。

当 [systemd](https://www.freedesktop.org/wiki/Software/systemd/) 是初始化系统时， **不** 推荐使用 `cgroupfs` 驱动，因为 systemd 期望系统上只有一个 cgroup 管理器。 此外，如果你使用 [cgroup v2](https://kubernetes.io/zh-cn/docs/concepts/architecture/cgroups)， 则应用 `systemd` cgroup 驱动取代 `cgroupfs`。

### systemd cgroup 驱动

当某个 Linux 系统发行版使用 [systemd](https://www.freedesktop.org/wiki/Software/systemd/) 作为其初始化系统时，初始化进程会生成并使用一个 root 控制组（`cgroup`），并充当 cgroup 管理器。

systemd 与 cgroup 集成紧密，并将为每个 systemd 单元分配一个 cgroup。 因此，如果你 `systemd` 用作初始化系统，同时使用 `cgroupfs` 驱动，则系统中会存在两个不同的 cgroup 管理器。

同时存在两个 cgroup 管理器将造成系统中针对可用的资源和使用中的资源出现两个视图。某些情况下， 将 kubelet 和容器运行时配置为使用 `cgroupfs`、但为剩余的进程使用 `systemd` 的那些节点将在资源压力增大时变得不稳定。

当 systemd 是选定的初始化系统时，缓解这个不稳定问题的方法是针对 kubelet 和容器运行时将 `systemd` 用作 cgroup 驱动。

要将 `systemd` 设置为 cgroup 驱动，需编辑 [`KubeletConfiguration`](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubelet-config-file/) 的 `cgroupDriver` 选项，并将其设置为 `systemd`。例如：

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
...
cgroupDriver: systemd
```

如果你将 `systemd` 配置为 kubelet 的 cgroup 驱动，你也必须将 `systemd` 配置为容器运行时的 cgroup 驱动。 参阅容器运行时文档，了解指示说明。例如：

- [containerd](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#containerd-systemd)
- [CRI-O](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/#cri-o)

**注意：**

> 注意：更改已加入集群的节点的 cgroup 驱动是一项敏感的操作。 如果 kubelet 已经使用某 cgroup 驱动的语义创建了 Pod，更改运行时以使用别的 cgroup 驱动，当为现有 Pod 重新创建 PodSandbox 时会产生错误。 重启 kubelet 也可能无法解决此类问题。
> 如果你有切实可行的自动化方案，使用其他已更新配置的节点来替换该节点， 或者使用自动化方案来重新安装。

### 将 kubeadm 管理的集群迁移到 `systemd` 驱动

如果你希望将现有的由 kubeadm 管理的集群迁移到 `systemd` cgroup 驱动， 请按照[配置 cgroup 驱动](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/)操作。

## CRI 版本支持

你的容器运行时必须至少支持 v1alpha2 版本的容器运行时接口。

Kubernetes 1.26 默认使用 v1 版本的 CRI API。如果容器运行时不支持 v1 版本的 API， 则 kubelet 会回退到使用（已弃用的）v1alpha2 版本的 API。

## 容器运行时

>  **说明：** 本部分链接到提供 Kubernetes 所需功能的第三方项目。Kubernetes 项目作者不负责这些项目。此页面遵循[CNCF 网站指南](https://github.com/cncf/foundation/blob/master/website-guidelines.md)，按字母顺序列出项目。要将项目添加到此列表中，请在提交更改之前阅读[内容指南](https://kubernetes.io/docs/contribute/style/content-guide/#third-party-content)。

安装 containerd

```bash
dnf install containerd -y 

cat > /etc/systemd/system/containerd.service.d/http_proxy.conf << EOF
[Service]
Environment="HTTP_PROXY=http://192.168.101.77:8899"
Environment="HTTPS_PROXY=http://192.168.101.77:8899"
Environment="ALL_PROXY=socks5://192.168.101.77:8899"
Environment="NO_PROXY=127.0.0.1,localhost,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.liaosirui.com"
EOF

```

docker 添加代理
```bash
 mkdir -p /etc/systemd/system/docker.service.d
cat > /etc/systemd/system/docker.service.d/proxy.conf << EOF
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:8899"
Environment="HTTPS_PROXY=http://127.0.0.1:8899"
Environment="ALL_PROXY=socks5://127.0.0.1:8899"
Environment="NO_PROXY=127.0.0.1,localhost,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.liaosirui.com"
EOF

```







## 正式安装

1、 安装kubeadm kubectl kubelet;指定对应关系版本安装

```bash

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

dnf install kubelet-1.23.0-0 kubeadm-1.23.0-0 kubectl-1.23.0-0 --nogpgcheck -y

sudo systemctl enable --now kubelet
```





2、 安装crictl

所有节点执行

```bash
[root@master1 opt]# VERSION="v1.23.0"
[root@master1 opt]# curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-${VERSION}-linux-amd64.tar.gz --output crictl-${VERSION}-linux-amd64.tar.gz

[root@master1 opt]# crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock
```



3、直接安装集群

```bash
kubeadm init  --apiserver-advertise-address=192.168.101.77 --kubernetes-version=v1.23.0

```

4、保存集群信息

```tex
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.101.77:6443 --token 38zz8j.fu3ajbpyvs8ijv6m \
	--discovery-token-ca-cert-hash sha256:e946655813d627968e7abb2a7cf21a0c6ac1627fb8b5395093d944736e0c9915
```

5、安装网络插件

```bash
cat >> ./values.yml << EOF 
hubble:
  enabled: true
  relay:
    enabled: true
  ui:
    enabled: true
ipam:
  operator:
    clusterPoolIPv4PodCIDR: "10.4.0.0/16"
    clusterPoolIPv4MaskSize: 24
k8s:
  requireIPv4PodCIDR: true
hostPort:
  enabled: true
nodePort:
  enabled: true
kubeProxyReplacement: strict
k8sServiceHost: apiserver.local.liaosirui.com
k8sServicePort: 6443
loadBalancer:
  algorithm: maglev
EOF
```



```bash
helm repo add cilium https://helm.cilium.io/
helm upgrade --install cilium cilium/cilium \
  --version 1.12.4 \
  --namespace kube-system \
  -f ./values.yml
```



6、 安装 metrics-server

```yaml
cat >> ./values.yml << EOF
defaultArgs:
  - --cert-dir=/tmp
  - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
  - --kubelet-use-node-status-port
  - --metric-resolution=15s
  - --kubelet-insecure-tls
EOF

```



```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm upgrade --install metrics-server metrics-server/metrics-server -f ./values.yml -n kube-system
```



7、配置 krew



```bash
cd "$(mktemp -d)" &&
OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
KREW="krew-${OS}_${ARCH}" &&
curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
tar zxvf "${KREW}.tar.gz" &&
./"${KREW}" install krew

export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```



```bash
export KUBE_KREW_INSTALL_CMD=( "kubectl" "krew" "install" )

kubectl krew index add kvaps https://github.com/kvaps/krew-index
"${KUBE_KREW_INSTALL_CMD[@]}" kvaps/node-shell
"${KUBE_KREW_INSTALL_CMD[@]}" ice
"${KUBE_KREW_INSTALL_CMD[@]}" whoami
"${KUBE_KREW_INSTALL_CMD[@]}" ctx
"${KUBE_KREW_INSTALL_CMD[@]}" ns
"${KUBE_KREW_INSTALL_CMD[@]}" grep
"${KUBE_KREW_INSTALL_CMD[@]}" iexec
"${KUBE_KREW_INSTALL_CMD[@]}" doctor
"${KUBE_KREW_INSTALL_CMD[@]}" df-pv
"${KUBE_KREW_INSTALL_CMD[@]}" tail
"${KUBE_KREW_INSTALL_CMD[@]}" resource-capacity
"${KUBE_KREW_INSTALL_CMD[@]}" view-allocations
"${KUBE_KREW_INSTALL_CMD[@]}" ingress-nginx
"${KUBE_KREW_INSTALL_CMD[@]}" minio
```



8、 helm 安装 ingress

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update


cat >> ./values.yml << EOF
controller:
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
  nodeSelector:
    kubernetes.io/os: linux
    ingress: "true"
  kind: Deployment
  hostNetwork: true
EOF

kubectl create ns ingress-nginx

helm install ingress-nginx -n ingress-nginx -f ./values.yml
```



9、安装rancher

```bash
# 安装rancher 前准备
kubectl create namespace cattle-system
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
##  安装cert-manager

cat >> ./values.yml << EOF
cainjector:
  tolerations:
  - operator: Exists
installCRDs: true
prometheus:
  enabled: false
startupapicheck:
  tolerations:
  - operator: Exists
tolerations:
- operator: Exists
webhook:
  tolerations:
  - operator: Exists

EOF

helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.11.1 \
  -f ./values.yml


```

10、安装 rancher

```bash
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=cmzhu.cn \
  --set bootstrapPassword=admin
  
## 例子
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set bootstrapPassword=admin
```

11、 安装minio
```bash
helm repo add minio https://charts.min.io/
helm repo upgrade

helm \
upgrade --install \
--namespace minio \
--create-namespace \
--set rootUser=rootuser,rootPassword=rootpass123 \
--debug \
minio minio/minio \
-f ./values.yml
```
12、reloader 安装
```bash
helm repo add stakater https://stakater.github.io/stakater-charts
helm repo update
helm pull stakater/reloader --untar
helm \
upgrade --install \
reloader \
./reloader \
-n cmzhu
```

## 故障处理
1、 cri-o 不兼容问题
```bash
[root@cmzhu cmzhu]# crictl ps

FATA[0000] validate service connection: CRI v1 runtime API is not implemented for endpoint "unix:///run/containerd/containerd.sock": rpc error: code = Unimplemented desc = unknown service runtime.v1.RuntimeService
[root@cmzhu cmzhu]# containerd config default


```
 **解决办法**
 1、开启cgroup
 ```bash
dnf install containerd -y

containerd config default | tee /etc/containerd/config.toml

sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml

crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock --set image-endpoint=unix:///run/containerd/containerd.sock

systemctl restart containerd
 ```
 2、 修改containerd 配置文件

```bash 
vim /etc/containerd/conf.toml
enabled_plugins = ["cri"]
[plugins."io.containerd.grpc.v1.cri".containerd]
  endpoint = "unix:///var/run/containerd/containerd.sock"
```

 

