---
title: postgres免密登录
description: 
published: true
date: 2023-01-16T02:45:06.511Z
tags: 中间件
editor: markdown
dateCreated: 2023-01-16T02:45:06.511Z
---

# postgres 免密登录

## 背景

1、 postgres 在使用时会要求输入密码登录数据库,这样会对脚本实现等造成困扰



## 解决方案

### 1、 环境变量

PGPASSWORD: PGPASSWORD 是postgresql 的系统环境变量,在客户端连接远程数据库时会优先使用此密码

eg:

```bash
# 使用 postgres/abc123! 登录 本地postgres 数据库;
PGPASSWORD='abc123!' psql -U postgres
```

>  注意: 使用环境变量运行,不会弹出密码提示框,但是从安全性考虑,这种方法并不推荐



### 2、 使用密码文件

在 /home/postgres/.pgpass 隐藏文件中,保存账户密码,这样也可避免登录时弹出输入密码提示.

#### 创建密码文件

```bash
vi /home/postgres/.pgpass 

格式:
hostname:port:database:username:password  

```



eg:

```tex
postgres:5432:postgres:postgres:abc123!
```



#### 设置文件权限

```bash
chmod 600 /home/postgres/.pgpass
```

>    备注：在/home/postgres 目录创建了密码文件 .pgpass 文件后，并正确配置连接信息，那么客户端连接数据时会优先使用 .pgass文件， 并使用匹配记录的密码，从而不跳出密码输入提示,这种方法比方法一更安全，所以推荐使用创建 .pgpass 文件方式。

#### 3、修改服务端pg_hba.conf 文件

修改认证文件 $PGDATA/pg_hba.conf, 添加以下行， 并 reload使配置立即生效。
host   ZY-BigData    postgresql     192.168.8.169/32      trust

示例:

```tx
# IPv4 local connections:

host    all                      all                     0.0.0.0/32                      trust
host    all                      all                    127.0.0.1/32                   trust
host    ZY-BigData      postgresql         192.168.8.169/32           trust
host    all                      all                     0.0.0.0/0                         md5
```

备注：修改服务端 pg_hba.conf 并 reload 后，不再弹出密码输入提示。