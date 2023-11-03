---
title: Linux 设置代理
description: 
published: true
date: 2023-01-04T06:48:59.180Z
tags: linux
editor: markdown
dateCreated: 2023-01-04T06:48:59.180Z
---

# linux 配置代理

## 原理

linux 的代理配置是对系统环境变量进行设置

- HTTP_PROXY     http代理
- HTTPS_PROXY     https 代理
- NO_PROXY        # 禁止使用代理的地址





配置代理的脚本,并将脚本写入~/.bashrc 中

```bash
# where need proxy
proxy() {
    export PROXY_IP=192.168.101.77
    export PROXY_PORT=8899
    export NO_PROXY=127.0.0.1,localhost,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,apiserver.cluster.local
    export https_proxy=http://${PROXY_IP}:${PROXY_PORT} http_proxy=http://${PROXY_IP}:${PROXY_PORT} all_proxy=socks5://${PROXY_IP}:${PROXY_PORT}
    export HTTPS_PROXY="${https_proxy}" HTTP_PROXY="${http_proxy}" ALL_PROXY="${all_proxy}"
    echo "System http, https, socks5 Proxy on, proxy ip: ${PROXY_IP}, proxy port: ${PROXY_PORT}"
}

# where need noproxy
noproxy() {
    echo "System http, https, socks5 Proxy off, proxy ip: ${PROXY_IP}, proxy port: ${PROXY_PORT}"
    unset https_proxy http_proxy all_proxy HTTPS_PROXY HTTP_PROXY ALL_PROXY
}
```



执行指令开启代理

```bash
proxy
```



执行指令关闭代理

```bash
noproxy
```

