---
title: postgres 主页
description: 
published: true
date: 2023-04-07T02:30:37.378Z
tags: postgres
editor: markdown
dateCreated: 2022-11-29T08:37:26.236Z
---

# Postgres 主页

## 安装

1、 docker-compose 安装 
postgres docker-compose 搭建,其中 pgadmin4是属于pg 的前端界面操作. 
docker-compose.yaml
```bash
      # Use postgres/example user/password credentials 
    # https://hub.docker.com/_/postgres?tab=description
    # 在当前目录下运行：sudo docker-compose up -d
    # 若需停止运行，在当前目录运行：sudo docker-compose down
    # docker路由地址查看： sudo docker inspect postgres_baicai
    # sudo docker kill $(sudo docker ps -aq)
    # sudo docker rm $(sudo docker ps -aq)
    version: '3.1'

    services:

    db:
        image: postgres
        restart: always
        privileged: true
        container_name: postgres_baicai
        ports:
        - 5432:5432
        environment:
        POSTGRES_PASSWORD: 你的密码
        PGDATA: /var/lib/postgresql/data/pgdata
        volumes:
        - /navxin/kn1/baicai_docker/baicai_postgres/pg_data:/var/lib/postgresql/data
        # - pgdata:/var/lib/postgresql/data

    pgadmin4:
        image: dpage/pgadmin4
        restart: always
        container_name: pgadmin_baicai
        ports:
        - 5080:80
        environment:
        PGADMIN_DEFAULT_EMAIL: "hi@nav.xin"
        PGADMIN_DEFAULT_PASSWORD: 你的密码
```

2、 k8s 安装

```yaml
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: postgres
  namespace: cmzhu
spec:
  replicas: 1
  selector:
    matchLabels:
      access_type: protected
      app: postgres
      role: system
  template:
    metadata:
      creationTimestamp: null
      labels:
        access_type: protected
        app: postgres
        role: system
    spec:
      volumes:
        - name: postgres-storage
          hostPath:
            path: /data/postgres_data
            type: DirectoryOrCreate
      containers:
        - resources: {}
          terminationMessagePath: /dev/termination-log
          name: postgres
          env:
            - name: POSTGRES_USER
              value: aiadm
            - name: POSTGRES_PASSWORD
              value: 4ZKSZL7tqpEwefzYiy1A8IN4k
          ports:
            - name: postgres
              containerPort: 5432
              protocol: TCP
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
          terminationMessagePolicy: File
          image: 'postgres:13.4'
      restartPolicy: Always

```

## 问题
- [修改postgres最大连接数](/中间件/postgres/修改postgres最大连接数)
- [postgres迁移数据库](/中间件/postgres/postgres迁移数据库)
- [postgres免密登录](/中间件/postgres/postgres免密登录)
- [postgres删除数据库](/中间件/postgres/postgres删除数据库)
- [postgres数据库数据增量同步](/中间件/postgres/postgres数据库数据增量同步)

{.links-list}
