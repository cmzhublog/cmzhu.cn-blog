## mysql 主从同步异常后如何恢复同步

### 背景

mysql 主从同步时，因为一些不知名原因导致主从同步异常；

解决思路：

1. 表级别同步异常，可以跳过部分表继续同步
2. 如果方案1 不生效或者有大量表都不同步，这时候就需要重新全量同步

### 跳过部分表的同步（GTID同步）

若启用GTID，需手动设置空事务来跳过特定GTID：

```sql
STOP SLAVE;
SET GTID_NEXT='cf716fda-74e2-11e2-b7b7-000c290a6b8f:2';  -- 替换为实际报错的GTID
BEGIN; COMMIT;
SET GTID_NEXT='AUTOMATIC';
START SLAVE;
```

通过SHOW SLAVE STATUS\G 获取GTID 值；



### 全量备份恢复方案（GTID 同步）



执行前需锁住主库，可读不可写

```sql
FLUSH TABLES WITH READ LOCK;
```



 当数据库差异过大时，建议重建从库

1、 主库执行全量备份

```bash
$ mysqldump --all-databases --master-data=1 --single-transaction > master_backup.sql
```

> --master-data: 是 MySQL 数据库备份工具的一个关键参数，主要用于在主从复制（Replication）场景中记录主库的二进制日志（binlog）位置信息
>
> - 如果从零新建从库，**--master-data=1** 时，`CHANGE MASTER TO` 语句‌**不被注释**‌，导入备份文件时会自动执行此语句
> - 如果删除旧从库数据，重新恢复： **--master-data=2** **注释**‌ `CHANGE MASTER TO` 语句‌，需要手动配置主从同步

2、查看备份文件中的gtid 位点，如果sql 文件中下述设置位点命令不存在，则需要去主库中查询，并在从库中手动执行，如果存在，则不在导入时会自动执行

```bash
$ cat master_backup.sql | grep GTID_PURGED
SET @@GLOBAL.GTID_PURGED=/*!80000 '+'*/ 'dd32df07-3d3a-1xf0-852d-fa163e959e63:1-6995841';
```

3、在从库中执行数据导入操作,先停止SLAVE 

```bash
# 在从库中执行
$ STOP SLAVE;
# 清理当前卡住的gtid 同步信息
$ RESET MASTER ;
```

4、 在从库中导入备份文件,source 全量备份文件

```bash
$ source master_backup.sql;
```

5、 启动slave ,

```bash
$ START SLAVE;
```

6、查看SLAVE 状态,根据具体的同步状态来做不同的事情

```bash
$ SHOW SLAVE STATUS;
```

