## Prometheus-Operator

Prometheus-Operator 是 k8s 里面Prometheus 的应用方案

github 地址: https://github.com/prometheus-operator/prometheus-operator

提供Prometheus 和相关监控组件的Kubernetes 的原生部署和管理;

### 安装

#### Helm 

github 地址: https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

##### 版本要求

- Kubernetes 1.19+
- Helm 3+

##### Helm 部署

1、安装helm 仓库

```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo update
```

2、 使用helm charts 安装 prometheus

```bash
$ helm install [RELEASE_NAME] prometheus-community/kube-prometheus-stack
```

3、 查看 helm-charts 相应的变量配置, 可以通过下面命令查询;

```bash
$ helm show values prometheus-community/kube-prometheus-stack
```

也可以将命令重定向到文件values.yaml 中, 调整变量后, 可以按照重新配置, 在安装时可以通过-f 命令指定values.yaml 文件来安装

```bash
$ helm install [RELEASE_NAME] prometheus-community/kube-prometheus-stack -f ./values.yaml
```

4、 完整的安装命令如下

```bash
$ helm \
  upgrade --install \
  -n monitoring --create-namespace \
  kube-prometheus-stack \
  prometheus-community/kube-prometheus-stack \
  -f ./values.yaml
```

5、 安装完成之后如何查看是否安装成功

```bash
$ kubectl --namespace monitoring get pods -l "release=kube-prometheus-stack"
NAME                                                        READY   STATUS    RESTARTS   AGE
kube-prometheus-stack-kube-state-metrics-6ddb9b4d48-lz47v   1/1     Running   0          33s
kube-prometheus-stack-operator-746454b794-59c2r             1/1     Running   0          33s
kube-prometheus-stack-prometheus-node-exporter-2qkkf        1/1     Running   0          33s
```

