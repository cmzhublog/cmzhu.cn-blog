## Code-Server 安装部署

github仓库： [https://github.com/coder/code-server](https://github.com/coder/code-server)

> 本地使用helm 在k8s 部署code-server



### 部署方式

1、 从github 上拉取仓库

```bash
$ git clone https://github.com/coder/code-server.git
```

2、从仓库中找到对应helm chart 的位置[github地址](https://github.com/coder/code-server/tree/main/ci/helm-chart)

```bash
$ cd  code-server/ci/helm-chart/
```

3、 编辑对应的values 文件

```yaml
replicaCount: 1
image:
  repository: dockerhub.cmzhu.cn:5000/docker.io/codercom/code-server
  tag: '4.13.0.buildx.v1.0.1'
  pullPolicy: Always
imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""
hostnameOverride: ""
serviceAccount:
  create: true
  annotations: {}
  name: ""
podAnnotations: {}
podSecurityContext: {}
priorityClassName: ""
service:
  type: ClusterIP
  port: 8080
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod   
  hosts:
    - host: codeserver.cmzhu.cn
      paths: 
        - / 
  ingressClassName: ""
  tls: 
    - secretName: code-server
      hosts:
        - codeserver.cmzhu.cn
extraArgs: []
extraVars: []
volumePermissions:
  enabled: true
  securityContext:
    runAsUser: 0
securityContext:
  enabled: true
  fsGroup: 1000
  runAsUser: 1000
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi
nodeSelector: {}
tolerations: []
affinity: {}
persistence:
  enabled: true
  storageClass: "openebs-hostpath"
  accessMode: ReadWriteOnce
  size: 10Gi
  annotations: {}
lifecycle:
  enabled: false
extraContainers: |
extraInitContainers: |
extraSecretMounts: []
extraVolumeMounts: []
extraConfigmapMounts: []
extraPorts: []

```

4、使用helm 命令部署

```bash
$ helm \
  upgrade --install code-server \
  ci/helm-chart \
  -n code-server \
  --create-namespace
```

5、 安装完成之后可以通过命令查看密码

```bash
Release "code-server" has been upgraded. Happy Helming!
NAME: code-server
LAST DEPLOYED: Fri Mar 15 22:37:26 2024
NAMESPACE: code-server
STATUS: deployed
REVISION: 2
NOTES:
1. Get the application URL by running these commands:
  https://codeserver.cmzhu.cn/

Administrator credentials:

  Password: echo $(kubectl get secret --namespace code-server code-server -o jsonpath="{.data.password}" | base64 --decode)
```

