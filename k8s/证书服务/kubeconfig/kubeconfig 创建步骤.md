## kubeconfig 创建步骤



如果～/.kube/config 权限太高， 不方便暴露出去，可以使用如下方案创建一个新的kubeconfig

### 创建用户

为 Kubernetes 创建一个新用户涉及到生成证书、创建 `kubeconfig` 文件，并为该用户配置适当的权限。以下是一个简单的示例，展示如何为 Kubernetes 创建一个新用户`test`，并给予其适当的权限。

1. **生成私钥和公钥（证书签名请求）**：
   生成新用户的私钥 (`test.key`) 和证书签名请求 (`test.csr`)：

```bash
$ openssl genrsa -out test.key 2048
$ openssl req -new -key test.key -out test.csr -subj "/CN=test"
## 在上面的命令中，/CN=newuser 指的是新用户的通用名称 (Common Name)，它将被用作 Kubernetes 用户名。
```

2. 获取 Kubernetes 集群的 CA 证书和私钥,通常从`/etc/kubernetes/pki/ `目录中拷贝出来

```bash
$ cp /etc/kubernetes/pki/ca.crt .
$ cp /etc/kubernetes/pki/ca.key .
```

3. 使用ca 证书， 生成test用户的csr 证书

```bash
$ openssl x509 -req -in test.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out test.crt -days 500
```

4、创建kubeconfig 文件

```bash
$ kubectl config set-credentials test --client-certificate=test.crt --client-key=test.key --embed-certs=true
$ kubectl config set-context test-context --cluster=cluster-name --namespace=default --user=test
```

5. 为新test 用户赋权

```yaml
# Role.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

# RoleBinding.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: test
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

6. 生成kubeconfig 文件

```bash
$ kubeadm kubeconfig user --client-name=test > test-kubeconfig.yaml
```

7. 使用kubeconfig 文件访问测试

```bash
$ kubectl get pod --kubeconfig=test-kubeconfig.yaml -A


```

