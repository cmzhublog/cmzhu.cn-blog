## Harbor 存储 helm-chart

###  Harbor 支持让helm-chart存储在Harbor 仓库中

1、对于2.8之前的Harbor, 需要使用命令行开启chart 功能

```bash
$ ./prepare --with-chartmuseum
```

2、对于之后的版本, Harbor 弃用了chartmuseum功能, helm-chart 文件和镜像存储在相同的目录下, 没有新开页面

如何使用:

2.1 登陆Harbor 仓库, 根据提示输入用户名和密码即可

```bash
$ helm registry login dockerhub.testcmzhu.top:5000
```

2.2 将helm chart push 到仓库中, 注意存储在仓库的格式

```bash
$ helm push cilium-1.12.6.tgz oci://dockerhub.testcmzhu.top:5000/helm-chart/cilium
```

2.3 对于没有https 证书的非安全连接, 可以取消证书校验功能

```bash
$ helm push cilium-1.12.6.tgz oci://dockerhub.testcmzhu.top:5000/helm-chart/cilium --insecure-skip-tls-verify
```

