---
title: postgres迁移数据库
description: 
published: true
date: 2023-01-10T10:23:28.369Z
tags: 中间件
editor: markdown
dateCreated: 2023-01-10T08:36:11.267Z
---

# postgres 数据库备份

## 导出数据

pg_dump 

> 将制定查询,制定数据库导出成文件,下操作将制定的数据库test 导成文件test.sql

```bash
pg_dump -d test -U postgres -f test.sql
```



Pg_dumpall 

> 将所有数据库都导成文件,下面指令将数据库导为testall.sql

```bash
pg_dumpall -U postgres -f testall.sql 
```



以上两种方式是导出成copy 个是,这种格式备份数据,恢复时可能会存在一定问题,可以使用 --inserts 将配置备份成sql 样式,这样会更加稳定,上面的命令可修改成如下

```bash
pg_dump -d test -U postgres -f test.sql
pg_dumpall -U postgres -f testall.sql  --inserts
```



## 导入数据

导入数据就比较简单

```bash
psql -U postgres -f test.sql 
```


## 示例
To dump a database called mydb into an SQL-script file:
```bash
$ pg_dump mydb > db.sql
```
To reload such a script into a (freshly created) database named newdb:
```bash
$ psql -d newdb -f db.sql
```
To dump a database into a custom-format archive file:
```bash
$ pg_dump -Fc mydb > db.dump
```
To dump a database into a directory-format archive:
```bash
$ pg_dump -Fd mydb -f dumpdir
```
To dump a database into a directory-format archive in parallel with 5 worker jobs:
```bash
$ pg_dump -Fd mydb -j 5 -f dumpdir
```
To reload an archive file into a (freshly created) database named newdb:
```bash
$ pg_restore -d newdb db.dump
```
To reload an archive file into the same database it was dumped from, discarding the current contents of that database:
```bash
$ pg_restore -d postgres --clean --create db.dump
```
To dump a single table named mytab:
```bash
$ pg_dump -t mytab mydb > db.sql
```
To dump all tables whose names start with emp in the detroit schema, except for the table named employee_log:
```bash
$ pg_dump -t 'detroit.emp*' -T detroit.employee_log mydb > db.sql
```
To dump all schemas whose names start with east or west and end in gsm, excluding any schemas whose names contain the word test:
```bash
$ pg_dump -n 'east*gsm' -n 'west*gsm' -N '*test*' mydb > db.sql
```
The same, using regular expression notation to consolidate the switches:
```bash
$ pg_dump -n '(east|west)*gsm' -N '*test*' mydb > db.sql
```
To dump all database objects except for tables whose names begin with ts_:
```bash
$ pg_dump -T 'ts_*' mydb > db.sql
```
To specify an upper-case or mixed-case name in -t and related switches, you need to double-quote the name; else it will be folded to lower case (see Patterns below). But double quotes are special to the shell, so in turn they must be quoted. Thus, to dump a single table with a mixed-case name, you need something like
```bash
$ pg_dump -t "\"MixedCaseName\"" mydb > mytab.sql
```
