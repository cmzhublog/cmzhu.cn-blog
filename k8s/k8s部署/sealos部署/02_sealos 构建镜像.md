## sealos 构建ingress 的镜像

下面展示如何使用helm 构建一个sealos 可用的ingress-nginx 的镜像

### 下载helm Charts

```bash
$ mkdir ingress-nginx && cd ingress-nginx
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm pull ingress-nginx/ingress-nginx
```

随后就能找到下载的 chart：

```shell
$ ls
ingress-nginx-4.8.2.tgz
```

#### 添加镜像列表

解压helm charts ， 然后查看ingress-nginx 的values 文件， 获得所有需要的镜像；

```bash
 $ tar -zxvf ingress-nginx-4.8.2.tgz
 $ cat ingress-nginx/values.yaml  
```

目录必须形如 `images/shim/[your image list filename]`：

```bash
$ cat images/shim/nginxImages

registry.k8s.io/ingress-nginx/controller:v1.9.3
registry.k8s.io/ingress-nginx/opentelemetry:v20230721
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
registry.k8s.io/defaultbackend-amd64:1.5

```

### 编写Dockerfile

```dockerfile
FROM scratch
COPY ../ingress-nginx .
CMD ["helm upgrade --install  ingress-nginx  --set controller.hostNetwork=true --set controller.dnsPolicy=ClusterFirstWithHostNet   ingress-nginx-4.8.2.tgz --namespace ingress-nginx --create-namespace"]
```

### 构建集群镜像

```bash
$ sealos build -f Dockerfile -t dockerhub.cmzhu.cn:5000/k8s/ingress-nginx:v1.2.0 .
```

Seals 在构建镜像时会自动将前面镜像列表中定义的镜像, 添加到镜像依赖中, 通过神奇的方式保存了里面依赖的Docker 镜像;并且在到别的环境中运行的时候更神奇的自动检测集群中是否有 Docker 镜像，有的话自动下载，没有的话才会去 k8s.gcr.io 下载。 用户无需修改 helm chart 中的 docker 镜像地址，这里用到了镜像缓存代理的黑科技。

```bash
$ sealos images
REPOSITORY                                             TAG       IMAGE ID       CREATED              SIZE
dockerhub.cmzhu.cn:5000/k8s/ingress-nginx              v1.2.0    976cba7a3f39   About a minute ago   135 MB

$ sealos push dockerhub.cmzhu.cn:5000/k8s/ingress-nginx:v1.2.0
Getting image source signatures
Copying blob a36012ff20cf done
Copying config 976cba7a3f done
Writing manifest to image destination
Storing signatures
```

### 运行集群镜像

```bash
$ sealos run dockerhub.cmzhu.cn:5000/k8s/ingress-nginx:v1.2.0
```

