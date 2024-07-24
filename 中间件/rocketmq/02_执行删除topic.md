## 删除topic 操作

有时候需要在命令行重建topic，具体操作步骤如下

1、 删除topic

```bash
#!/bin/bash
ROCKETMQ_HOME="/data/rocketmq/rocketmq-5.1.4"
NAMESRV_ADDR="127.0.0.1:9876"
TOPIC_NAME="test"
Cluster_Name="DefaultCluster"
${ROCKETMQ_HOME}/bin/mqadmin deleteTopic -n "${NAMESRV_ADDR}" -t "${TOPIC_NAME}" -c "${Cluster_Name}"
```

2、 通过如下命令可以获取cluster name

```bash
${ROCKETMQ_HOME}/bin/mqadmin clusterList -n 127.0.0.1:9876
```

3、创建topic

```bash
${ROCKETMQ_HOME}/bin/mqadmin  updateTopic -n "${NAMESRV_ADDR}" -t "${TOPIC_NAME}" -c "${Cluster_Name}" -a +message.type=FIFO
```

