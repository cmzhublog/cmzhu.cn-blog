## Mysql 搭建MHA 高可用方案

服务器资源类型

| 主机IP         | 主机名        | 用途                                | 操作系统      |
| -------------- | ------------- | ----------------------------------- | ------------- |
| 192.168.101.62 | mha-master    | mysql Master 节点                   | Rocky Linux 9 |
| 192.168.101.61 | mha-master-bk | mysql backup 节点， 可提升为Master  | Rocky Linux 9 |
| 192.168.101.60 | Mha-slave     | mysql slave 节点， 不可提升为Master | Rocky Linux 9 |
| 192.168.101.60 |               | MHA Manager 节点                    | Rocky Linux 9 |

### 部署步骤

1、 优先在所有节点中安装 mysql ，可参考如下文档

[mysql 5.7 手动安装](https://wiki.cmzhu.cn/cmzhu.cn-blog/04_devops%20%E8%87%AA%E5%8A%A8%E5%8C%96%E8%BF%90%E7%BB%B4/02.%E8%BF%90%E7%BB%B4%E4%B8%AD%E9%97%B4%E4%BB%B6/mysql/01_mysql%20%E6%89%8B%E5%8A%A8%E6%90%AD%E5%BB%BA/)

2、在三台主机中分别修改主机名

```bash
# master
$ hostnamectl set-hostname mha-master
# master-bk
$ hostnamectl set-hostname mha-master-bk
# slave 
$ hostnamectl set-hostname mha-slave
```

3、在三台节点之间实现ssh 免密登录，将密钥分别拷贝到其他三台节点

```bash
## 1、创建密钥（三台机器都执行）
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:4U0nreyAHq0K8BbUbEU6AZc2GgKjTmuaWGe6W6DDufE root@mha-master
The key's randomart image is:
+---[RSA 3072]----+
|+ ..ooo          |
|o..+++     .     |
|.o.+*.  . o o    |
|o.o. . + = +     |
|.++ o o S +      |
|=*.* . o o       |
|*+= . o   .      |
| o++ .           |
| .oE.            |
+----[SHA256]-----+

## 实现每台服务都相互
$ ssh-copy-id 192.168.101.60
$ ssh-copy-id 192.168.101.61
$ ssh-copy-id 192.168.101.62
## 实现免密 master-bk 登录到master 和 slave
```

节点都配置主机名相关的hosts（所有节点执行）

```bash
$ cat >> /etc/hosts << EOF
192.168.101.60 mha-slave
192.168.101.61 mha-master-bk
192.168.101.62 mha-master
EOF
```

每个节点关闭selinux 和firewalld （每个节点都执行）

```bash
$ sed -i "s#SELINUX=.*#SELINUX=disabled#g" /etc/selinux/config
$ reboot 
$ systemctl disable --now firewalld 
```



4、开始配置MHA，并对配置文件进行解释，修改master 节点配置

```bash
$ cat > /etc/my.cnf << EOF
[client]
## 设置默认字符集为utf8mb4, 支持完整的Unicode 字符（emoji）
default-character-set=utf8mb4
[mysqld]
## 启用严格模式：禁止无效日期、零除错误、自动创建用户等，确保数据一致性
sql_mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

## 启用GTID全局事务标识，简化主从复制管理
## 行级二进制日志（ROW格式）确保数据一致性
## log_slave_updates允许从库记录复制事件
server-id=1
binlog_format=row
relay-log=relay-bin
## 重点配置 开启gtid 基于
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
log-bin=mysql-bin
sync_binlog = 1
datadir=/usr/local/mysql/data
port=3306
socket=/tmp/mysql.sock
symbolic-links=0
character-set-server=utf8
collation-server=utf8_general_ci
open_files_limit=655350
innodb_open_files=20000
## 支持高并发连接和大数据包传输
max_connections=2532
max_allowed_packet=1024M

# 记录慢查询（>2秒）和未使用索引的查询，便于优化7。
slow_query_log=1
slow_query_log_file=/var/log/mysql/mysql-slow.log
long_query_time=2
log_queries_not_using_indexes=1
skip-name-resolve
[mysqld_safe]
log-error=/var/log/mysql/error.log
EOF

$ chown mysql:mysql /etc/my.cnf

$ systemctl restart mysql
```

5、masker-bk 节点 可使用如下配置进行启动mysql,如下命令在master-bk 中执行

```bash
$ cat > /etc/my.cnf << EOF
[client]
default-character-set=utf8mb4
[mysqld]
sql_mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
server-id=2
binlog_format=row
relay-log=relay-bin
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
log-bin=mysql-bin
read_only=1
sync_binlog = 1
datadir=/usr/local/mysql/data
port=3306
socket=/tmp/mysql.sock
symbolic-links=0
character-set-server=utf8
collation-server=utf8_general_ci
max_connections=2532
open_files_limit=655350
innodb_open_files=20000
max_allowed_packet=1024M
slow_query_log=1
slow_query_log_file=/var/log/mysql/mysql-slow.log
long_query_time=2
log_queries_not_using_indexes=1
skip-name-resolve
[mysqld_safe]
log-error=/var/log/mysql/error.log
EOF

$ chown mysql:mysql /etc/my.cnf

$ systemctl restart mysql
```

6、 slave 节点执行如下命令，并重启mysql

```bash
$ cat > /etc/my.cnf << EOF
[client]
default-character-set=utf8mb4
[mysqld]
sql_mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
server-id=3
binlog_format=row
relay-log=relay-bin
gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON
log-bin=mysql-bin
read_only=1
sync_binlog = 1
datadir=/usr/local/mysql/data
port=3306
socket=/tmp/mysql.sock
symbolic-links=0
character-set-server=utf8
collation-server=utf8_general_ci
max_connections=2532
open_files_limit=655350
innodb_open_files=20000
max_allowed_packet=1024M
slow_query_log=1
slow_query_log_file=/var/log/mysql/mysql-slow.log
long_query_time=2
log_queries_not_using_indexes=1
skip-name-resolve
[mysqld_safe]
log-error=/var/log/mysql/error.log
EOF

$ chown mysql:mysql /etc/my.cnf

$ systemctl restart mysql
```

7、 所有节点安装mha4mysql-node 脚本，此脚本用于实现主从切换

[项目仓库](https://github.com/yoshinorim/mha4mysql-node)

Rocky Linux 9 安装如下

```bash
$ wget https://github.com/yoshinorim/mha4mysql-node/releases/download/v0.58/mha4mysql-node-0.58-0.el7.centos.noarch.rpm && dnf localinstall mha4mysql-node-0.58-0.el7.centos.noarch.rpm -y 
```

Ubuntu22 安装时需要下载依赖

```bash
$ apt install -y libdbd-mysql-perl libconfig-tiny-perl liblog-dispatch-perl libparallel-forkmanager-perl
```

验证是否安装完成

```bash
$ perl -e 'use DBD::mysql; print "Module loaded\n"'
```

8、管理节点安装 MHA manager 软件(Rocky Linux 9 )

[项目仓库]( https://github.com/yoshinorim/mha4mysql-manager/)

验证得知，不能直接使用原始的安装包进行安装，只能通过编译安装， [参考文档](https://github.com/yoshinorim/mha4mysql-manager/wiki/Installation)

```bash
$ dnf install -y  epel-release
```

对于无法使用 dnf 下载的依赖，可以使用cpan 进行安装

```bash
$ dnf install -y perl-CPAN-2.29-5.el9_6.noarch

$ cpan -i Log::Dispatch
$ cpan -i Log::Dispatch::File
$ cpan -i Log::Dispatch::Screen
$ cpan -i  inc::Module::Install
```

开始编译安装

```bash
$ wget https://github.com/yoshinorim/mha4mysql-manager/releases/download/v0.58/mha4mysql-manager-0.58.tar.gz && tar -zxvf mha4mysql-manager-0.58.tar.gz && cd mha4mysql-manager-0.58

# 生成makefile
$ perl Makefile.PL
# 编译安装
$ make
$ sudo make install
```

9、master 节点创建 需要使用的账户(master 节点执行)

主从复制账户

```sql
> CREATE USER 'repl'@'%' IDENTIFIED BY 'bFgs4TDJ!kK@RNQM';
> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
```

创建mha 账户

```sql
> CREATE USER 'mha'@'%' IDENTIFIED BY 'bFgs4TDJ!kK@RNQM';
> GRANT ALL PRIVILEGES ON *.*  TO 'mha'@'%';
```

10、 主库启用半同步复制(master 节点执行)

```bash
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
SET GLOBAL rpl_semi_sync_master_enabled=1;
```

11、Slave 节点开始配置主从 （master-bk 和slave 都需要执行）

```sql
> CHANGE MASTER TO
    MASTER_HOST='mha-master',
    MASTER_USER='repl',
    MASTER_PASSWORD='bFgs4TDJ!kK@RNQM',
    MASTER_AUTO_POSITION=1;
> START SLAVE;
```

从库执行实现半同步复制

```sql
> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
> SET GLOBAL rpl_semi_sync_slave_enabled=1;
```

12、开始配置MHA 管理员配置,将如下配置写入/etc/mha/mha.conf  文件，后续根据实际情况调整配置，如下仅适用本文本

```bash
cat > /etc/mha/mha.conf << EOF
[server default]
manager_log=/var/log/mha/app/manager.log
manager_workdir=/var/log/mha/app
master_binlog_dir=/var/lib/mysql
user=mha
password=bFgs4TDJ!kK@RNQM 
ssh_user=root
repl_user=repl
repl_password=bFgs4TDJ!kK@RNQM
ping_interval=3

[server1]
hostname=mha-master
port=3306
candidate_master=1

[server2]
hostname=mha-master-bk
port=3306
candidate_master=1

[server3]
hostname=mha-slave
port=3306
no_master=1
EOF
```

验证MHA 功能

```bash
# 验证 SSH 互通
$ masterha_check_ssh --conf=/etc/mha/mha.conf
# 验证主从复制
$ masterha_check_repl --conf=/etc/mha/mha.conf
```

13、创建mha systemd 管理，并执行,创建守护进程 mha-app

```bash
$ cat > /etc/systemd/system/mha-app.service << EOF
[Unit]
Description=MHA Manager for app
After=network.target mysqld.service

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/masterha_manager --conf=/etc/mha/mha.conf
WorkingDirectory=/var/log/mha/app
StandardOutput=append:/var/log/mha/app/manager.log
StandardError=inherit
Restart=on-failure
RestartSec=5s
TimeoutSec=300

[Install]
WantedBy=multi-user.target
EOF
$ systemctl daemon-reload
$ systemctl restart mha-app
```







### 问题处理

1、uuid 配置一致，需要修改uuid（虚拟机验证时克隆导致的故障）

```tex
                Last_IO_Error: Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.
```

处理方案,编辑数据目录中的auto.cnf 文件，本文数据目录为（/usr/local/mysql/data）

```bash
vim /usr/local/mysql/data/auto.cnf
```

执行完成后使用如下命令验证

```sql
> SHOW VARIABLES LIKE 'server_uuid';
```



2、 执行检查 `masterha_check_repl --conf=/etc/mha/mha.conf` 报错：

```bash
$ masterha_check_repl --conf=/etc/mha/mha.conf
Tue Jul  8 16:19:22 2025 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Tue Jul  8 16:19:22 2025 - [info] Reading application default configuration from /etc/mha/mha.conf..
Tue Jul  8 16:19:22 2025 - [info] Reading server configuration from /etc/mha/mha.conf..
Tue Jul  8 16:19:22 2025 - [info] MHA::MasterMonitor version 0.58.
Creating directory /var/log/mha/app.. done.
Tue Jul  8 16:19:24 2025 - [error][/usr/local/share/perl5/5.32/MHA/MasterMonitor.pm, ln427] Error happened on checking configurations. Redundant argument in sprintf at /usr/share/perl5/vendor_perl/MHA/NodeUtil.pm line 201.
Tue Jul  8 16:19:24 2025 - [error][/usr/local/share/perl5/5.32/MHA/MasterMonitor.pm, ln525] Error happened on monitoring servers.
Tue Jul  8 16:19:24 2025 - [info] Got exit code 1 (Not master dead).

MySQL Replication Health is NOT OK!
```

处理方案：

1、编辑 文件/usr/local/share/perl5/5.32/MHA/MasterMonitor.pm 

```perl
sub parse_mysql_major_version($) {
  my $str = shift;
  my $result = sprintf( '%03d%03d', $str =~ m/(\d+)/g );
  return $result;
}
```

修改后的方法

```perl
sub parse_mysql_major_version($) {
  my $str = shift;
  my ($first, $second) = ($str =~ m/(\d+)/g);
  $first  //= 0;  # 设置默认值
  $second //= 0;
  my $result = sprintf('%03d%03d', $first, $second);
  return $result;
}
```



