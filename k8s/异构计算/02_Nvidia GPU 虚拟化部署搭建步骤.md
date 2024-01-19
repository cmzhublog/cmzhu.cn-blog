## NVIDIA GPU 虚拟化部署

### 基础环境

- Red Hat 7.6
- 3.10.0-957.el7.x86_64
- helm >=3.14 [helm下载地址](https://github.com/helm/helm/releases)

### 开始安装

#### 安装nvidia-container-toolkit

1、 安装nvidia-container-toolkit(因为nvidia驱动已经安装, 暂不安装驱动)

参照文档:[https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

```bash
## 将安装包导出成本地rpm 包
### 添加对应的下载源
$ curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | \
  sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
### 启动下载源
$ yum-config-manager --enable nvidia-container-toolkit-experimental
### 离线下载nvidia-container-toolkit 的安装包
$ yum install --downloadonly --downloaddir=/nvidia nvidia-container-toolkit
  
```

#### 安装GPU-Operator

安装时需要离线安装

文档参照: [https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html)

1、 下载helm 

 [helm下载地址](https://github.com/helm/helm/releases)

2、 下载对应的helm chart 

```bash
$ helm pull 
```







