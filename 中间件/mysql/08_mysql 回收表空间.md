## mysql 删除数据

1、mysql 删除大量数据时（DELETE）操作，不会真正从磁盘上删除数据，而是呈现逻辑删除；需要删除实际的磁盘数据需要使用

```sql
$ OPTIMIZE TABLE table_name ;
```

> - 仅InnoDB和MyISAM引擎支持`OPTIMIZE TABLE`语句。
> - 实例剩余磁盘空间需要大于等于需释放表的空间。当实例剩余磁盘空间不足时，建议先扩容磁盘空间(将表拷贝一份临时表，再删除原表，再改名)
>
> - 如果未使用`DELETE`语句删除大量数据，直接执行`OPTIMIZE TABLE`语句将无法有效降低表空间使用率。因此，需要先通过`DELETE`语句删除数据，再执行`OPTIMIZE TABLE`以释放表空间。
> - 在RDS MySQL 5.7和8.0 中，`OPTIMIZE TABLE`采用Online DDL方式执行，支持并发DML操作。但对大表执行该操作可能引发突发的IO和Buffer资源占用，存在锁表或资源抢占风险，业务高峰期可能导致实例不可用或监控中断。因此，建议在业务低峰期执行此操作以避免影响正常业务。