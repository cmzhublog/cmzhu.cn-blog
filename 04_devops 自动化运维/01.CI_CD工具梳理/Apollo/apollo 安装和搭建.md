## Apoolo 安装和搭建

### 什么是Apolo?

 文档参照: [https://github.com/apolloconfig/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D#1what-is-apollo](https://github.com/apolloconfig/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D#1what-is-apollo)



### Apollo 部署步骤

[参考文档](https://github.com/apolloconfig/apollo-helm-chart/tree/main)

本文介绍使用helm 部署apollo，在部署之前需要提前初始化数据库

#### 初始化数据库

1、 初始化数据库用户,创建新的数据库(通过下术命令，可以创建apolloconfigdb和apolloportaldb 数据库，并将apollo 账户赋予数据库管理权限)

```sql
$ CREATE USER 'apollo'@'%' IDENTIFIED BY '123456';
$ create database apolloconfigdb  DEFAULT CHARACTER SET utf8mb4 ;
$ GRANT ALL PRIVILEGES ON apolloconfigdb.* TO 'apollo'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

$ create database apolloportaldb  DEFAULT CHARACTER SET utf8mb4 ;
$ GRANT ALL PRIVILEGES ON apolloportaldb.* TO 'apollo'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```

2、数据库初始化，注意要分别对两个不同的数据库进行初始化[初始化脚本位置](https://github.com/apolloconfig/apollo-quick-start/tree/master/sql)

```bash
$ mysql -uapollo -p123456 
## 初始化 apolloconfigdb
$ use apolloconfigdb;
$ source apolloconfigdb.sql
## 初始化 apolloportaldb
$ use apolloportaldb
$ source apolloportaldb.sql
```

####  安装apollo

安装apollo 需要安装两个服务（apollo-portal和apollo-service），安装步骤如下：

依赖：

- Kubernetes 1.10+
- Helm 3

1、 helm 导入仓库,

```bash
$ helm repo add apollo https://charts.apolloconfig.com
$ helm search repo apollo
```

2、 安装apollo-service 服务

```bash
$ helm upgrade --install apollo-service \
    -n db --create-namespace \
    --set configdb.host=127.0.0.1 \
    --set configdb.userName=apollo \
    --set configdb.dbName=apolloconfigdb \
    --set configdb.password=123456 \
    --set configdb.service.enabled=true \
    --set configService.replicaCount=1 \
    --set adminService.replicaCount=1 \
    apollo/apollo-service
```

3、安装  apollo-portal 服务

```bash
$ helm install apollo-portal \
    --set portaldb.host=127.0.0.1 \
    --set portaldb.userName=apollo \
    --set portaldb.dbName=apolloportaldb \
    --set portaldb.password=123456 \
    --set portaldb.service.enabled=true \
    --set config.envs="dev\,pro" \
    --set config.metaServers.dev=http://apollo-service-apollo-configservice:8080 \
    --set config.metaServers.pro=http://apollo-service-apollo-configservice:8080 \
    --set replicaCount=1 \
    -n db --create-namespace \
    apollo/apollo-portal
```

4、安装完成后，可以看到三个容器正常运行,这样三个服务就创建完成了

```bash
$ k get pod -n db
NAME                                                   READY   STATUS      RESTARTS       AGE
apollo-portal-56778574f5-j5dnt                         1/1     Running     0              167m
apollo-service-apollo-adminservice-dd7d777b8-kl2kd     1/1     Running     4 (173m ago)   176m
apollo-service-apollo-configservice-679fbdbb5b-4wv5d   1/1     Running     3 (173m ago)   176m

```

5、安装完成后，可以有如下的svc ,可将 apollo-service-apollo-configservice 服务提供给后端代码服务，用于读取apollo 配置

```bash
$ k get svc -n db 
NAME                                  TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                                                       AGE
apollo-portal                         ClusterIP   10.96.1.247   <none>        8070/TCP                                                      168m
apollo-portal-apollo-portaldb         ClusterIP   10.96.1.248   <none>        3306/TCP                                                      168m
apollo-service-apollo-adminservice    ClusterIP   10.96.0.168   <none>        8090/TCP                                                      176m
apollo-service-apollo-configdb        ClusterIP   10.96.0.167   <none>        3306/TCP                                                      176m
apollo-service-apollo-configservice   ClusterIP   10.96.1.83    <none>        8080/TCP
```

6、为外部成员提供界面管理apollo,使用apollo.cmzhu.cn 进行访问

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apollo-ingress
  namespace: db
spec:
  ingressClassName: nginx
  rules:
    - host: apollo.cmzhu.cn
      http:
        paths:
          - pathType: Prefix
            backend:
              service:
                name: apollo-portal
                port:
                  number: 8070
            path: /
```

7、 默认管理员密码为: apollo/admin

 