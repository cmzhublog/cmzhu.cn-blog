## docker 运行容器失败

### 报错

```bash
library initialization failed - unable to allocate file descriptor table - out of memoryAborted (core dumped)
```

### 解决思路

1、修改主机和容器的ulimit值, 可通过ulimit -n 查看， 使用如下命令修改

```bash
$ ulimit -n 65535
```

2、容器修改

```bash
$ docker run -d \
  --name rocketmq-dashboard \
  --ulimit nofile=65536:65536 \ ## 指定ulimit 值
  -e "JAVA_OPTS=-Drocketmq.namesrv.addr=127.0.0.1:9876" \
  -p 8080:8080 \
  -t apacherocketmq/rocketmq-dashboard:latest
```

