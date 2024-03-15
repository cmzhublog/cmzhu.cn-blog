## console 配置解决

使用 Kubernetes 自带的Dashboard 进行安装

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

$ k apply -f recommended.yaml
Warning: resource namespaces/kubernetes-dashboard is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
namespace/kubernetes-dashboard configured
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

默认会在 `kubernetes-dashboard` 命名空间下 安装一个 Dashboard 服务

部署完成之后

```bash
$  k get pod -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-64bcc67c9c-spfx4   1/1     Running   0          2m53s
kubernetes-dashboard-5c8bd6b59-992s5         1/1     Running   0          2m53s
```

配置ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: console
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: HTTPS # 指定访问端服务的方式为 HTTPS, 否则会报错400
spec:
  ingressClassName: nginx
  rules:
  - host: console.cmzhu.cn
    http:
      paths:
      - backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 443
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - console.cmzhu.cn
    secretName: movietls
```

### 创建用户登录的beartoken

1、 创建权限ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

2、 创建角色绑定

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

3、 指定用户创建beartoken

```bash
$ kubectl -n kubernetes-dashboard create token admin-user

eyJhbGciOiJSUzI1NiIsImtpZC ...... pc6rp77E9yX3TGvvWVjSbYVKvARPAAezPe-9qHemdbWqvWoauTPlS2PSbc56bL1e0L4ZGBQ4KW5PAhQN9jODJA
```

4、 为指定用户创建一个永久的 Beartoken

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token 
```

5、 查看beartoken

```bash
$ kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d

eyJhbGciOiJSUzI1 ...... 60nkj7YGAKvw
```



6、如果觉得自己测试管理, 每次都需要输入beartoken 很复杂, 可以将beartoken 的访问方式关闭, 对应修改配置

```yaml
## cat recommended.yaml

...
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      name: https
      ## service上添加 非安全的访问接口
    - port: 80
      targetPort: 9090
      name: http
  selector:
    k8s-app: kubernetes-dashboard
...

···

      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.7.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
              name: https
             ## 暴露出 9090 这一非安全的端口
            - containerPort: 9090
              protocol: TCP
              name: http
          args:
          ### 注释掉以下启动参数 --auto-generate-certificates
            # - --auto-generate-certificates
            - --namespace=kubernetes-dashboard

···
```

