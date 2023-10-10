---
title: screen
description: 
published: true
date: 2023-05-31T02:33:30.883Z
tags: linux
editor: markdown
dateCreated: 2023-05-31T02:33:30.883Z
---

## screen 使用方法

- `screen` 可用于在 `linux` 情况下的多重窗口管理 
- `screen` 可后台运行, 解决`ssh`下`bash`超时退出的问题

### screen 常用使用
使用screen --help 可以查看 screen的全部选项
```bash
>  screen --help
Use: screen [-opts] [cmd [args]]
 or: screen -r [host.tty]

Options:
-4            Resolve hostnames only to IPv4 addresses.
-6            Resolve hostnames only to IPv6 addresses.
-a            Force all capabilities into each window's termcap.
-A -[r|R]     Adapt all windows to the new display width & height.
-c file       Read configuration file instead of '.screenrc'.
-d (-r)       Detach the elsewhere running screen (and reattach here).
-dmS name     Start as daemon: Screen session in detached mode.
-D (-r)       Detach and logout remote (and reattach here).
-D -RR        Do whatever is needed to get a screen session.
-e xy         Change command characters.
-f            Flow control on, -fn = off, -fa = auto.
-h lines      Set the size of the scrollback history buffer.
-i            Interrupt output sooner when flow control is on.
-l            Login mode on (update /var/run/utmp), -ln = off.
-ls [match]   or
-list         Do nothing, just list our SockDir [on possible matches].
-L            Turn on output logging.
-Logfile file Set logfile name.
-m            ignore $STY variable, do create a new screen session.
-O            Choose optimal output rather than exact vt100 emulation.
-p window     Preselect the named window if it exists.
-q            Quiet startup. Exits with non-zero return code if unsuccessful.
-Q            Commands will send the response to the stdout of the querying process.
-r [session]  Reattach to a detached screen process.
-R            Reattach if possible, otherwise start a new session.
-s shell      Shell to execute rather than $SHELL.
-S sockname   Name this session <pid>.sockname instead of <pid>.<tty>.<host>.
-t title      Set title. (window's name).
-T term       Use term as $TERM for windows, rather than "screen".
-U            Tell screen to use UTF-8 encoding.
-v            Print "Screen version 4.08.00 (GNU) 05-Feb-20".
-wipe [match] Do nothing, just clean up SockDir [on possible matches].
-x            Attach to a not detached screen. (Multi display mode).
-X            Execute <cmd> as a screen command in the specified session.
```

参数说明：
- -A 　将所有的视窗都调整为目前终端机的大小。
- -d<作业名称> 　将指定的screen作业离线。
- -h<行数> 　指定视窗的缓冲区行数。
- -m 　即使目前已在作业中的screen作业，仍强制建立新的screen作业。
- -r<作业名称> 　恢复离线的screen作业。
- -R 　先试图恢复离线的作业。若找不到离线的作业，即建立新的screen作业。
- -s<shell> 　指定建立新视窗时，所要执行的shell。
- -S<作业名称> 　指定screen作业的名称。
- -v 　显示版本信息。
- -x 　恢复之前离线的screen作业。
- -ls或--list 　显示目前所有的screen作业。
- -wipe 　检查目前所有的screen作业，并删除已经无法使用的screen作业。

### 例子
1、 创建一个 screen 终端
```bash
> screen // 创建 screen 终端
```

2、创建一个终端, 并执行相关命令
```bash
> screen tail -f /dev/null
```

3、离开`screen`终端是在`screen` 界面下`ctrl+a d`(这里的离开不是退出,后台仍然在继续运行, 退出使用quit;)

4、 如何查看`screen`终端
```bash
> screen -ls
There are screens on:
	23665.pts-11.control-master004	(Detached)
	17790.sn	(Detached)
	31578.delete_dai_deleted	(Detached)
3 Sockets in /var/run/screen/S-root.
```

5、 如何重新进入终端
```bash
> screen -r 23665
```
