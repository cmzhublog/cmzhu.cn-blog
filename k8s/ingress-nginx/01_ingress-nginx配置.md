---
title: 01_ingress-nginx 配置
description: 
published: true
date: 2023-07-12T06:13:34.431Z
tags: k8s
editor: markdown
dateCreated: 2023-05-11T07:15:48.684Z
---

## `ingress-nginx` 部署

`Helm` 部署`ingress`

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update


cat >> ./values.yml << EOF
controller:
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
dhParam:
  nodeSelector:
    kubernetes.io/os: linux
    ingress: "true"
  kind: DaemonSet
EOF

kubectl create ns ingress-nginx

helm install ingress-nginx -n ingress-nginx -f ./values.yml
```



## `ingress` 配置

1、 将http 请求重定向为https 请求

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: 'true'
    nginx.ingress.kubernetes.io/force-ssl-redirect: 'true'
    
...
```

## 问题记录
1、 http 重定向成https配置
```bash
annactions:
  # 设置后不会删除uri最后的/, 直接原样输出
  nginx.ingress.kubernetes.io/preserve-trailing-slash: "true"
  # 设置成http 请求 自动重定向成https 请求
  nginx.ingress.kubernetes.io/ssl-redirect: 'true'
  nginx.ingress.kubernetes.io/force-ssl-redirect: 'true'

```


> 如果这样配置仍然不生效? 清检查`ingress-nginx` 配置 的 `use-proxy-protocol` 是否为`false`, 如果不是,请配置它
>
> 配置参考官方文档:
>
> https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/

