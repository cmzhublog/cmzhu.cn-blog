---
title: HAproxy负载均衡
description: 
published: true
date: 2022-12-03T07:10:24.335Z
tags: k8s
editor: markdown
dateCreated: 2022-12-02T07:29:24.248Z
---

# Hyproxy 学习

1、 官方文档

HAProxy 官方文档: http://docs.haproxy.org/2.7/configuration.html#2.1
HAProxy github 仓库: https://github.com/haproxy/haproxy



## 配置方式

haproxy.cfg

```nginx
# This is a sample configuration. It illustrates how to separate static objects
# traffic from dynamic traffic, and how to dynamically regulate the server load.
#
# It listens on 192.168.1.10:80, and directs all requests for Host 'img' or
# URIs starting with /img or /css to a dedicated group of servers. URIs
# starting with /admin/stats deliver the stats page.
#

global
        maxconn         10000
        stats socket    /var/lib/haproxy/haproxy.stat mode 600 level admin

        log             127.0.0.1 local0
        uid             0
        gid             0
        daemon

# The public 'www' address in the DMZ
frontend public
        bind            0.0.0.0:80 name clear
        #bind            192.168.1.10:443 ssl crt /etc/haproxy/haproxy.pem
        mode            http
        log             global
        option          httplog
        option          dontlognull
        monitor-uri     /monitoruri
        maxconn         8000
        timeout client  30s

        stats uri       /admin/stats
        use_backend     static if { hdr_beg(host) -i img }
        use_backend     static if { path_beg /img /css   }
        default_backend dynamic

# The static backend backend for 'Host: img', /img and /css.
backend static
        mode            http
        balance         roundrobin
        option prefer-last-server
        retries         2
        option redispatch
        timeout connect 5s
        timeout server  5s
        option httpchk  HEAD /favicon.ico
        server          statsrv1 192.168.1.8:80 check inter 1000
        server          statsrv2 192.168.1.9:80 check inter 1000

# the application servers go here
backend dynamic
        mode            http
        balance         roundrobin
        retries         2
        option redispatch
        timeout connect 5s
        timeout server  30s
        timeout queue   30s
        option httpchk  HEAD /login.php
        cookie          DYNSRV insert indirect nocache
        fullconn        4000 # the servers will be used at full load above this number of connections
        server          dynsrv1 10.24.2.14:80 minconn 50 maxconn 500 cookie s1 check inter 1000
        server          dynsrv2 10.24.2.15:80 minconn 50 maxconn 500 cookie s2 check inter 1000
        server          dynsrv3 10.24.2.16:80 minconn 50 maxconn 500 cookie s3 check inter 1000
```



haproxy 启动方案

restart_haproxy.sh

```bash
#!/bin/bash
docker rm -f my-running-haproxy
docker run -d --name my-running-haproxy --privileged --sysctl net.ipv4.ip_unprivileged_port_start=0 -v ${PWD}/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg -p 80:80  haproxy
```




### 使用TCP 方式

haproxy.cfg
```nginx
# Global settings
#---------------------------------------------------------------------
global
    maxconn     20000
    # log         /dev/log local0 info
    log         127.0.0.1 local0 debug
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/haproxy.stat mode 600 level admin
#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
#    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s

    timeout tunnel 1h
    timeout client-fin 30s
    timeout queue           1m
    timeout connect         10s
    timeout client          300s
    timeout server          300s
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 200000

listen stats
    bind :9000
    mode http
    stats enable
    stats uri /

#    server      master2 192.168.145.219:8443 check

frontend route-80
    bind *:80
    acl denylist src 171.83.9.9 113.108.77.24 139.224.229.201
    tcp-request connection reject if denylist
    redirect scheme https if !{ ssl_fc }
    capture request header Host len 20
    capture request header Referer len 60
    default_backend route-80
    mode http
    option tcplog
    option httplog

backend route-80
    balance source
    mode http
    option forwardfor
    server      master5 10.24.2.14:80  send-proxy  check inter 3000 fall 3 rise 5
    server      master4 10.24.2.15:80  send-proxy  check inter 3000 fall 3 rise 5
    server      master6 10.24.2.16:80  send-proxy  check inter 3000 fall 3 rise 5

frontend route-443
    bind *:443
    default_backend route-443
    mode tcp
    option tcplog


backend route-443
    balance source
    option forwardfor
    mode tcp
    server      master5 10.24.2.14:443  send-proxy  check inter 3000 fall 3 rise 5
    server      master4 10.24.2.15:443  send-proxy  check inter 3000 fall 3 rise 5
    server      master6 10.24.2.16:443  send-proxy  check inter 3000 fall 3 rise 5

```

restart.sh
```shell
#!/bin/bash
docker rm -f my-running-haproxy
docker run -d --name my-running-haproxy --privileged --sysctl net.ipv4.ip_unprivileged_port_start=0 -v ${PWD}/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg -p 80:80 -p 443:443  haproxy

```
## keepalived 



