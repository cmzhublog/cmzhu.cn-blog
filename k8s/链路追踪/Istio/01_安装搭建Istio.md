## 安装部署Istio

### Helm 部署

文档参照: [https://istio.io/latest/docs/setup/install/helm/](https://istio.io/latest/docs/setup/install/helm/)

本文使用Helm 安装和Istioctl 和Operator 安装后的使用是一致的

### 前置准备

- 平台需要提前设置
- Pod 服务检查
- 安装高版本的Helm(>= 3.6)
- 配置helm 的仓库

```bash
$ helm repo add istio https://istio-release.storage.googleapis.com/charts
$ helm repo update
```

### 安装步骤

1、使用helm 安装的通用模板如下;

```bash
$ helm upgrade --install \
  <release> <chart> --namespace <namespace> --create-namespace [--set <other_parameters>]
```

变量用途

- release: 用于安装后的helm 名字
- chart: chart 的路径, 可以是本地目录, 也可以是地址url
- namespace: 安装部署的命名空间
- --set <other_parameters>: 用于修改默认变量

2、 使用CRD 的方式安装Istio Base Chart 

```bash
$ helm upgrade --install  istio-base istio/base -n istio-system --create-namespace --set defaultRevision=default
```

查询 helm chart 的状态,需要确保 status 是deployed

```bash
 $  helm ls -n istio-system
NAME      	NAMESPACE   	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
istio-base	istio-system	1       	2024-01-17 02:33:35.684605702 -0500 EST	deployed	base-1.20.2	1.20.2
```

3、如果需要使用到 Istio 的图表, 可以按照下面步骤操作

```bash
```

