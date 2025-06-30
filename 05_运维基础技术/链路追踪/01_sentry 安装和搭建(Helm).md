## Sentry 安装和搭建(helm) 不推荐

[参考文档](https://github.com/sentry-kubernetes/charts)

### 安装步骤

1、 导入sentry 源以及以及更新sentry 配置

```bash
$ helm repo add sentry https://sentry-kubernetes.github.io/charts
$ helm repo update
```

2、将sentry 的values 文件拉取下来，并根据实际情况进行修改

```bash
$ helm show values sentry/sentry > values.yaml
```

3、根据实际情况调整sentry 配置，生成可用的values.yaml

```bash
$ cat > values.yaml << EOF
global:
  nodeSelector:
    sentry: 'true'
  tolerations:
  - key: "pre-worker"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
filestore:
  filesystem:
    persistence:
      storageClass: "openebs-hostpath"
geodata:
  persistence:
    storageClass: "openebs-hostpath"
EOF
```

4、使用 values.yaml 搭建sentry

```bash
$ helm upgrade --install -n devops --create-namespace sentry sentry/sentry --wait --timeout=1000s
```

