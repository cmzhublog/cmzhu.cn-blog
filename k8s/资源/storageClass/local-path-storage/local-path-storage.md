## local-path-storage 配置

此配置为每一个k8s 的节点提供本地存储的方法,根据用户配置在节点上创建hostPath 或者local 的永久存储卷, 使其成为k8s 的StorageClass 的形式



优势:

- 无法对本地卷进行配置
- 使用Node 的本地卷

劣势:

- 不支持卷容量限制

github: https://github.com/rancher/local-path-provisioner

### 安装

- Stable 稳定

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

```

- Development 开发版本

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```



文件例子:

```yaml
root@cmzhu: # cat local-path-storage.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: local-path-storage

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: local-path-provisioner-service-account
  namespace: local-path-storage

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: local-path-provisioner-role
rules:
  - apiGroups: [ "" ]
    resources: [ "nodes", "persistentvolumeclaims", "configmaps" ]
    verbs: [ "get", "list", "watch" ]
  - apiGroups: [ "" ]
    resources: [ "endpoints", "persistentvolumes", "pods" ]
    verbs: [ "*" ]
  - apiGroups: [ "" ]
    resources: [ "events" ]
    verbs: [ "create", "patch" ]
  - apiGroups: [ "storage.k8s.io" ]
    resources: [ "storageclasses" ]
    verbs: [ "get", "list", "watch" ]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: local-path-provisioner-bind
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: local-path-provisioner-role
subjects:
  - kind: ServiceAccount
    name: local-path-provisioner-service-account
    namespace: local-path-storage

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: local-path-provisioner
  namespace: local-path-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: local-path-provisioner
  template:
    metadata:
      labels:
        app: local-path-provisioner
    spec:
      serviceAccountName: local-path-provisioner-service-account
      containers:
        - name: local-path-provisioner
          image: dockerhub.cmzhu.cn:5000/docker.io/rancher/local-path-provisioner:v0.0.24
          imagePullPolicy: IfNotPresent
          command:
            - local-path-provisioner
            - --debug
            - start
            - --config
            - /etc/config/config.json
          volumeMounts:
            - name: config-volume
              mountPath: /etc/config/
          env:
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      volumes:
        - name: config-volume
          configMap:
            name: local-path-config

---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: local-path-config
  namespace: local-path-storage
data:
  config.json: |-
    {
            "nodePathMap":[
            {
                    "node":"DEFAULT_PATH_FOR_NON_LISTED_NODES",
                    "paths":["/mnt/fastdev/local-path-provisioner"]
            }
            ]
    }
  setup: |-
    #!/bin/sh
    set -eu
    mkdir -m 0777 -p "$VOL_DIR"
  teardown: |-
    #!/bin/sh
    set -eu
    rm -rf "$VOL_DIR"
  helperPod.yaml: |-
    apiVersion: v1
    kind: Pod
    metadata:
      name: helper-pod
    spec:
      containers:
      - name: helper-pod
        image: busybox
        imagePullPolicy: IfNotPresent


```

### 使用

StorageClass 创建

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ssd-local-path
provisioner: cluster.local/local-path-provisioner
parameters:
  nodePath: /data/ssd
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

使用StorageClass 创建pvc

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name:  pvc-sfs-auto-example
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: ssd-local-path 
```

 