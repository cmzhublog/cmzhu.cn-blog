## rancher 的安装和部署

### 基础配置

- k8s: v1.25.0

### 安装步骤

本次安装步骤使用helm 来安装rancher 的服务

1、 先在本地导入对应的helm-charts 的repos 源

```bash
 $ helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
"rancher-latest" already exists with the same configuration, skipping
```

2、 将源拉取下来, 并修改values 的配置

```bash
$ helm pull --untar rancher-latest/rancher --version 2.8.2
$ cp rancher/values.yaml .
```

3、直接部署rancher 管理界面

```bash
 $ cat install.sh
helm upgrade --install \
  rancher rancher-latest/rancher \
  --version 2.8.2 \
  --namespace cattle-system  --create-namespace \
  --set hostname=rancher.cmzhu.cn \
  --set replicas=1 \
  --set bootstrapPassword=abc123!
```



参考文档 : [https://ranchermanager.docs.rancher.com/getting-started/quick-start-guides/deploy-rancher-manager/helm-cli](https://ranchermanager.docs.rancher.com/getting-started/quick-start-guides/deploy-rancher-manager/helm-cli)

 