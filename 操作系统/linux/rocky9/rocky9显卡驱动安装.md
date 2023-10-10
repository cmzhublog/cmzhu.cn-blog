---
title: rocky9 显卡驱动安装
description: 
published: true
date: 2023-02-23T02:28:09.097Z
tags: linux
editor: markdown
dateCreated: 2023-02-23T02:28:09.097Z
---

# NVIDIA 显卡驱动安装

背景: rocky9 安装1050Ti 显卡驱动

```bash
# 安装源
dnf -y install epel-release
# 安装依赖
dnf -y install gcc kernel-devel dkms libglvnd-devel.x86_64

# 去官网下载驱动 (https://www.nvidia.cn/Download/index.aspx?lang=cn)
wget https://cn.download.nvidia.com/XFree86/Linux-x86_64/525.89.02/NVIDIA-Linux-x86_64-525.89.02.run

# 添加执行权限
 chmod u+x NVIDIA-Linux-x86_64-525.89.02.run
 
 ./NVIDIA-Linux-x86_64-525.89.02.run

```

 

