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