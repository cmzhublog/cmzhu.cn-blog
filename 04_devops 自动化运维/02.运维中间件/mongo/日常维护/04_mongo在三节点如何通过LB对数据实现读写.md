## mongo 集群模式下通过LB 地址访问集群异常

### 背景

 问题描述

在集群内部搭建了一个mongo三节点集群,集群详细信息如下`rs.status()`

```json
rs0 [direct: secondary] admin> rs.status()
{
  set: 'rs0',
  date: ISODate('2025-09-16T02:59:48.258Z'),
  myState: 2,
  term: Long('3'),
  syncSourceHost: '192.168.101.100:27017',
  syncSourceId: 2,
  heartbeatIntervalMillis: Long('2000'),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
    lastCommittedWallTime: ISODate('2025-09-16T02:59:44.482Z'),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
    appliedOpTime: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
    durableOpTime: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
    lastAppliedWallTime: ISODate('2025-09-16T02:59:44.482Z'),
    lastDurableWallTime: ISODate('2025-09-16T02:59:44.482Z')
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1757991564, i: 1 }),
  members: [
    {
      _id: 0,
      name: '192.168.101.101:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 2159024,
      optime: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
      optimeDate: ISODate('2025-09-16T02:59:44.000Z'),
      lastAppliedWallTime: ISODate('2025-09-16T02:59:44.482Z'),
      lastDurableWallTime: ISODate('2025-09-16T02:59:44.482Z'),
      syncSourceHost: '192.168.101.100:27017',
      syncSourceId: 2,
      infoMessage: '',
      configVersion: 1,
      configTerm: 3,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: '192.168.101.102:27017',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 2159021,
      optime: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
      optimeDurable: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
      optimeDate: ISODate('2025-09-16T02:59:44.000Z'),
      optimeDurableDate: ISODate('2025-09-16T02:59:44.000Z'),
      lastAppliedWallTime: ISODate('2025-09-16T02:59:44.482Z'),
      lastDurableWallTime: ISODate('2025-09-16T02:59:44.482Z'),
      lastHeartbeat: ISODate('2025-09-16T02:59:48.152Z'),
      lastHeartbeatRecv: ISODate('2025-09-16T02:59:48.152Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1754419909, i: 1 }),
      electionDate: ISODate('2025-08-05T18:51:49.000Z'),
      configVersion: 1,
      configTerm: 3
    },
    {
      _id: 2,
      name: '192.168.101.100:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 2159021,
      optime: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
      optimeDurable: { ts: Timestamp({ t: 1757991584, i: 1 }), t: Long('3') },
      optimeDate: ISODate('2025-09-16T02:59:44.000Z'),
      optimeDurableDate: ISODate('2025-09-16T02:59:44.000Z'),
      lastAppliedWallTime: ISODate('2025-09-16T02:59:44.482Z'),
      lastDurableWallTime: ISODate('2025-09-16T02:59:44.482Z'),
      lastHeartbeat: ISODate('2025-09-16T02:59:48.154Z'),
      lastHeartbeatRecv: ISODate('2025-09-16T02:59:48.153Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: '192.168.101.102:27017',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 1,
      configTerm: 3
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1757991584, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('tJ3y+ypCIZwN5wm0dYBJtcNw3/w=', 0),
      keyId: Long('7514145735047118854')
    }
  },
  operationTime: Timestamp({ t: 1757991584, i: 1 })
}
```

在外部需要使用python 的pymongo 来连接该集群



### 处理过程

1、 建立TCP 四层代理，这里使用云厂商提供的TCP 层的LB 进行实现，也可以使用nginx 来实现四层代理；例如：

> 公网IP地址为： real_public_ip

```nginx
stream {
    upstream mongodb_cluster {
        server 192.168.101.100:27017;
        server 192.168.101.101:27017;
        server 192.168.101.102:27017;
    }
    server {
        listen 27017;
        proxy_pass mongodb_cluster;
        proxy_timeout 60s;
        proxy_responses 1;  # 避免代理缓冲导致元数据延迟
    }
}
```

2、使用python3 来进行连接验证

```python
from pymongo import MongoClient
client = MongoClient("mongodb://username:password@real_public_ip:27017/test?replicaSet=rs0&authSource=test")
```

报错如下，在使用过程中还是使用了内网地址进行连接，需要进一步排查

