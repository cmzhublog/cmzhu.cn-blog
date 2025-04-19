## mongo 获取当前数据库的连接数信息



mongo 排查问题时需要检查连接数， 下面检查连接数方案

1、登录mongo 数据库

```bash
$ mongosh -u root -p example --authenticationDatabase admin
```

2、使用admin 数据库

```bash
$ use admin
```

3、查询mongo 连接数

```bash
$ db.serverStatus().connections
{
  current: 135,
  available: 838725,
  totalCreated: 536,
  rejected: 0,
  active: 83,
  threaded: 135,
  exhaustIsMaster: 0,
  exhaustHello: 80,
  awaitingTopologyChanges: 82
}

```

