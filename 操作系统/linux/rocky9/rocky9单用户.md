---
title: rocky9 单用户
description: 
published: true
date: 2023-01-16T02:49:59.097Z
tags: linux
editor: markdown
dateCreated: 2023-01-16T02:47:58.067Z
---

# rocky9 进入单用户

找到系统运行时的一行,并修改配置`rw init=/sysroot/bin/sh`.

```
# rw init=/sysroot/bin/sh
```

![centos8pw1](https://www.layerstack.com/img/docs/resources/centos8pw1.png)

![centos8pw2](https://www.layerstack.com/img/docs/resources/centos8pw2.png)

修改权限
```bash
chroot /sysroot
```

修改密码
```bash
passwd
touch /.autorelabel
reboot -f
```

