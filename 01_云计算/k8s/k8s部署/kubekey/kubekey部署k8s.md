---
title: kubekey 部署k8s
description: 
published: true
date: 2023-08-21T07:21:56.494Z
tags: k8s
editor: markdown
dateCreated: 2023-08-21T07:21:56.494Z
---

## kubekey 部署k8s 集群

> 官方网站: https://github.com/kubesphere/kubekey/blob/master/README_zh-CN.md

### 介绍

KubeKey是一个开源的轻量级工具，用于部署Kubernetes集群。它提供了一种灵活、快速、方便的方式来安装Kubernetes/K3s、Kubernetes/K3s和KubeSphere，以及相关的云原生附加组件。它也是扩展和升级集群的有效工具。

此外，KubeKey还支持定制离线包（artifact），方便用户在离线环境下快速部署集群。

KubeKey 通过了 CNCF kubernetes 一致性认证。

有三种情况可以使用 KubeKey。

仅安装 Kubernetes
用一个命令中安装 Kubernetes 和 KubeSphere
首先安装 Kubernetes，然后使用 ks-installer 在其上部署 KubeSphere
重要提示：Kubekey 将会帮您安装 Kubernetes，若已有 Kubernetes 集群请参考 在 Kubernetes 之上安装 KubeSphere。

### 优势

基于 Ansible 的安装程序具有大量软件依赖性，例如 Python。KubeKey 是使用 Go 语言开发的，可以消除在各种环境中出现的问题，从而提高安装成功率。
KubeKey 使用 Kubeadm 在节点上尽可能多地并行安装 K8s 集群，以降低安装复杂性并提高效率。与较早的安装程序相比，它将大大节省安装时间。
KubeKey 支持将集群从 all-in-one 扩展到多节点集群甚至 HA 集群。
KubeKey 旨在将集群当作一个对象操作，即 CaaO。

### 支持版本

操作系统

- Ubuntu 16.04, 18.04, 20.04, 22.04
- Debian Bullseye, Buster, Stretch
- CentOS/RHEL 7
- AlmaLinux 9.0
- SUSE Linux Enterprise Server 15

k8s 版本支持

> https://github.com/kubesphere/kubekey/blob/master/docs/kubernetes-versions.md

### 部署

1、 创建集群部署配置文件(这里只创建 k8s 集群)

```bash
kk create config --with-kubernetes v1.26.2 -f ~/.kubekey/config.conf
```

此配置表示安装一个单节点的服务, 网络插件指定 cilium, 版本选择 1.26.2

```yaml
> cat ~/.kubekey/config.conf

apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: cmzhu, address: 10.100.100.77, internalAddress: 10.100.100.77, user: root, password: "f"}
  roleGroups:
    etcd:
    - cmzhu
    control-plane:
    - cmzhu
    worker:
  controlPlaneEndpoint:
    ## Internal loadbalancer for apiservers
    # internalLoadbalancer: haproxy

    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.26.2
    clusterName: cluster.local
    autoRenewCerts: true
    containerManager: containerd
  etcd:
    type: kubekey
  network:
    plugin: cilium
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
    ## multus support. https://github.com/k8snetworkplumbingwg/multus-cni
    multusCNI:
      enabled: false
  registry:
    privateRegistry: ""
    namespaceOverride: ""
    registryMirrors: []
    insecureRegistries: []
  addons: []
```

运行安装命令

```bash
kk create cluster -f ~/.kubekey/config.conf

```