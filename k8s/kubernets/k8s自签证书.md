---
title: k8s 自签证书
description: 
published: true
date: 2023-01-05T04:41:55.018Z
tags: k8s
editor: markdown
dateCreated: 2022-12-08T09:19:10.590Z
---

 

# k8s 自签证书 

1、 k8s 自签证书,并创建secret

```yaml
# 生成证书  https准备
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BJ/O=nginx/CN=itheima.com"
​
# 创建密钥
kubectl create secret tls tls-secret --key tls.key --cert tls.crt
 
 
 
 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-https
  namespace: dev
spec:
  tls:
    - hosts:
      - nginx.will.com
      secretName: tls-secret # 指定秘钥，如果指定了秘钥就是https，未指定就是http。
  rules:
  - host: nginx.will.com
    http:
      paths:
      - path: /                          # 路径信息
        backend:
          serviceName: nginx-service     # 指定service的名称
          servicePort: 80                # 指定端口信息
```


2、 openssl 1.1.1x 创建证书

```bash
#!/bin/bash
#generate ssl for gitlab domain and create secret

domainName=ssl-name
consoleNS=console-NS

# 支持 addext 参数需要将 OpenSSL 更新到 1.1.1d
openssl req -x509 -nodes -days 700 -newkey rsa:2048 -keyout ${consoleNS}-tls.key -out ${consoleNS}-ca.crt -subj "/CN=${domainName}" -addext "subjectAltName = DNS:${domainName}"

kubectl -n $consoleNS  create secret tls  sso-x509-https-secret  --key ${consoleNS}-tls.key --cert ${consoleNS}-ca.crt --dry-run -o yaml |kubectl apply  -f -


```
