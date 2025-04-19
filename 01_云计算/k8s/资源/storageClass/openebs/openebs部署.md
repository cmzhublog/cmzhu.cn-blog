---
title: openebs 部署
description: 
published: true
date: 2023-04-10T03:52:43.567Z
tags: k8s
editor: markdown
dateCreated: 2023-04-10T03:52:43.567Z
---

# openebs

## openebs 安装

使用helm 安装

1、 添加helm repo

```bash
[root@cmzhu cmzhu]# helm repo add openebs https://openebs.github.io/charts
```

2、 查看helm repo

```bash
[root@cmzhu cmzhu]# helm repo list
```

```bash
[root@cmzhu cmzhu]# helm repo list
NAME                	URL
github              	https://kubernetes.github.io/ingress-nginx
bitnami             	https://charts.bitnami.com/bitnami
prometheus-community	https://prometheus-community.github.io/helm-charts
metrics-server      	https://kubernetes-sigs.github.io/metrics-server/
cilium              	https://helm.cilium.io/
openfunction        	https://openfunction.github.io/charts/
strimzi             	https://strimzi.io/charts/
rancher-alpha       	https://releases.rancher.com/server-charts/alpha
jetstack            	https://charts.jetstack.io
rancher-latest      	https://releases.rancher.com/server-charts/latest
rancher-stable      	https://releases.rancher.com/server-charts/stable
ingress-nginx       	https://kubernetes.github.io/ingress-nginx
openebs             	https://openebs.github.io/charts
```

3、 更新

```bash
[root@cmzhu cmzhu]# helm repo upgrade
```

4、 拉取`values.yml`

```bash
[root@cmzhu cmzhu]# helm show values openebs/openebs -n kube-system > ./values.yml
```

5、 修改values.yml 地址

```bash
localprovisioner:
  basePath: "/data/openebs/local"  ## 修改local path 底层存储地址
```

6、 部署

```bash
[root@cmzhu cmzhu]# helm upgrade --install openebs  openebs/openebs -n kube-system -f ./values.yml
```

7、 查看

```bash
[root@cmzhu openebs]# helm list -n kube-system
NAME          	NAMESPACE  	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSION
openebs       	kube-system	1       	2023-04-10 10:51:38.078873251 +0800 CST	deployed	openebs-3.5.0       	3.5.0
```

```bash
[root@cmzhu openebs]# k get sc
NAME               PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device     openebs.io/local   Delete          WaitForFirstConsumer   false                  59m
openebs-hostpath   openebs.io/local   Delete          WaitForFirstConsumer   false                  59m
```

