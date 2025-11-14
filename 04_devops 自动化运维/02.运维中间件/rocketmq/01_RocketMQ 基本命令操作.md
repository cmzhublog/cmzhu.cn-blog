## RocketMQ 基本操作

### 1、删除topic 操作

有时候需要在命令行重建topic，具体操作步骤如下

1、 删除topic

```bash
#!/bin/bash
ROCKETMQ_HOME="/data/rocketmq/rocketmq-5.1.4"
NAMESRV_ADDR="127.0.0.1:9876"
TOPIC_NAME="test"
Cluster_Name="DefaultCluster"
GROUP_NAME="cmzhu"
${ROCKETMQ_HOME}/bin/mqadmin deleteTopic -n "${NAMESRV_ADDR}" -t "${TOPIC_NAME}" -c "${Cluster_Name}"
```

2、 通过如下命令可以获取cluster name

```bash
$ ${ROCKETMQ_HOME}/bin/mqadmin clusterList -n 127.0.0.1:9876
```

3、创建topic

```bash
$ ${ROCKETMQ_HOME}/bin/mqadmin  updateTopic -n "${NAMESRV_ADDR}" -t "${TOPIC_NAME}" -c "${Cluster_Name}" -a +message.type=FIFO
```

4、根据topic 查看消息

```bash
$ ${ROCKETMQ_HOME}/bin/mqadmin printMsg -t keyword  -n 127.0.0.1:9876 -s 1818195464378384384
```

### 2、创建Topic 操作

通过如下命令创建Topic

```bash
$ ${ROCKETMQ_HOME}/bin/mqadmin  updateTopic -n "${NAMESRV_ADDR}" -t "${TOPIC_NAME}" -c "${Cluster_Name}" -a +message.type=FIFO
```

### 3、 创建`Customer Group`

通过以下命令创建消费者组

```bash
$ ${ROCKETMQ_HOME}/bin/mqadmin updateSubGroup  -g ${GROUP_NAME} -b "${Cluster_Name}"
```

查询命令

```bash
$  ${ROCKETMQ_HOME}/bin/mqadmin consumerProgress -g ${GROUP_NAME}
```



如果遇见不能查询消费者组的情况,例如：

```bash
./mqadmin consumerProgress -g xxxx
org.apache.rocketmq.tools.command.SubCommandException: ConsumerProgressSubCommand command failed
	at org.apache.rocketmq.tools.command.consumer.ConsumerProgressSubCommand.execute(ConsumerProgressSubCommand.java:269)
	at org.apache.rocketmq.tools.command.MQAdminStartup.main0(MQAdminStartup.java:181)
	at org.apache.rocketmq.tools.command.MQAdminStartup.main(MQAdminStartup.java:131)
Caused by: org.apache.rocketmq.client.exception.MQClientException: CODE: 17  DESC: No topic route info in name server for the topic: %RETRY%xxxx
See https://rocketmq.apache.org/docs/bestPractice/06FAQ for further details.
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.getTopicRouteInfoFromNameServer(MQClientAPIImpl.java:2110)
	at org.apache.rocketmq.client.impl.MQClientAPIImpl.getTopicRouteInfoFromNameServer(MQClientAPIImpl.java:2081)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExtImpl.examineTopicRouteInfo(DefaultMQAdminExtImpl.java:602)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExtImpl.examineConsumeStats(DefaultMQAdminExtImpl.java:440)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExtImpl.examineConsumeStats(DefaultMQAdminExtImpl.java:413)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExt.examineConsumeStats(DefaultMQAdminExt.java:322)
	at org.apache.rocketmq.tools.admin.DefaultMQAdminExt.examineConsumeStats(DefaultMQAdminExt.java:309)
	at org.apache.rocketmq.tools.command.consumer.ConsumerProgressSubCommand.execute(ConsumerProgressSubCommand.java:123)
	... 2 more
```

可通过创建一个Retry Topic 进行处理

```bash
$ ${ROCKETMQ_HOME}/bin/mqadmin updateTopic  -c "${Cluster_Name}" -t %RETRY%${GROUP_NAME}
```