```tex
>>> client.server_info()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/python3.9/site-packages/pymongo/synchronous/mongo_client.py", line 2311, in server_info
    self.admin.command(
  File "/usr/local/lib/python3.9/site-packages/pymongo/_csot.py", line 125, in csot_wrapper
    return func(self, *args, **kwargs)
  File "/usr/local/lib/python3.9/site-packages/pymongo/synchronous/database.py", line 931, in command
    with self._client._conn_for_reads(read_preference, session, operation=command_name) as (
  File "/usr/local/lib/python3.9/site-packages/pymongo/synchronous/mongo_client.py", line 1885, in _conn_for_reads
    server = self._select_server(read_preference, session, operation)
  File "/usr/local/lib/python3.9/site-packages/pymongo/synchronous/mongo_client.py", line 1833, in _select_server
    server = topology.select_server(
  File "/usr/local/lib/python3.9/site-packages/pymongo/synchronous/topology.py", line 409, in select_server
    server = self._select_server(
  File "/usr/local/lib/python3.9/site-packages/pymongo/synchronous/topology.py", line 387, in _select_server
    servers = self.select_servers(
  File "/usr/local/lib/python3.9/site-packages/pymongo/synchronous/topology.py", line 294, in select_servers
    server_descriptions = self._select_servers_loop(
  File "/usr/local/lib/python3.9/site-packages/pymongo/synchronous/topology.py", line 344, in _select_servers_loop
    raise ServerSelectionTimeoutError(
pymongo.errors.ServerSelectionTimeoutError: 192.168.101.100:27017: timed out (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms),192.168.101.102:27017: [Errno 111] Connection refused (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms),192.168.101.101:27017: timed out (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms), Timeout: 30s, Topology Description: <TopologyDescription id: 68c8dadce3986fbb2476422f, topology_type: ReplicaSetNoPrimary, servers: [<ServerDescription ('192.168.101.100', 27017) server_type: Unknown, rtt: None, error=NetworkTimeout('192.168.101.100:27017: timed out (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms)')>, <ServerDescription ('192.168.101.102', 27017) server_type: Unknown, rtt: None, error=AutoReconnect('192.168.101.102:27017: [Errno 111] Connection refused (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms)')>, <ServerDescription ('192.168.101.101', 27017) server_type: Unknown, rtt: None, error=NetworkTimeout('192.168.101.101:27017: timed out (configured timeouts: socketTimeoutMS: 20000.0ms, connectTimeoutMS: 20000.0ms)')>]>
```

3、通过deepseek 得知使用强制直接连接，可忽略mongo 的rs0 配置host 参数

优化后的连接语句如下：

```python
from pymongo import MongoClient
client = MongoClient("mongodb://username:password@real_public_ip:27017/test?replicaSet=rs0&authSource=test",directConnection="true",retryWrites=True)
```

验证新语句能否达到想要的效果,显示连接成功

```python
> client.server_info()
{'version': '7.0.21', 'gitVersion': 'a47b62aff2bae1914085c3ef1d90fc099acf000c', 'modules': [], 'allocator': 'tcmalloc', 'javascriptEngine': 'mozjs', 'sysInfo': 'deprecated', 'versionArray': [7, 0, 21, 0], 'openssl': {'running': 'OpenSSL 3.0.2 15 Mar 2022', 'compiled': 'OpenSSL 3.0.2 15 Mar 2022'}, 'buildEnvironment': {'distmod': 'ubuntu2204', 'distarch': 'x86_64', 'cc': '/opt/mongodbtoolchain/v4/bin/gcc: gcc (GCC) 11.3.0', 'ccflags': '-Werror -include mongo/platform/basic.h -ffp-contract=off -fasynchronous-unwind-tables -g2 -Wall -Wsign-compare -Wno-unknown-pragmas -Winvalid-pch -gdwarf-5 -fno-omit-frame-pointer -fno-strict-aliasing -O2 -march=sandybridge -mtune=generic -mprefer-vector-width=128 -Wno-unused-local-typedefs -Wno-unused-function -Wno-deprecated-declarations -Wno-unused-const-variable -Wno-unused-but-set-variable -Wno-missing-braces -fstack-protector-strong -gdwarf64 -Wa,--nocompress-debug-sections -fno-builtin-memcmp -Wimplicit-fallthrough=5', 'cxx': '/opt/mongodbtoolchain/v4/bin/g++: g++ (GCC) 11.3.0', 'cxxflags': '-Woverloaded-virtual -Wpessimizing-move -Wno-maybe-uninitialized -fsized-deallocation -Wno-deprecated -std=c++20', 'linkflags': '-Wl,--fatal-warnings -B/opt/mongodbtoolchain/v4/bin -gdwarf-5 -pthread -Wl,-z,now -fuse-ld=lld -fstack-protector-strong -gdwarf64 -Wl,--build-id -Wl,--hash-style=gnu -Wl,-z,noexecstack -Wl,--warn-execstack -Wl,-z,relro -Wl,--compress-debug-sections=none -Wl,-z,origin -Wl,--enable-new-dtags', 'target_arch': 'x86_64', 'target_os': 'linux', 'cppdefines': 'SAFEINT_USE_INTRINSICS 0 PCRE2_STATIC NDEBUG _XOPEN_SOURCE 700 _GNU_SOURCE _FORTIFY_SOURCE 2 ABSL_FORCE_ALIGNED_ACCESS BOOST_ENABLE_ASSERT_DEBUG_HANDLER BOOST_FILESYSTEM_NO_CXX20_ATOMIC_REF BOOST_LOG_NO_SHORTHAND_NAMES BOOST_LOG_USE_NATIVE_SYSLOG BOOST_LOG_WITHOUT_THREAD_ATTR BOOST_MATH_NO_LONG_DOUBLE_MATH_FUNCTIONS BOOST_SYSTEM_NO_DEPRECATED BOOST_THREAD_USES_DATETIME BOOST_THREAD_VERSION 5'}, 'bits': 64, 'debug': False, 'maxBsonObjectSize': 16777216, 'storageEngines': ['devnull', 'wiredTiger'], 'ok': 1.0, '$clusterTime': {'clusterTime': Timestamp(1757992924, 1), 'signature': {'hash': b'\xaeB\xebL\xb3\xee~R\xf2\xf8\x97\xa3`\x7f\x0fn\x066\xc5\xf2', 'keyId': 7514145735047118854}}, 'operationTime': Timestamp(1757992924, 1)}
```



