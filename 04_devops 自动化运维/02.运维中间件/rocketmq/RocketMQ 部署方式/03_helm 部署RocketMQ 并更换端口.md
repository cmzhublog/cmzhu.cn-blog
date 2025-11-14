## 使用Helm 部署RocketMQ

1、 helm 添加仓库到 rocketm-repo 

```bash
$ helm repo add rocketm-repo 	https://helm-charts.itboon.top/rocketmq
$ helm repo  update
```

2、根据实际情况创建values.yaml 文件

```bash
broker:
  master:
    resources:
      limits:
        cpu: 1
        memory: 2Gi
      requests:
        cpu: 200m
        memory: 1Gi
  replica:
    resources:
      limits:
        cpu: 2
        memory: 4Gi
      requests:
        cpu: 50m
        memory: 1Gi
  persistence:
    enabled: true
    size: 10Gi
    storageClass: "csi-disk"
  aclConfigMapEnabled: true
  aclConfig: |
    globalWhiteRemoteAddresses:
      - '*'
      - 10.*.*.*
      - 192.168.*.*
    accounts:
      - accessKey: RocketMQ
        secretKey: 12345678
        whiteRemoteAddress:
        admin: false
        defaultTopicPerm: DENY
        defaultGroupPerm: SUB
        topicPerms:
          - topicA=DENY
          - topicB=PUB|SUB
          - topicC=SUB
        groupPerms:
          # the group should convert to retry topic
          - groupA=DENY
          - groupB=PUB|SUB
          - groupC=SUB

      - accessKey: rocketmq2
        secretKey: 12345678
        # whiteRemoteAddress: *
        # if it is admin, it could access all resources
        admin: true
  config:
    ## brokerClusterName brokerName brokerRole brokerId 由内置脚本自动生成
    deleteWhen: "04"
    fileReservedTime: "48"
    flushDiskType: "ASYNC_FLUSH"
    namesrvAddr: "rocketmq-nameserver:9876"
    brokerIP1: "192.168.101.70"
    autoCreateSubscriptionGroup: "true"
    autoCreateTopicEnable: "true"
    listenPort: "31711"
    fastListenPort: "31709"
    haListenPort: "31712"
    diskSpaceWarningLevelRatio: "0.95"
    waitTimeMillsInSendQueue: "1000"
    aclEnable: "true"
nameserver:
  resources:
    limits:
      memory: 4Gi
  persistence:
    enabled: true
    size: 10Gi
    storageClass: "csi-disk"
proxy:
  resources:
    limits:
      cpu: 1
      memory: 2Gi
dashboard:
  enabled: true
  replicaCount: 1
  image:
    repository: "apacherocketmq/rocketmq-dashboard"
    tag: "1.0.0"
  ingress:
    enabled: true
    className: "nginx"
    annotations:
        nginx.ingress.kubernetes.io/whitelist-source-range: 171.221.174.15,154.206.15.221,190.92.202.241
    hosts:
      - host: rq2.cmzhu.cn
```

执行部署步骤，部署完成了

```bash
$ helm \
upgrade --install \
-n rocketmq --create-namespace \
rocketmq \
rocketmq-repo/rocketmq -f values.yaml 

```

如果要修改broker 的暴露地址，需要为broker 新加一个svc，并用nodePort 方式暴露公网；

```yaml
apiVersion: v1
kind: Service
metadata:
  name: rocketmq-broker-master-svc-0
spec:
  type: NodePort
  ports:
    - name: vip
      nodePort: 31709
      port: 31709
      targetPort: 31709
      protocol: TCP
    - name: main
      nodePort: 31711
      port: 31711
      targetPort: 31711
      protocol: TCP
    - name: ha
      nodePort: 31712
      port: 31712
      targetPort: 31712
      protocol: TCP

  selector:
      app.kubernetes.io/name: rocketmq
      broker: rocketmq-broker-master
      component: broker
      statefulset.kubernetes.io/pod-name: rocketmq-broker-master-0
```

并且开通公网访问后，broker 健康检查才会正常启动运行

这种情况下，proxy 会启动报错，为解决问题，在proxy 服务的启动参数中，添加ak/sk

```bash
$ kubectl -n rocketmq edit cm rocketmq-server-config 
```

修改如下位置配置

```yaml
  proxy.json: |
    {
      "rocketMQClusterName": "rocketmq-helm",
      "authenticationEnabled": true,
      "authenticationProvider": "org.apache.rocketmq.auth.authentication.provider.DefaultAuthenticationProvider",
      "authenticationMetadataProvider": "org.apache.rocketmq.proxy.auth.ProxyAuthenticationMetadataProvider",
      "innerClientAuthenticationCredentials": "{\"accessKey\":\"rocketmq2\", \"secretKey\":\"12345678\"}",
      "authorizationEnabled": true,
      "authorizationProvider": "org.apache.rocketmq.auth.authorization.provider.DefaultAuthorizationProvider",
      "authorizationMetadataProvider": "org.apache.rocketmq.proxy.auth.ProxyAuthorizationMetadataProvider"
    }
```







