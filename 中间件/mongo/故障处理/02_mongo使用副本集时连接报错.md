## mongo 使用副本集时连接报错

### 问题描述

mongo部署方案

- 副本集 ：rs0

- 部署方式： docker 

- 连接方式： pymongo

- 脚本：
  ```python
  from pymongo import MongoClient
  
  # 连接到MongoDB实例
  client = MongoClient('mongodb://localhost:27017/')
  
  # 使用admin数据库
  db = client['admin']
  
  # 查询服务器状态
  server_status = db.command('serverStatus')
  
  # 打印服务器状态信息
  print(server_status)
  ```

  

- 报错如下： 

```bash
$ server_status = db.command('serverStatus')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib64/python3.9/site-packages/pymongo/_csot.py", line 108, in csot_wrapper
    return func(self, *args, **kwargs)
  File "/usr/local/lib64/python3.9/site-packages/pymongo/database.py", line 893, in command
    with self.__client._conn_for_reads(read_preference, session, operation=command_name) as (
  File "/usr/local/lib64/python3.9/site-packages/pymongo/mongo_client.py", line 1375, in _conn_for_reads
    server = self._select_server(read_preference, session, operation)
  File "/usr/local/lib64/python3.9/site-packages/pymongo/mongo_client.py", line 1322, in _select_server
    server = topology.select_server(
  File "/usr/local/lib64/python3.9/site-packages/pymongo/topology.py", line 368, in select_server
    server = self._select_server(
  File "/usr/local/lib64/python3.9/site-packages/pymongo/topology.py", line 346, in _select_server
    servers = self.select_servers(
  File "/usr/local/lib64/python3.9/site-packages/pymongo/topology.py", line 253, in select_servers
    server_descriptions = self._select_servers_loop(
  File "/usr/local/lib64/python3.9/site-packages/pymongo/topology.py", line 303, in _select_servers_loop
    raise ServerSelectionTimeoutError(
pymongo.errors.ServerSelectionTimeoutError: Could not reach any servers in [('mongo', 27017)]. Replica set is configured with internal hostnames or IPs?, Timeout: 30s, Topology Description: <TopologyDescription id: 66bc5c2bc962cd21a880378c, topology_type: ReplicaSetNoPrimary, servers: [<ServerDescription ('mongo', 27017) server_type: Unknown, rtt: None, error=AutoReconnect('mongo:27017: [Errno -3] Temporary failure in name resolution (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms)')>]>
```

### 解决方案

1、连接时指定：directConnection=true

```bash
directConnection=true
```

![image-20240814160057698](./02_mongo%E4%BD%BF%E7%94%A8%E5%89%AF%E6%9C%AC%E9%9B%86%E6%97%B6%E8%BF%9E%E6%8E%A5%E6%8A%A5%E9%94%99.assets/image-20240814160057698.png)

2、 对于副本集来说， 主节点是自动选举出来的；

	- 单节点副本集无法自动选举主节点
	- 多节点副本集可以通过调整优先级来干预自动选举主节点

参考文档

- [https://www.mongodb.com/zh-cn/docs/drivers/go/current/fundamentals/connections/connection-guide/](https://www.mongodb.com/zh-cn/docs/drivers/go/current/fundamentals/connections/connection-guide/)