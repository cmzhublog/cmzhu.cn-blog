## Sealos

sealos 是一种k8s 应用管理和编排工具, 具有非常好的扩展性

### 工具安装和下载

- 可以通过以下方式来获取sealos 的版本列表

```bash
curl --silent "https://api.github.com/repos/labring/sealos/releases" | jq -r '.[].tag_name'
```

结果如下:

```bash
root@cmzhu:/data# curl --silent "https://api.github.com/repos/labring/sealos/releases" | jq -r '.[].tag_name'
v4.4.0-beta1
v4.3.5
v4.3.4
v4.4.0-alpha3
v4.4.0-alpha1
v4.3.3
v4.3.2
v4.3.1
v4.3.1-rc2
v4.3.1-rc1
v4.3.0
v4.3.0-rc1
v4.2.3
v4.2.2
v5.0.0-alpha1
v4.2.1
v4.2.1-rc6
v4.2.1-rc5
v4.2.1-rc4
v4.2.1-rc3
v4.2.1-rc1
v4.2.0
v4.2.0-alpha3
v4.2.0-alpha2
v4.2.0-alpha1
v4.1.7
v4.1.6
v4.1.5
v4.1.5-rc3
v4.1.5-rc2
root@cmzhu:/data#
```

>  其中:` v4.3.4 `表示稳定版 `v4.4.0-alpha3`,`v4.4.0-beta1` 表示预发版本

使用下面命令可以获取最新的稳定版本号

```bash
VERSION=`curl -s https://api.github.com/repos/labring/sealos/releases/latest | grep -oE '"tag_name": "[^"]+"' | head -n1 | cut -d'"' -f4`
```

然后开始下载二进制,并且安装

```bash
curl -sfL https://raw.githubusercontent.com/labring/sealos/${VERSION}/scripts/install.sh |
  sh -s ${VERSION} labring/sealos
```

也可以手动下载二进制安装

```bash
$ wget https://github.com/labring/sealos/releases/download/${VERSION}/sealos_${VERSION#v}_linux_amd64.tar.gz \
   && tar zxvf sealos_${VERSION#v}_linux_amd64.tar.gz sealos && chmod +x sealos && mv sealos /usr/bin
```

#### 包管理工具安装

##### deb(ubuntu/ debian)

```bash
echo "deb [trusted=yes] https://apt.fury.io/labring/ /" | sudo tee /etc/apt/sources.list.d/labring.list
sudo apt update
sudo apt install sealos
```

##### rpm(Centos/Rocky Linux)

```bash
sudo cat > /etc/yum.repos.d/labring.repo << EOF
[fury]
name=labring Yum Repo
baseurl=https://yum.fury.io/labring/
enabled=1
gpgcheck=0
EOF
sudo yum clean all
sudo yum install sealos
```

#### 源码安装

##### 前置依赖

1. `linux`
2. `git`
3. `golang` 1.20+
4. `libgpgme-dev libbtrfs-dev libdevmapper-dev`

如果在 `arm64` 环境下需要添加 `:arm64` 后缀。

##### 构建

```bash
# git clone the repo
git clone https://github.com/labring/sealos.git
# just make it
make build BINS=sealos
```

### 安装k8s 集群

#### 先决条件

sealos 是一个简单的 go 二进制文件，可以安装在大多数 Linux 操作系统中。

以下是一些基本的安装要求：

- 每个集群节点应该有不同的主机名。 主机名不要带下划线。
- 所有节点的时间同步。
- 在 Kubernetes 集群的第一个节点上运行`sealos run`命令，目前集群外的节点不支持集群安装。
- 建议使用干净的操作系统来创建集群。不要自己装 Docker。
- 支持大多数 Linux 发行版，例如：Ubuntu CentOS Rocky linux。
- 支持 [DockerHub](https://hub.docker.com/r/labring/kubernetes/tags) 中支持的 Kubernetes 版本。
- 支持使用 containerd 作为容器运行时。
- 在公有云上请使用私有 IP。

#### CPU 架构

目前支持 `amd64` 和 `arm64` 架构。

#### 单机安装 Kuberentes

提前将需要用到的镜像打包到本地仓库(远程仓库国内使用比较慢, 建议先将镜像同步回本地仓库)

```bash
docker pull labring/kubernetes:v1.25.0
docker pull labring/helm:v3.8.2
docker pull labring/calico:v3.24.1
```



```bash
sealos run labring/kubernetes:v1.25.0 labring/helm:v3.8.2 dockerhub.cmzhu.cn:5000/docker.io/labring/calico:v3.24.1 --single
```

#### 集群安装 Kuberentes

```bash
sealos run labring/kubernetes:v1.25.0 labring/helm:v3.8.2 labring/calico:v3.24.1 \
     --masters 192.168.64.2,192.168.64.22,192.168.64.20 \
     --nodes 192.168.64.21,192.168.64.19 -p [your-ssh-passwd]
```

