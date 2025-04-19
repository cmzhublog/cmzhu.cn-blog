## Argo 简介

- 参考链接: [https://argo-cd.readthedocs.io/en/stable/](https://argo-cd.readthedocs.io/en/stable/)



1、 Argo CD 是`声明式（declarative）`的针对 k8s 的持续交付工具。

![Argo CD UI](01_Argo CD get start.assets/argocd-ui.gif)

### 1 部署

#### 1.1 快速部署

使用k8s 的yaml 文件直接进行部署，部署方式如下

```bash
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

