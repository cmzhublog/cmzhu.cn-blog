---
title: Linux 进程学习
description: 
published: true
date: 2023-03-01T09:58:39.169Z
tags: linux
editor: markdown
dateCreated: 2023-03-01T09:51:13.826Z
---

# linux 进程用户交互



## 背景

- 在ssh 登录时需要输入密码, 不能使用公钥

- 使用bash 执行交互任务,需要用户输入时也可以使用



## 解决方案

### 使用except 解决问题

except 主要进行自动化的交互，except 能够模拟用户的输入，也可以读取标准输出，这非常适合需要用户输入的场景

expect 关键命令

- `send`”向进程发送字符串，用于模拟用户的输入，注意一定要加 `\r` 回车
- `expect`：从进程接收字符串
- `spawn`：启动进程（由 spawn 启动的进程的输出可以被 expect 所捕获）
- `interact`：用户交互

```bash
#!/usr/bin/env bash

read -r -p "please put IP address:" SSH_IP
read -r -p "please put username:" SSH_USERNAME
read -r -p "please put password:" SSH_PASSWORD

/usr/bin/expect <<-__END__
set timeout 2000

# 启动ssh 进程
spawn ssh -l "${SSH_USERNAME}" "${SSH_IP}"

# 交互式访问格式 
expect {
    "yes/no" {send "yes\r";exp_continue}
    "*Y/N" {send "Y\r";exp_continue}
    "*password*" {send "${SSH_PASSWORD}\r"}
}

# ssh 登录之后需要执行的指令
expect {
    "#*" {
        send "dnf update -y\r"
        send "exit\r"
    }
}
#interact
expect eof
exit
__END__
```



### 使用sshpass 解决远程登录输入密码问题

**安装**

```bash
# rocky linux 9
dnf install sshpass -y

# ubuntu and debian
apt-get install sshpass -y
```



**用法**

使用 -h 查看用法

```bash
[root@cmzhu cmzhu]# sshpass -h
Usage: sshpass [-f|-d|-p|-e] [-hV] command parameters
   -f filename   Take password to use from file
   -d number     Use number as file descriptor for getting password
   -p password   Provide password as argument (security unwise)
   -e            Password is passed as env-var "SSHPASS"
   With no parameters - password will be taken from stdin

   -P prompt     Which string should sshpass search for to detect a password prompt
   -v            Be verbose about what you're doing
   -h            Show help (this screen)
   -V            Print version information
At most one of -f, -d, -p or -e should be used
```

**例子**

输入密码

```bash
sshpass -p <PASSWORD> ssh <USER>@<IP> 'df -h' 
```



使用环境变量

```bash
 SSHPASS='<PASSWORD>' sshpass -e ssh <USER>@<IP> 'df -h' 
```

