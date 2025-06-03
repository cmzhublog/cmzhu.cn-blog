## mysql 5.7 手动搭建

1、 解决环境依赖，确保当前服务器上只存在一个mysql,多余的服务卸载

```bash
$ apt update 
$ apt upgrade 
$ apt autoremove --purge mysql-server-* mysql-common
$ rm -rf /var/lib/mysql /etc/mysql
```

2、 安装mysql 运行时所需要的依赖环境

```bash
$ apt update 
$ apt install libncurses5 libaio1 libtinfo5 -y
```

3、 开始安装mysql,手动下载安装包， 并解压mysql

```bash
$ cd /usr/local 
$ wget https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.35-linux-glibc2.12-x86_64.tar.gz
$ tar -zxvf mysql-5.7.35-linux-glibc2.12-x86_64.tar.gz && mv mysql-5.7.35-linux-glibc2.12-x86_64 mysql
```

4、 创建mysql 用户，用于运行mysql

```bash
$ groupadd mysql
## 不允许mysql 用户进入shell
$ useradd -r -g mysql -s /bin/false mysql
$ chown -R mysql:mysql /usr/local/mysql
$ chmod -R 755 /usr/local/mysql
```

5、 启动 mysql 服务初始化

```bash
$ cd /usr/local/mysql/bin && ./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data
```

6、创建mysql 启动参数文件 my.cnf

```bash
## /etc/my.cnf
[mysqld]
server-id=1
log_bin=mysql-bin
binlog_format=row
sync_binlog = 1
datadir=/usr/local/mysql/data
port=3306
socket=/tmp/mysql.sock
symbolic-links=0
[mysqld_safe]
log-error=/var/log/mysql/error.log
pid-file=/var/run/mysql/mysql.pid
```

```bash
$ chown mysql.mysql my.cnf
```



7、 创建日志目录

```bash
$ mkdir /var/log/mysql && chown -R mysql:mysql /var/log/mysql
```

8 、 启动mysql 并手动执行初始化,记录并使用初始化密码

```bash
$ /usr/local/mysql/support-files/mysql.server start

## 初始化化密码
$ mysql -uroot -p
 > ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
 > GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
```

9、使用 systemd 进行管理mysql 进程

```bash
### vim /etc/systemd/system/mysql.service

[Unit]
Description=MySQL Server
After=network.target

[Service]
User=mysql
Group=mysql
ExecStart=/usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/data
Restart=always

[Install]
WantedBy=multi-user.target

```

```bash
$ systemctl daemon-reload
$ reboot
```

