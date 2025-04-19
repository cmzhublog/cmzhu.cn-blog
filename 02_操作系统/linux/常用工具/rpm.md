---
title: linux 常用命令
description: 
published: true
date: 2023-08-29T01:39:07.548Z
tags: linux
editor: markdown
dateCreated: 2022-12-19T08:46:55.496Z
---

# Linux 常用命令

获取设置开机自启动的节点应用

```bash
systemctl list-unit-files |grep enabled
```

### rpm 包管理
1、 查看某个文件属于rpm 包安装

```bash
rpm -qf /usr/bin/vim
```

2、 确定rpm 包中安装了多少文件
```bash
rpm -ql vim-enhanced-7.4.629-8.el7_9.x86_64

```
3、 nginx 管理配置项功能

http://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_pass

4、 删除 xx 天之前的文件

> 使用`find` 指令找到 所有`7` 天前的文件, 使用`for` 循环遍历并使用`rm -rf` 删除

```bash
for file in $(find . -type f -mtime +7); do rm -rf ${file}; done
```

