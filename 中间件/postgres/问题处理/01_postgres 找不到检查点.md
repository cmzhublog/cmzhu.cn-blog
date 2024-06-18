## postgres 找不到检查点问题

### 问题报错现象

```tex
> postgresql error PANIC: could not locate a valid checkpoint record
```



### 问题分析

- 服务器重启之后导致pg数据写入失败

### 问题解决

1、 postgres < 10

```bash
$ pg_resetxlog -f <DATADIR> 
```

2、postgres >= 10

```bash
$ pg_resetwal -f DATADIR
```

### 知识积累

1、 [WAL](https://www.postgresql.org/docs/current/wal-intro.html) :  预写日志记录( Write-Ahead Logging  ), 是确保数据完整性的有效方法, WAL 的核心概念是:只有在记录更改之后, 描述更改的WAL 记录刷新到磁盘之后, 才会写入对应的数据





