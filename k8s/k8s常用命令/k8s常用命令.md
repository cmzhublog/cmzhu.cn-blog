---
title: k8s 常用命令
description: 
published: true
date: 2023-04-10T02:11:45.382Z
tags: k8s
editor: markdown
dateCreated: 2022-12-19T06:43:35.804Z
---

# k8s 常用命令

## k8s 常用命令

- 获取命名空间下的所有在使用镜像

```bash
kubectl get pod -n cmzhu -o jsonpath='{.items[*].spec.containers[*].image}'
```
- kubectl 设置自动补全
```bash
  # 安装bash-completion
  yum install bash-completion -y
  
  # 命令写入到bashrc 中
  echo 'source <(kubectl completion bash)' >>~/.bashrc

```

- k8s 创建role and rolebinding;

role.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cronjob-deploy
  namespace: csc-alpha-digital-lab
rules:
- apiGroups:
  - "apps"
  resources:
  - jobs
  - cronjobs
  - daemonsets
  - deployments
  - horizontalpodautoscalers
  - ingresses
  - endpoints
  - configmaps
  - events
  - persistentvolumeclaims
  - pods
  - podtemplates
  - pods/log
  - secrets
  - services
  - replicationcontrollers
  - persistentvolumes
  verbs:
  - "*"
```

rolebinding.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cronjob-deploy
  namespace: csc-alpha-digital-lab
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: <role-name>
subjects:
- kind: ServiceAccount
  name: <sa-name>
  namespace: <namespace>
```

clusterrolebinding.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: jt-default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name:  <sa-name>
  namespace: <sa-namespace>
```


### krew 安装和使用


安装 git
```bash
yum install -y git
```

```bash
cd "$(mktemp -d)" 
OS="$(uname | tr '[:upper:]' '[:lower:]')" 
ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" 
KREW="krew-${OS}_${ARCH}" 
curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" 
tar zxvf "${KREW}.tar.gz" 
./"${KREW}" install krew
```



- krew 常用命令

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
"${KUBE_KREW_INSTALL_CMD[@]}" greps
"${KUBE_KREW_INSTALL_CMD[@]}" grep

```

- k8s 查看所有节点的资源分配

```sh

```

- 获取镜像使用多少

```bash
kubectl get deployments -n userbox -o json | jq '.items[] | {name: .metadata.name, image: .spec.template.spec.containers[].image}'|grep -B1 "master_0086441_230210140701"
```
- 获取所有Pod 常用的镜像
```bash
k get pod -o jsonpath='{.items[*].spec.containers[*].ima -Age}' | sed 's/ /\n/g'
```
- 污点和容忍度
[污点和容忍度](/k8s/k8s常用命令/污点和容忍度)

