## sentry 安装和搭建（YAML）

[参考文档](https://hub.docker.com/_/sentry)

### 部署前检查

1、 检查是否存在可用的storageClass ，如果不存在，则需要手动创建pv 和pvc 或者改为hostPath 方式创建持久化

```bash
$  k get sc
NAME                 PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
csi-disk (default)   openebs.io/local   Delete          WaitForFirstConsumer   false                  53d
openebs-device       openebs.io/local   Delete          WaitForFirstConsumer   false                  53d
```

从结果中可以看到，可以使用csi-disk 的storageClass 来进行创建

### 部署步骤

使用yaml 文件进行部署，依赖redis， postgres 的服务,创建deployment.yaml 具体内容如下：

```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  labels:
    app: redis
spec:
  serviceName: redis-service
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7.0-alpine
        ports:
        - containerPort: 6379
          name: redis
        volumeMounts:
        - name: redis-data
          mountPath: /data
        command: ["redis-server"]
        args: ["--appendonly", "yes"]
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "csi-disk"  
      resources:
        requests:
          storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  clusterIP: None
  ports:
  - port: 6379
    name: redis
  selector:
    app: redis

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sentry-postgres
spec:
  serviceName: postgres-headless
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_USER
          value: "sentry"
        - name: POSTGRES_PASSWORD
          value: "qcHW4b1Tq1NqR3"
        - name: PGDATA
          value: "/var/lib/postgresql/data/pgdata"
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "csi-disk"
      resources:
        requests:
          storage: 10Gi

---
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None
  ports:
  - port: 5432
  selector:
    app: postgres

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sentry-pvc
spec:
  storageClassName: "csi-disk"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sentry
spec:
  selector:
    matchLabels:
      app: sentry 
  template:
    metadata:
      labels:
        app: sentry 
    spec:
      initContainers:
      - name: db-migration
        image: sentry:latest
        command: ["sentry", "upgrade", "--noinput"]
        env:
        - name: SENTRY_SECRET_KEY
          value: "ql=ejw2vr0bfft@3^1fh2g6yoi36s@wh)pktwz@pjfks7xfq)!"
        - name: SENTRY_POSTGRES_HOST
          value: "postgres-headless"
        - name: SENTRY_REDIS_HOST
          value: "redis-service"
        - name: SENTRY_DB_USER
          value: "sentry"
        - name: SENTRY_DB_PASSWORD
          value: "qcHW4b1Tq1NqR3"
        volumeMounts:
        - name: sentry-data
          mountPath: /var/lib/sentry/files
      - name: create-admin
        image: sentry:latest
        command: ['sh', '-c', 
          'if sentry exec -c "from sentry.models import User; print(User.objects.filter(email=\"$(SENTRY_ADMIN_EMAIL)\").exists())" | grep -q "False"; then
            echo "Creating admin user";
            sentry createuser --email "$(SENTRY_ADMIN_EMAIL)" --password "$(SENTRY_ADMIN_PASSWORD)" --superuser;
          else
            echo "Admin user already exists";
          fi']
        env:
        - name: SENTRY_SECRET_KEY
          value: "ql=ejw2vr0bfft@3^1fh2g6yoi36s@wh)pktwz@pjfks7xfq)!"
        - name: SENTRY_POSTGRES_HOST
          value: "postgres-headless"
        - name: SENTRY_REDIS_HOST
          value: "redis-service"
        - name: SENTRY_DB_USER
          value: "sentry"
        - name: SENTRY_DB_PASSWORD
          value: "qcHW4b1Tq1NqR3"
        - name: SENTRY_ADMIN_EMAIL
          value: "admin@luminova.ltd"
        - name: SENTRY_ADMIN_PASSWORD
          value: "Abcd1234!"
        volumeMounts:
        - name: sentry-data
          mountPath: /var/lib/sentry/files

      containers:
      - name: web
        image: sentry:latest
        command: ["sentry", "run", "web"]
        ports:
        - containerPort: 9000
        env:
        - name: SENTRY_SECRET_KEY
          value: "ql=ejw2vr0bfft@3^1fh2g6yoi36s@wh)pktwz@pjfks7xfq)!"
        - name: SENTRY_POSTGRES_HOST
          value: "postgres-headless"
        - name: SENTRY_REDIS_HOST
          value: "redis-service"
        - name: SENTRY_DB_USER
          value: "sentry"
        - name: SENTRY_DB_PASSWORD
          value: "qcHW4b1Tq1NqR3"
        - name: C_FORCE_ROOT
          value: 'true'
        volumeMounts:
        - name: sentry-data
          mountPath: /var/lib/sentry/files
      - name: worker
        image: sentry:latest
        command: ["sentry", "run", "worker"]
        env:
        - name: SENTRY_SECRET_KEY
          value: "ql=ejw2vr0bfft@3^1fh2g6yoi36s@wh)pktwz@pjfks7xfq)!"
        - name: SENTRY_POSTGRES_HOST
          value: "postgres-headless"
        - name: SENTRY_REDIS_HOST
          value: "redis-service"
        - name: SENTRY_DB_USER
          value: "sentry"
        - name: SENTRY_DB_PASSWORD
          value: "qcHW4b1Tq1NqR3"
        - name: C_FORCE_ROOT
          value: 'true'
        volumeMounts:
        - name: sentry-data
          mountPath: /var/lib/sentry/files
      - name: cron
        image: sentry:latest
        command: ["sentry", "run", "cron"]
        env:
        - name: SENTRY_SECRET_KEY
          value: "ql=ejw2vr0bfft@3^1fh2g6yoi36s@wh)pktwz@pjfks7xfq)!"
        - name: SENTRY_POSTGRES_HOST
          value: "postgres-headless"
        - name: SENTRY_REDIS_HOST
          value: "redis-service"
        - name: SENTRY_DB_USER
          value: "sentry"
        - name: SENTRY_DB_PASSWORD
          value: "qcHW4b1Tq1NqR3"
        - name: C_FORCE_ROOT
          value: 'true'
        volumeMounts:
        - name: sentry-data
          mountPath: /var/lib/sentry/files
      volumes:
      - name: sentry-data
        persistentVolumeClaim:
          claimName: sentry-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: sentry-service
spec:
  selector:
    app: sentry
  ports:
    - protocol: TCP
      port: 9000
      targetPort: 9000
  type: ClusterIP

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sentry-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: sentry.cmzhu.cn
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: sentry-service
            port:
              number: 9000
```

2、根据实际情况修改storageClassName 变量的值，然后开始发布

```bash
$ kubectl apply -f deployment.yaml
```

3、执行完成后检查配置,全部运行即可以安装完成

```bash
$  k get pod
NAME                      READY   STATUS    RESTARTS   AGE
redis-0                   1/1     Running   0          46m
sentry-5f5b6456f7-g957w   3/3     Running   0          8m21s
sentry-postgres-0         1/1     Running   0          46m
```



