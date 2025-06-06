## k8s 批量操作所有Deployment

### 背景

- 需要将该命名空间下所有的deployment 添加一个本地时区挂载的需求,将主机的`/etc/localtime` 挂载进容器

### 解决脚本

1、 创建脚本 `update_deploymenmt_localtime.sh `

```bash
#!/bin/bash

# 检查参数
if [ $# -ne 1 ]; then
    echo "Usage: $0 <namespace>"
    exit 1
fi

NAMESPACE=$1

# 获取命名空间下所有deployment名称
DEPLOYMENTS=$(kubectl get deployments -n $NAMESPACE -o jsonpath='{.items[*].metadata.name}')

for DEPLOY in $DEPLOYMENTS; do
    echo "Processing deployment: $DEPLOY"
    
    # 获取当前deployment的yaml并添加volume和volumeMount
    kubectl get deployment $DEPLOY -n $NAMESPACE -o yaml | \
    yq eval '
        .spec.template.spec.volumes += [{"name": "localtime", "hostPath": {"path": "/etc/localtime", "type": "File"}}] |
        .spec.template.spec.containers[].volumeMounts += [{"name": "localtime", "mountPath": "/etc/localtime", "readOnly": true}]
    ' - | \
    kubectl apply -f -
    
    echo "Updated deployment $DEPLOY with localtime mount"
done
```



脚本使用

```bash
$  bash update_deploymenmt_localtime.sh <namespace>
```

