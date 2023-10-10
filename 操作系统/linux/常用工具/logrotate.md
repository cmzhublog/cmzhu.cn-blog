---
title: logrotate
description: 
published: true
date: 2023-07-04T06:57:05.159Z
tags: linux
editor: markdown
dateCreated: 2023-07-04T06:53:48.277Z
---

## logrotate

### 用途

用于linux 系统中的日志转存和切割神器; 日常使用中 日志量会逐渐增大之后, 对旧日志作转存会非常有用处; 可以有效防止日志文件无限增大

### logrotate 运行机制

- `logrotate` 是`linux` 默认安装的
- 系统定时运行`logrotate`, 一般是每天一次
- 旧版使用 `/etc/cron.daily/logrotate` 实现, 例如 `Centos 7`, `Centos 6`
- 新版使用 `logrotate.timer`, 例如 `Rocky Linux 9`

服务查看

```bash
> systemctl status logrotate.timer 
● logrotate.timer - Daily rotation of log files
     Loaded: loaded (/usr/lib/systemd/system/logrotate.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Sat 2023-04-22 09:48:09 CST; 2 months 12 days ago
      Until: Sat 2023-04-22 09:48:09 CST; 2 months 12 days ago
    Trigger: Wed 2023-07-05 00:00:00 CST; 9h left
   Triggers: ● logrotate.service
       Docs: man:logrotate(8)
             man:logrotate.conf(5)

Notice: journal has been rotated since unit was started, output may be incomplete.

```

查看`logrotate.timer` 详细配置
```bash
> cat /usr/lib/systemd/system/logrotate.timer
[Unit]
Description=Daily rotation of log files
Documentation=man:logrotate(8) man:logrotate.conf(5)

[Timer]
OnCalendar=daily
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target
```


参数用途如下
```text
[Unit]
Description=Daily rotation of log files                    : 表示该timer 的描述信息, `Daily rotation of log files ` 表示每天轮转日志
Documentation=man:logrotate(8) man:logrotate.conf(5)       : 提供文档地址获取方案, 提供文档查询相关方法

[Timer]
OnCalendar=daily                                           : 指定时间触发表, 参数为daily,表示每天执行一次
AccuracySec=1h                                             : 指定了定时器的准确性。设置为 1h 表示定时器的触发时间可能会在预定时间上最多有 1 小时的偏差。这样的设置可以提高系统对于定时任务的灵活性，并避免精确时间同步所带来的负担。
Persistent=true                                            : 设置为 true，表示当定时器超过设定的时间间隔后被触发时，它将保持激活状态。这意味着即使在系统重新启动后，logrotate.timer 仍然会保持激活并按照预定的时间表执行。

[Install]
WantedBy=timers.target                                     : 指定了该单位所属的目标单元。在这种情况下，logrotate.timer 被设置为 timers.target，这意味着它将作为一个定时器单元在系统启动时自动安装和启用。
```

这个`logrotate.timer` 是如何引用的, 如下为查看`logrotate.service` 的状态, 可以发现 `TriggeredBy: ● logrotate.timer`, 可以发小依赖于
```bash
> systemctl status logrotate.service 
○ logrotate.service - Rotate log files
     Loaded: loaded (/usr/lib/systemd/system/logrotate.service; static)
     Active: inactive (dead) since Tue 2023-07-04 14:06:54 CST; 23min ago
TriggeredBy: ● logrotate.timer
       Docs: man:logrotate(8)
             man:logrotate.conf(5)
   Main PID: 1295861 (code=exited, status=0/SUCCESS)
        CPU: 606ms

Jul 04 14:06:54 bqdev01 systemd[1]: Starting Rotate log files...
Jul 04 14:06:54 bqdev01 systemd[1]: logrotate.service: Deactivated successfully.
Jul 04 14:06:54 bqdev01 systemd[1]: Finished Rotate log files.
``` 

查看 logrotate.service 的配置
```bash
> cat /usr/lib/systemd/system/logrotate.service
[Unit]
Description=Rotate log files
Documentation=man:logrotate(8) man:logrotate.conf(5)
RequiresMountsFor=/var/log
ConditionACPower=true

[Service]
Type=oneshot
ExecStart=/usr/sbin/logrotate /etc/logrotate.conf

# performance options
Nice=19
IOSchedulingClass=best-effort
IOSchedulingPriority=7

# hardening options
#  details: https://www.freedesktop.org/software/systemd/man/systemd.exec.html
#  no ProtectHome for userdir logs
#  no PrivateNetwork for mail deliviery
#  no NoNewPrivileges for third party rotate scripts
#  no RestrictSUIDSGID for creating setgid directories
LockPersonality=true
MemoryDenyWriteExecute=true
PrivateDevices=true
PrivateTmp=true
ProtectClock=true
ProtectControlGroups=true
ProtectHostname=true
ProtectKernelLogs=true
ProtectKernelModules=true
ProtectKernelTunables=true
ProtectSystem=full
RestrictNamespaces=true
RestrictRealtime=true
```

### logrotate 相关配置

1、 查看`/etc/logrotate.conf` 并解释含义

```bash
> cat /etc/logrotate.conf
# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly   # 每周运行一次

# keep 4 weeks worth of backlogs
rotate 4   # 备份数据保持四份

# create new (empty) log files after rotating old ones
create     # 默认使用 create方式切割数据

# use date as a suffix of the rotated file
dateext    # 重命名转存文件, 带上日期

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d     # 导入其他转存配置

# system-specific logs may be also be configured here.
```

logrotate 配置详细参数如下

- compress：压缩被轮转后的日志文件（默认启用）。
- delaycompress：在下一次轮转之前延迟压缩上一次轮转生成的日志文件。
- copytruncate：将原始日志文件清空，并继续写入，而不是创建新的日志文件。(不会修改文件inode)
- nocopytruncate：不使用copytruncate选项，即创建新的日志文件。 (会修改文件inode)
- missingok：如果日志文件不存在，则不报错，继续进行下一个操作。
- notifempty：如果日志文件为空，则不执行轮转操作。
- sharedscripts：在所有轮转的日志文件完成后执行一次脚本，而不是每个日志文件都执行。
- postrotate <command>：在轮转完成后执行用户定义的命令。
- prerotate <command>：在轮转开始之前执行用户定义的命令。
- firstaction <command>：在第一次轮转时执行用户定义的命令。
- lastaction <command>：在最后一次轮转后执行用户定义的命令。
- create <mode> <user> <group>：创建新的日志文件并设置权限、用户和组。 (修改日志文件inode)
- rotate <count>：保留的轮转文件数量。
- size <size>：触发轮转的日志文件大小。
- time <time>：指定轮转的时间间隔。
- dateext：在轮转后的日志文件名中添加日期扩展。
- extension <extension>：指定轮转后的日志文件扩展名。
- ifempty：即使日志文件为空，也执行轮转操作。
- olddir <directory>：将旧的轮转日志文件移动到指定的目录。

例子: 
```bash
> cat /etc/logrotate.d/biggateway 
/var/app/log/xxx/error.log {
    daily
    rotate 10
    minsize 1k
    dateext

    copytruncate
    sharedscripts
    prerotate
        /usr/bin/chattr -a /var/app/log/xxx/error.log
    endscript

    sharedscripts
    postrotate
        /usr/bin/chattr +a /var/app/log/xxx/error.log
    endscript
}
```
  
如何手动之行日志转存

```bash
> logrotate -f /etc/logrotate.d/biggateway

```
