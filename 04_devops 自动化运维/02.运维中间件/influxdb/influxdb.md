---
title: influxdb
description: 
published: true
date: 2023-06-25T06:55:19.708Z
tags: influxdb
editor: markdown
dateCreated: 2023-06-25T06:55:19.708Z
---

## influxdb

### 背景

`influxdb` 是用于分析和存储时间序列数据的数据库
主要具有:

- 内置`http` 请求接口, 使用方便
- 数据可以打标记, 查询灵活
- 类`sql` 查询语句用法
- 安装管理简单, 数据读取很高效
- 能实时查询, 数据在写入时被索引后能立即查出

### 安装

#### 安装前准备

- 安装 `influxdb` 需要使用`root` 或者具有管理员权限的账户
- 需要保证`8086` `8088` 端口未被占用

#### 网络需求

`InfluxDB`默认使用下面的网络端口：

- `TCP`端口`8086`用作`InfluxDB`的客户端和服务端的`http api`通信
- `TCP`端口`8088`给备份和恢复数据的`RPC`服务使用
另外，`InfluxDB`也提供了多个可能需要自定义端口的插件，所以的端口映射都可以通过配置文件修改，对于默认安装的`InfluxDB`，这个配置文件位于`/etc/influxdb/influxdb.conf`

#### NTP

`influxdb` 使用的本地时间给数据加时间戳, 并且是使用 `UTC` 时区的, 使用`NTP` 来同步本地服务器之间的时间, 如果服务器没有配置`NTP`, 可能会导致写入`influxdb` 的数据时间戳不准确

#### RedHat & Centos

`RedHat`和`CentOS`用户可以直接用`yum`包管理来安装最新版本的`InfluxDB`

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```

添加了`yum`源之后, 可以通过以下命令来安装和启动 `influxdb` 服务

```bash
## Rocky linux 9
dnf install influxdb
systemctl start influxdb && systemctl enable influxdb

## Centos 7
yum install influxdb
systemctl start influxdb && systemctl enable influxdb
```

#### 配置

安装完成之后, 每个配置文件都有默认的配置, 可以通过命令 `influxd config` 来查看这些默认配置

其中 在配置文件 `/etc/influxdb/influxdb.conf` 的大部分配置都是被注释过的, 使用的系统自带的默认配置;但可以通过修改该配置文件, 来覆盖哪些系统默认配置

有下面两种方式来通过自定义配置文件来运行 `influxdb`

- 运行可通过 `-config` 来指定配置文件

```bash
influxd -config /etc/influxdb/influxdb.conf
```

- 设置环境变量来指定 `INFLUXDB_CONFIG_PATH`

```bash

echo $INFLUXDB_CONFIG_PATH
/etc/influxdb/influxdb.conf


influxd

```

### influxdb 使用

#### 写入数据

写入`influxdb` 数据的方式有很多种, 命令行, 客户端, 以及一些`Graphite` 类型的插件, 都可以实现数据写入;

- 使用 http 接口创建数据库

```bash
### 使用 POST 的方式发送到 URL 的 /query 接口, 参数为 q=CREATE DATABASE <database>

#### 创建 名为cmzhu 的数据库
curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE cmzhu"
```

- 使用 `http` 接口写入数据

```bash
### 通过 HTTP 接口 POST 数据到 /write 路径是我们往 InfluxDB 写数据的主要方式。
### 下面的例子写了一条数据到 cmzhu 数据库。
### 这条数据的组成部分是 measurement 为 cpu_load_short ，tag的key为host和region，对应tag的value是server01和us-west，
### field的key是value，对应的数值为0.64，而时间戳是1434055562000000000。


curl -i -XPOST 'http://localhost:8086/write?db=cmzhu' --data-binary 'cpu_load_short,host=server01,region=us-west value=0.64 1434055562000000000'

```

POST的请求体我们称之为`Line Protocol`，包含了你希望存储的时间序列数据。它的组成部分有`measurement`，`tags`，`fields`和`timestamp`。`measurement`是`InfluxDB`必须的，严格地说，`tags`是可选的，但是对于大部分数据都会包含`tags`用来区分数据的来源，让查询变得容易和高效。`tag`的`key`和`value`都必须是字符串。`fields`的`key`也是必须的，而且是字符串，默认情况下`field`的`value`是`float`类型的。`timestamp`在这个请求行的最后，是一个从`1/1/1970 UTC`开始到现在的一个纳秒级的`Unix time`，它是可选的，如果不传，`InfluxDB`会使用服务器的本地的纳米级的`timestamp`来作为数据的时间戳，注意无论哪种方式，在`InfluxDB`中的`timestamp`只能是`UTC`时间。

- 写入文件中的数据

之遥文件格式满足`influxdb` 的语法, 就可以通过 `@filename` 的方式写入数据

给定一个例子: `file.txt`

```txt
cpu_load_short,host=server02 value=0.67
cpu_load_short,host=server02,region=us-west value=0.55 1422568543702900257
cpu_load_short,direction=in,host=server01,region=us-west value=2.0 1422568543702900257
```

如何将文件中的数据写入到 `cmzhu` 数据库中呢?

```bash
curl -i -XPOST 'http://localhost:8086/write?db=cmzhu' --data-binary @file.txt
```

**返回含义分析**

- 2xx : 表示正常写入`context`, 表示写入成功
- 4xx : 表示`influxdb` 不识别写入的数据是什么?
- 5xx : 服务端异常, 系统过载或者受损
