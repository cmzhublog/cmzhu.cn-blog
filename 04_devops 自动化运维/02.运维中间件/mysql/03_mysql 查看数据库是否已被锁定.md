## 数据库是否被锁表

可通过下面命令查询数据库是否已被锁定(in_use: 0 表示未锁定， 1 表示已锁表)

```sql
$ SHOW OPEN TABLES WHERE `Table` LIKE 'users'; 
```

查看正在锁的事务

```sql
$ SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS; 
```

查看等待锁的事务

```sql
$ SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS; 
```



