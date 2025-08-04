## 删除数据库 test


```bash
drop database test
```

```bash
SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE datname='test' AND pid<>pg_backend_pid();
```