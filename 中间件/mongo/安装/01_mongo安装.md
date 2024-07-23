## mongo 安装

mongodb 存在两个版本： 社区版和企业版

### Rech 版本安装

本安装方式支持redhat 和 Centos

支持一下这些系统的64 位版本

- RHEL / CentOS Stream / Oracle / Rocky / AlmaLinux 9
- RHEL / CentOS Stream / Oracle / Rocky / AlmaLinux 8
- RHEL / CentOS / Oracle 7

#### 安装步骤

1、 配置yum 源， 创建源文件（/etc/yum.repos.d/mongodb-org-7.0.repo）， 用于安装mongodb

```bash
$ cat /etc/yum.repos.d/mongodb-org-7.0.repo/etc/yum.repos.d/mongodb-org-7.0.repo

[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
```

也可以去下载源文件，来安装

```bash
$ wget https://repo.mongodb.org/yum/redhat/mongodb-org-3.1.repo -O /etc/yum.repos.d/mongodb-org-7.0.repo/etc/yum.repos.d/mongodb-org-7.0.repo
```



2、 开始安装软件

```bash
# 安装最新版本的 mongodb
$ yum install -y mongodb-org

# 安装其他的 release 版本, 需要单独指定每一个组件的版本， 如下面链接所示
$ yum install -y mongodb-org-7.0.2 mongodb-org-database-7.0.2 mongodb-org-server-7.0.2 mongodb-mongosh-7.0.2 mongodb-org-mongos-7.0.2 mongodb-org-tools-7.0.2
```

3、 可以指定任何版本， 但是当检查到新版本出现时， yum会自动升级到最新版本， 若要防止意外升级， 需要固定安装包设置办法如下， 请将以下 exclude 指令添加到 /etc/yum.conf 文件中

```bash
exclude=mongodb-org,mongodb-org-database,mongodb-org-server,mongodb-mongosh,mongodb-org-mongos,mongodb-org-tools
```



### mongo 初始化副本集

参考文档： [https://www.mongodb.com/zh-cn/docs/manual/tutorial/convert-standalone-to-replica-set/](https://www.mongodb.com/zh-cn/docs/manual/tutorial/convert-standalone-to-replica-set/)
