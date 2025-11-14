## rocketmq dashboard 搭建步骤

[参考文档](https://github.com/apache/rocketmq-dashboard)

可以从[dockerhub]() 中查询相关的镜像信息，并使用最新的镜像(deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: rocketmq-dashboard
  name: rocketmq-dashboard
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rocketmq-dashboard
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: rocketmq-dashboard
    spec:
      containers:
      - env:
        - name: JAVA_OPTS
          value: -Drocketmq.namesrv.addr=127.0.0.1:9876
        image: apacherocketmq/rocketmq-dashboard:2.0.0
        name: rocketmq-dashboard
        ports:
        - containerPort: 8080
          protocol: TCP
        resources:
          limits:
            cpu: "1"
            memory: 2Gi
          requests:
            cpu: 100m
            memory: 100Mi
      dnsPolicy: ClusterFirst
```

