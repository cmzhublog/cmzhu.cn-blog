## Sealos 升级k8s 版本

### 背景和注意点

1. 升级只能一个版本一个版本升级比如 1.25.0 -> 1.26.0 这样升级 
2. 如果要1.25.0 升级到1.28.0 这样升级，需要从1.25.0-> 1.26.0->1.27.0 ->1.28.0 这样升级

### 升级步骤如下

1. 查看当前K8S 版本,发现是1.26.0 版本

```bash
$ sealos status |grep 1.26
  RegistryPullStatus: ok:Image is up to date for sha256:e6f1816883972d4be47bd48879a08919b96afcd344132622e4d444987919323c
  ImageShimPullStatus: ok:Image is up to date for sha256:e6f1816883972d4be47bd48879a08919b96afcd344132622e4d444987919323c
    - sealos.hub:5000/kube-apiserver:v1.26.0
    - sealos.hub:5000/kube-controller-manager:v1.26.0
    - sealos.hub:5000/kube-proxy:v1.26.0
    - sealos.hub:5000/kube-scheduler:v1.26.0
```

2. 下载和升级版本

```bash
$ sealos pull registry.cn-shanghai.aliyuncs.com/labring/kubernetes:v1.27.0

# 升级版本
$ sealos run registry.cn-shanghai.aliyuncs.com/labring/kubernetes:v1.26.0
```

3. 升级完检查版本 检查,检查pod 是否能正常启动 

```bash
$ kubectl get pod -A 
```

