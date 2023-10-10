---
title: kubebuilder 安装
description: 
published: true
date: 2022-12-03T03:29:45.968Z
tags: k8s
editor: markdown
dateCreated: 2022-12-03T03:16:57.129Z
---

# kubebuilder 安装

# kubebuild 快速入门

## 先决条件

- [go](https://golang.org/dl/)版本v1.19.0+
- [docker](https://docs.docker.com/install/)版本 17.03+。
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)版本 v1.11.3+。
- 访问 Kubernetes v1.11.3+ 集群。

## 安装

下载kubebuilder 二进制

```bash
# download kubebuilder and install locally.
curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)
chmod +x kubebuilder && mv kubebuilder /usr/local/bin/
```









