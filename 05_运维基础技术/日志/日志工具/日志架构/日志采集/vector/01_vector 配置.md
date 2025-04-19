### vector 配置

> 本文档表示将vector 收集到的kubernetes 的日志, 推送到elasticsearch



### 配置步骤

1、在vector 中配置对应的source, 表示数据来源于 kubernetes_logs; 

可参考: [https://vector.dev/docs/reference/configuration/sources/kubernetes_logs/](https://vector.dev/docs/reference/configuration/sources/kubernetes_logs/)

```yaml
    sources:
      kubernetes_logs:
        type: kubernetes_logs
```

2、 针对source 配置, 对源数据进行不同进行处理

可参考: [https://vector.dev/docs/reference/configuration/transforms/remap/](https://vector.dev/docs/reference/configuration/transforms/remap/)

```yaml
    transforms:
      kubernetes_logs_transforms:
        inputs:
          - kubernetes_logs
        source: |-
          .namespace = .kubernetes.pod_namespace
          .pod = .kubernetes.pod_name
          .container = .kubernetes.container_name
          del(.file)
          del(.kubernetes)
          del(.timestamp)
          del(.source_type)
          del(.stream)
        type: remap
```

3、修改对应的sinks ,用于将处理好的数据送到指定的es 中

可参考: [https://vector.dev/docs/reference/configuration/sinks/elasticsearch/](https://vector.dev/docs/reference/configuration/sinks/elasticsearch/)

```yaml
    sinks:
      kubernetes_log_sink:
        type: elasticsearch
        inputs:
          - kubernetes_logs_transforms
        compression: none
        endpoints:
          - http://10.96.1.199:9200
        mode: bulk
        bulk:
          action: index
          index: kubernetes-log-%Y.%m.%d
```



4、完整的vector 配置

```yaml
data_dir: /vector-data-dir
    api:
      enabled: true
      address: 127.0.0.1:8686
      playground: false
    sources:
      kubernetes_logs:
        type: kubernetes_logs
    transforms:
      kubernetes_logs_transforms:
        inputs:
          - kubernetes_logs
        source: |-
          .namespace = .kubernetes.pod_namespace
          .pod = .kubernetes.pod_name
          .container = .kubernetes.container_name
          del(.file)
          del(.kubernetes)
          del(.timestamp)
          del(.source_type)
          del(.stream)
        type: remap
    sinks:
      kubernetes_log_sink:
        type: elasticsearch
        inputs:
          - kubernetes_logs_transforms
        compression: none
        endpoints:
          - http://10.96.1.199:9200
        mode: bulk
        bulk:
          action: index
          index: kubernetes-log-%Y.%m.%d
```

