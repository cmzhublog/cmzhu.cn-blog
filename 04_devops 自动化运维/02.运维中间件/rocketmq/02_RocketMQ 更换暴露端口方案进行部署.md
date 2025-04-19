## rocketmq 部署

背景： 当使用rocketmq时发现broker 的`19011`, `10909` 等端口被占用时， 需要重新指定使用的端口， 然而指定时会发现， 简单的改docker-compose 的映射是不能完全修改好的；具体解决步骤如下

新的docker-compose.yml 文件如下：

```yaml
version: '3.8'
services:
  namesrv:
    image: apache/rocketmq:5.3.0
    container_name: rmqnamesrv
    ports:
      - 39876:9876
    networks:
      - rocketmq
    command: sh mqnamesrv
    volumes:
      - ${PWD}/broker/broker.conf:/home/rocketmq/rocketmq-5.3.0/conf/broker.conf
      - ${PWD}/namesrv/logs:/home/rocketmq/logs
      - ${PWD}/namesrv/store:/home/rocketmq/store
  broker:
    image: apache/rocketmq:5.3.0
    container_name: rmqbroker
    volumes:
      - ${PWD}/broker/broker.conf:/home/rocketmq/rocketmq-5.3.0/conf/broker.conf
      - ${PWD}/broker/logs:/home/rocketmq/logs
      - ${PWD}/broker/store:/home/rocketmq/store
    ports:
      - 30911:30911
      - 30909:30909
    environment:
      - NAMESRV_ADDR=namesrv:9876
    depends_on:
      - namesrv
    networks:
      - rocketmq
    command: sh mqbroker -c /home/rocketmq/rocketmq-5.3.0/conf/broker.conf
  proxy:
    image: apache/rocketmq:5.3.0
    container_name: rmqproxy
    networks:
      - rocketmq
    depends_on:
      - broker
      - namesrv
    ports:
      - 38080:8080
      - 38081:8081
    restart: on-failure
    environment:
      - NAMESRV_ADDR=namesrv:9876
    command: sh mqproxy
networks:
  rocketmq:
    driver: bridge

```

调整了broker 的配置文件， 和启动命令； 以及调整了需要暴露的端口信息；

broker.conf 配置文件如下

```bash
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
# add
maxMessageSize = 65536
autoCreateSubscriptionGroup = true ## 允许 Broker 自动创建订阅组
autoCreateTopicEnable = true  ## 允许自动创建topic
namesrvAddr = 192.168.101.100:39876  ## namesrv 的主机暴露地址
brokerIP1 = 192.168.101.100  ## broker 的ip
listenPort = 30911 # 普通通道端口，对应10911
fastListenPort = 30909 # vip 通道端口，对应10909
haListenPort = 30912  # 用于主从同步， 对应10912

```

按照如此方案进行修改之后， 可以直接



RocketMQ-Broker中的端口
**listenPort**
listenPort参数是broker的监听端口号，是remotingServer服务组件使用，作为对Producer和Consumer提供服务的端口号，默认为10911，可以通过配置文件修改。
打开broker-x.conf，修改或增加listenPort参数：

```bash
#Broker 对外服务的监听端口
listenPort=10911
```

**fastListenPort**
fastListenPort参数是fastRemotingServer服务组件使用，默认为listenPort - 2，可以通过配置文件修改。
打开broker-x.conf，修改或增加fastListenPort参数：

```bash
#主要用于slave同步master
fastListenPort=10909
```

**haListenPort**
haListenPort参数是HAService服务组件使用，用于Broker的主从同步，默认为listenPort - 1，可以通过配置文件修改。
打开broker-x.conf，修改或增加haListenPort参数：

```bash
#haService中使用
haListenPort=10912
```



