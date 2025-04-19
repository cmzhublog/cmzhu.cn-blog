---
title: postgres 删除数据库
description: 
published: true
date: 2023-03-01T06:20:11.204Z
tags: postgres
editor: markdown
dateCreated: 2023-03-01T06:20:11.204Z
---

# 删除数据库 test


```bash
drop database test
```

```bash
SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE datname='test' AND pid<>pg_backend_pid();
```