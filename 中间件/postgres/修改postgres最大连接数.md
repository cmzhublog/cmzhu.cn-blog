---
title: postgres最大链接数
description: 
published: true
date: 2023-01-10T08:40:16.265Z
tags: postgres
editor: markdown
dateCreated: 2022-11-29T08:53:17.501Z
---

# postgres 最大连接数

postgres 默认的 max_connections=100; 但是在实际的使用过程中,postgres 可能会超过这个值;这时就需要修改这个配置

修改方案1(临时修改- 容器重启,自动恢复为默认值)
```bash
alter system set max_connections=2000
```
修改方案2 (长期修改- 通过修改配置文件来修改)
```bash

cd /var/lib/postgresql/data
vim /var/lib/postgresql/data/postgresql.conf

修改max_connections 的值为需要的值
```

## 查询postgres 连接socket
使用以下sql 可以查询连接,

```bash
SELECT * from pg_stat_activity ;
```
当然,对sql 计数就可以获得连接的个数

```bash
SELECT count(*) from pg_stat_activity ;
```
