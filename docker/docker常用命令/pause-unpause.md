---
title: pause/unpause
description: 
published: true
date: 2022-10-30T01:07:16.642Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:07:16.642Z
---

# docker pause/unpause 命令

**docker pause** :暂停容器中所有的进程。

**docker unpause** :恢复容器中所有的进程。

## 语法

```
docker pause CONTAINER [CONTAINER...]
```

```
docker unpause CONTAINER [CONTAINER...]
```
## 实例
暂停数据库容器db01提供服务。

```
docker pause db01
```

恢复数据库容器 db01 提供服务。

```
docker unpause db01
```