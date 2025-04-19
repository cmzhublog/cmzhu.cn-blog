---
title: POC 部署
description: 
published: true
date: 2023-01-04T06:54:41.376Z
tags: poc
editor: markdown
dateCreated: 2022-11-25T01:55:38.346Z
---

# sealos ipvs 用法测试



## sealos部署k8s集群

| IP             | 端口  |
| -------------- | ----- |
| 192.168.101.77 | 11022 |



> 1、 部署前环境检查

```shell
#关闭防火墙和selinux
setenforce 0; sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/sysconfig/selinux; systemctl stop firewalld.service; systemctl disable firewalld.service; systemctl stop iptables; systemctl disable firewall
 
#关闭swap
swapoff -a ; sed -i '/swap/ s/^/#/' /etc/fstab
 
#配置内核参数
grep -q '^net.ipv4.ip_forward' /etc/sysctl.conf && sed -i '/net.ipv4.ip_forward/ s/[0-9]$/1/' /etc/sysctl.conf || echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
grep -q '^net.bridge.bridge-nf-call-iptables' /etc/sysctl.conf && sed -i '/net.bridge.bridge-nf-call-iptables/ s/[0-9]$/1/' /etc/sysctl.conf || echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
grep -q '^net.bridge.bridge-nf-call-ip6tables' /etc/sysctl.conf && sed -i '/nnet.bridge.bridge-nf-call-ip6tables/ s/[0-9]$/1/' /etc/sysctl.conf || echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
sysctl -p
 
#在配置好主机名后，添加hosts解析
primaryip=$(ip route show |grep default |awk '{print $5}' |xargs -i ip addr show {} |grep -w inet |awk '/^[0-9]+/; {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}')
echo "$primaryip $(hostname)" >> /etc/hosts
```



> 2、 开始部署

> 注意:
>
> 1、 部署时需要用到ssh的工具,在部署前请确保sealos 执行节点与其余所有节点均可密钥通信.
>
> 2、 部署时如果ssh 端口不是22 ,可以要求修改为22 也可以修改配置,来解决
>
> like:    sealos init --master 192.168.101.77:11022 --pkg-url=kube1.18.14.tar.gz --version=1.18.14 --podcidr=100.90.0.0/16

```bash
#sealos 部署paas平台
sealos init --master 192.168.101.77 --pkg-url=kube1.18.14.tar.gz --version=1.18.14 --podcidr=100.90.0.0/16
 
#查看pod状态
kubectl get pods -A
```



> 3、部署完成,调优kubelet 服务

```bash
echo """KUBELET_KUBEADM_ARGS=\"--allowed-unsafe-sysctls 'kernel.msg*,net.core.somaxconn' --cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2\" """ > /var/lib/kubelet/kubeadm-flags.env
systemctl restart kubelet.service
```


> 4、 删除 master 的污点
```bash
kubectl taint node xczq node-role.kubernetes.io/master-
```

## 验证 sealos ipvs 功能

1、 查看命令详细信息



```bash
$ sealos ipvs --help

sealos create or care local ipvs lb

Usage:
  sealos ipvs [flags]

Flags:
  -c, --clean                  clean Vip ipvs rule before join node, if Vip has no ipvs rule do nothing. (default true)
      --health-path string    health check path (default "/healthz")
      --health-schem string   health check schem (default "https")
  -h, --help                  help for ipvs
      --interval int32        health check interval, unit is sec. (default 5)
      --rs strings            virturl server like 192.168.0.2:6443
      --run-once              is run once mode
      --vs string             virturl server like 10.54.0.2:6443

Global Flags:
      --config string   config file (default is $HOME/.sealos/config.yaml)
      --info            logger ture for Info, false for Debug
```

 

2、 