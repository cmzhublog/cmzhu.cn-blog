---
title: rocky9 系统安装
description: 
published: true
date: 2023-01-04T01:54:42.908Z
tags: linux
editor: markdown
dateCreated: 2023-01-04T01:54:42.908Z
---

# rocky linux 9 

## 升级内容

### 操作系统

> 对比 CentOS 7.9 minimal 的重大变更项

- 内核 5.14.0-162.6.1.el9_1.x86_64
- cgroup v1 → cgroup2
- 网络转发过滤默认使用 nft(nftables)，替换 centos 的 iptables

### paas 平台相关

- sealos 升级 v3.3.9 → 3.3.9-rc.11（支持docker systemd 配置）

- docker 19.03.12 → 20.10.9 （支持 cgroups v2）
- calico v3.8.2 → v3.12.0 （支持 nftables）

## Master 节点替换流程

### 备份主机配置

备份节点信息

```
kubectl get nodes dev-master1 -oyaml > dev-master1.yaml
```

备份 ssh 密钥对

```
scp -r 10.24.2.14:/root/.ssh /root/hhma/ssh-bak
```

### 移除节点

登录 bqdev01 执行 sealos clean 命令

```
# 移除重装的节点 sealos clean --master 10.24.2.14
```

### 重装操作系统

重装系统时需要主要三点：无 swap 分区、删除 /home 分区，可用的容量都分配给 / 分区、主机名与 IP 与重装前保持一致。

### 恢复主机配置

登录 master 节点

```
scp -r 10.24.2.1:/root/hhma/ssh-bak/.ssh /root/
```

### 安装必要的安装包

```
dnf -y install tar ipvsadm
```

### 加入集群

手动更新 .sealos/config.yaml，将 passwd 对应的值设置为被加入集群节点的密码。

**注意：**没有出现以下错误可跳过此步

```
Error create ssh session failed,ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain
```

更新 kube1.19.16.tar.gz以 10.24.2.1 环境的文件为准。变更内容如下：

kube1.19.16.tar.gz：

- 升级 docker 到 20.10.9。19.03.12 不支持 cgroups v2，无法兼容 Rockylinux 9
- docker 和 kubelet Cgroup Driver 使用 systemd

添加节点

```
sealos join --master 10.24.2.14
```

### 恢复标签

```
kubectl label nodes dev-master1 ingress-nginx=true
```



### 配置 OIDC

拷贝 oidc 证书，登录其他 master 节点。把 /etc/kubernetes/pki/cmzhu-console-ca.crt 拷贝到本机相同的路径。

更新 /etc/kubernetes/manifests/kube-apiserver.yaml，添加 oidc 相关的配置。

```
    - --oidc-issuer-url=https://console.local.dev.cmzhu.cn/ai/users    - --oidc-client-id=k8s    - --oidc-username-claim=name    - --oidc-username-prefix=-    - --oidc-groups-claim=groups    - --oidc-ca-file=/etc/kubernetes/pki/cmzhu-console-ca.crt
```

## Node 节点替换流程

### 安装必要的安装包

```
dnf -y install tar ipvsadm socat dnf groupinstall -y "Development Tools"
```

### 添加节点

```
sealos join --node 10.24.105.244
```

### 恢复标签

```
kubectl label nodes <dev-whliao> node-role.kubernetes.io/kbdev= node-role.kubernetes.io/worker=
```



### 开发工具包安装

```bash
$ dnf install -y telnet traceroute nmap-ncat bind-utils net-tools iproute 
$ dnf install -y \
git git-lfs \
sudo \
passwd shadow-utils \
jq \
\
&& dnf install -y \
ca-certificates \
gnupg gnupg2 \
vim \
ccze \
wget curl rsync \
tree \
lsof \
tmux screen \
zip unzip \
unrar \
procps-ng \
man-pages man-db \
bash-completion \
which 

$ dnf install -y llvm llvm-libs clang libomp compiler-rt dnf --enablerepo=devel install -y \
mysql-devel \
libaec-devel \
hdf5-devel \
ncurses ncurses-devel \
bzip2 bzip2-devel \
openssl openssl-devel \
boost-devel \
sqlite-devel \
gdbm-devel \
readline-devel \
zlib-devel \
libffi-devel 

$ dnf install -y \
glibc \
libstdc++ \
libstdc++.so.6 \
libstdc++-devel \
glibc-all-langpacks \
kernel-doc \
kernel-headers \
kernel-devel \
kernel-debug-devel \
kernel-tools \
kernel-tools-libs 
$ dnf install -y langpacks-en langpacks-zh_CN dnf groupinstall -y Fonts
```

### 调整Kubelet

```
echo """KUBELET_KUBEADM_ARGS=\"--allowed-unsafe-sysctls 'kernel.msg*,net.core.somaxconn' --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2\" """ > /var/lib/kubelet/kubeadm-flags.env  systemctl restart kubelet.service
```



## 问题集锦

### Deployment 映射的 HostPort 未正常工作

**原因：**NetworkManger 后端支持 nftable，默认未启用 iptables

**解决：**升级 calico 到 v3.12.0。从 v3.12.0 开始，calico-node 自动检测 nftable。

###  kbdev-0 启动时解析 code.cmzhu.ai 失败

**原因：**待排查

**解决**：重启主机

## 参考：

- https://forums.rockylinux.org/t/iptable-kernel-modules-in-8-5/5297
- https://www.tigera.io/blog/whats-new-in-calico-v3-12/