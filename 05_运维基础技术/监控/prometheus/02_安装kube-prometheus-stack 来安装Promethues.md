## Kube-Prometheus-Stack

[参考文档](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)

1、 将仓库加入到本地helm 源。

```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
```

2、根据实际情况创建values.yaml 文件

```bash
$ cat > values.yaml << EOF
prometheus:
  prometheusSpec:
    storageSpec:  # 关键配置项
      volumeClaimTemplate:
        spec:
          storageClassName: "csi-disk"  # 替换为实际存储类名
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi
grafana:
  enabled: false
nodeExporter:
  enabled: false
kubeStateMetrics:
  enabled: false

EOF
```

3、部署 kube-prometheus-stack

```bash
$ helm upgrade --install -n monitor --create-namespace kube-prometheus-stack prometheus-community/kube-prometheus-stack -f ./values.yaml
```

4、部署完成后，会自带ServiceMonitor等CRD

 
