## nacos 集群化部署

[参考文档](https://github.com/nacos-group/nacos-k8s/blob/master/operator/README-CN.md?spm=55c5c5db.51f9ffce.0.0.2f6038c1DHMRAL&file=README-CN.md)

### 部署步骤

1、 从github 上clone [nacos-k8s](https://github.com/nacos-group/nacos-k8s/tree/master) 仓库

```bash
## 创建临时目录
$ cd $(mktemp -d )

$ git clone https://github.com/nacos-group/nacos-k8s.git && cd nacos-k8s/operator
```

2、 执行部署命令，用于创建operator 和crd 资源

```bash
# 直接使用helm方式安装operator
helm install nacos-operator ./chart/nacos-operator 

# 如果没有helm, 使用kubectl进行安装, 默认安装在default下面
kubectl apply -f chart/nacos-operator/nacos-operator-all.yaml
```

3、部署完成operator 之后，开始部署nacos

单节点

```bash
$ cat >> config/samples/nacos.yaml << EOF
apiVersion: nacos.io/v1alpha1
kind: Nacos
metadata:
  name: nacos
spec:
  type: standalone
  image: nacos/nacos-server:1.4.1
  replicas: 1

EOF

# 安装demo standalone模式
$ kubectl apply -f config/samples/nacos.yaml
```

多节点（集群模式）

```bash
cat >>  config/samples/nacos_cluster.yaml << EOF

apiVersion: nacos.io/v1alpha1
kind: Nacos
metadata:
  name: nacos
spec:
  type: cluster
  image: nacos/nacos-server:1.4.1
  replicas: 3
EOF
$ kubectl apply -f config/samples/nacos_cluster.yaml
```

### 全部CRD配置



全部参数如下

| 参数                         | 描述                                                     | 参考值                                                       |
| ---------------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| spec.type                    | 集群类型                                                 | 目前支持standalone 和 cluster                                |
| spec.image                   | 镜像地址，兼容社区镜像                                   | nacos/nacos-server:1.4.1                                     |
| spec.mysqlInitImage          | mysql数据初始镜像地址，mysql模式下将自动导入数据库       | registry.cn-hangzhou.aliyuncs.com/shenkonghui/mysql-client   |
| spec.replicas                | 实例数量                                                 | 1                                                            |
| spec.database.type           | 数据库类型                                               | 目前支持mysql和embedded                                      |
| spec.database.mysqlHost      | mysql连接地址                                            | 默认mysql                                                    |
| spec.database.mysqlPort      | mysql端口                                                | 默认3306                                                     |
| spec.database.mysqlUser      | mysql用户                                                | 默认root                                                     |
| spec.database.mysqlPassword  | mysql密码                                                | 默认123456                                                   |
| spec.database.mysqlDb        | mysq数据库                                               | 默认nacos                                                    |
| spec.volume.enabled          | 是否开启数据卷                                           | true，如果数据库类型是embedded，请开启数据卷，否则重启pod数据丢失 |
| spec.volume.requests.storage | 存储大小                                                 | 1Gi                                                          |
| spec.volume.storageClass     | 存储类                                                   | default                                                      |
| spec.config                  | 其他自定义配置，自动映射到custom.propretise              | 格式和configmap兼容                                          |
| spec.k8sWrapper              | 支持通用k8配置，即PodSpec对象，会自动覆盖所有内部pod对象 | 无                                                           |

#### 数据库配置

embedded数据库

```yaml
apiVersion: nacos.io/v1alpha1
kind: Nacos
metadata:
  name: nacos
spec:
  type: standalone
  image: nacos/nacos-server:1.4.1
  replicas: 1
  database:
    type: embedded
  # 启动数据卷，不然重启后数据丢失
  volume:
    enabled: true
    requests:
      storage: 1Gi
    storageClass: default
```



mysql数据库

该模式下需要提供外部mysql连接信息，会自动创建创建nacos数据库，并执行初始化sql

```yaml
apiVersion: nacos.io/v1alpha1
kind: Nacos
metadata:
  name: nacos
spec:
  type: standalone
  image: nacos/nacos-server:1.4.1
  replicas: 1
  database:
    type: mysql
    mysqlHost: mysql
    mysqlDb: nacos
    mysqlUser: root
    mysqlPort: "3306"
    mysqlPassword: "123456"
```

#### 自定义配置

通过环境变量配置 兼容nacos-docker项目， https://github.com/nacos-group/nacos-docker

```yaml
apiVersion: nacos.io/v1alpha1
kind: Nacos
metadata:
  name: nacos
spec:
  type: standalone
  env:
  - key: JVM_XMS
    value: 2g
```

