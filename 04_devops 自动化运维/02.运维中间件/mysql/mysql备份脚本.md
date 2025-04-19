---
title: mysql dump 备份脚本
description: 
published: true
date: 2023-04-12T07:42:40.844Z
tags: 中间件
editor: markdown
dateCreated: 2023-04-12T07:42:40.844Z
---

## mysql 定时备份脚本

```bash
#! /bin/bash

set -vx

HOST=${MYSQL_CLUSTER_HOST}
USER=${MYSQL_CLUSTER_USER}
PASSWORD=${MYSQL_CLUSTER_PASSWORD}
TMP_DIR=${TMP_DIR:-/root/}
BACKUP_DIR=${BACKUP_DIR}



DATE=`date "+%Y-%m-%d"`
cd $TMP_DIR
mkdir -p $DATE

mysqldump -h $HOST -u$USER -p$PASSWORD --single-transaction --skip-lock-tables --all-databases > ${BACKUP_DIR}/mysql_cluster_${DATE}.sql
```



备份表结构

```bash
$ mysqldump --opt -d bigai -uroot -p$MYSQL_ROOT_PASSWORD > bigai-struct.sql　
```

