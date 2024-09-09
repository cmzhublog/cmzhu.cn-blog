---
title: harbor 部署
description: 
published: true
date: 2022-11-03T09:46:16.129Z
tags: docker
editor: markdown
dateCreated: 2022-11-03T09:37:18.368Z
---

# 通过 Helm 搭建 Docker 镜像仓库 Harbor

**参数地址：**

- [在 Kubernetes 在快速安装 Harbor](https://www.qikqiak.com/post/harbor-quick-install/)
- [Harbor-Helm 的 Github](https://github.com/goharbor/harbor-helm)

helm 安装

### 创建harbor 的namespace

由于 Harbor 组件较多，一般我们会采取新建一个 Namespace 专用于部署 Harbor 相关组件，输入下面命令创建名为 `mydlq-hub` 的命名空间。

```bash
kubectl create namespace mydlq-hub
```



### 挂载存储

```
#挂载 NFS
$ mount -o vers=4.1 192.168.2.11:/nfs/ /nfs

#创建文件夹
mkdir -p /nfs/harbor/registry
mkdir -p /nfs/harbor/chartmuseum
mkdir -p /nfs/harbor/jobservice
mkdir -p /nfs/harbor/database
mkdir -p /nfs/harbor/redis
mkdir -p /nfs/harbor/trivy
```



创建PV 和 PVC 

harbor-pv.yaml

```yaml
#registry-PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-registry
  labels:
    app: harbor-registry
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "hub"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/registry
    server: 192.168.2.11
---
#harbor-chartmuseum-pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-chartmuseum
  labels:
    app: harbor-chartmuseum
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "hub"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/chartmuseum
    server: 192.168.2.11
---
#harbor-jobservice-pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-jobservice
  labels:
    app: harbor-jobservice
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "hub"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/jobservice
    server: 192.168.2.11
---
#harbor-database-pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-database
  labels:
    app: harbor-database
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "hub"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/database
    server: 192.168.2.11
---
#harbor-redis-pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-redis
  labels:
    app: harbor-redis
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "hub"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/redis
    server: 192.168.2.11
---
#harbor-trivy-pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-trivy
  labels:
    app: harbor-trivy
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "hub"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/trivy
    server: 192.168.2.11
```



创建pv

```shell
kubectl apply -f harbor-pv.yaml
```

**创建 PVC 部署文件 harbor-pvc.yaml**



```yaml
#harbor-registry-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-registry
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 100Gi
  selector:
    matchLabels:
      app: harbor-registry
---
#harbor-chartmuseum-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-chartmuseum
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-chartmuseum
---
#harbor-jobservice-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-jobservice
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-jobservice 
---
#harbor-database-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-database
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-database  
---
#harbor-redis-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-redis
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-redis
---
#harbor-trivy-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-trivy
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-trivy
```

创建PVC

```shell
kubectl apply -f harbor-pvc.yaml -n mydlq-hub
```

