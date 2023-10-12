---
title: qemu
description: 
published: true
date: 2022-12-05T10:42:18.128Z
tags: linux
editor: markdown
dateCreated: 2022-12-05T10:42:18.128Z
---

# linux qemu

## 安装

```bash
apt install qemu
```





1、 查看 是否支持CPU 虚拟化, 如果获得的值大于1 表示支持cpu 虚拟化, 其余内容表示不支持

```bash
egrep -c 'vmx|svm' /proc/cpuinfo
```

2、 

安装指令

Arch64 架构

```bash
sudo apt-get install libvirt0 libvirt-daemon qemu virt-manager bridge-utils libvirt-clients python-libvirt qemu-efi uml-utilities virtinst qemu-system
```

amd64 架构

```bash
apt-get install virt-manager bridge-utils libvirt-clients qemu qemu-kvm
```

