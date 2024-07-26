### **2.2 安装buildx**

1、下载安装包并将buildx 拷贝到指定位置, 在此仓库中可以下载指定版本的文件

```
mkdir -p ~/.docker/cli-plugins
wget https://github.com/docker/buildx/releases/download/v0.16.2/buildx-v0.16.2.linux-amd64 -O ~/.docker/cli-plugins/docker-buildx && chmod +x ~/.docker/cli-plugins/docker-buildx
```

2、buildkit 的代理设置和docker 打包有明显不同

- 镜像仓库如果不可信，没办法直接拉取打包需要的镜像， 需要提供tls证书可信的镜像仓库；
- 最好不要使用docker.io 的镜像仓库/拉取镜像会比较复杂
- 内网镜像可以使用skopeo 将所有的版本全部同步到本地镜像仓库， 然后buildkit 打包时使用buildkit 来进行打包

```
$ skopeo sync --all --src docker --dest docker python:3.11-buster dockerhub.netfang.cn:5000/3rdpartyimages/
```

### **2.3 测试buildkit**

1、创建buildx 运行环境

```
#创建构建容器
docker buildx create --name mybuilder
#buildx使用构建容器
docker buildx use mybuilder
#初始化构建容器
docker buildx inspect --bootstrap
```

执行结束之后， 使用 `docker ps`可以看到启动了一个buildx 的运行容器编译镜像全在这一容器中进行

```
$ docker ps | grep buildx
1fa2ccdc2153   moby/buildkit:buildx-stable-1         "buildkitd --allow-i…"   42 hours ago   Up 23 hours                                                                                                                                 buildx_buildkit_mybuilder0
```

2、开始进行镜像编译， 这里准备一个Dockerfile

```
FROM dockerhub.testcmzhu.top:5000/3rdpartyimages/rockylinux:9.3.20231119


## 设置 代码根目录
RUN mkdir -p /data/workspace
WORKDIR /data/workspace

## 设置运行用户 为root
USER root

### 安装必要的插件
RUN \
  dnf -y groupinstall "Development Tools" \
  && dnf -y install epel-release \
  && dnf -y install htop \
  && dnf -y install git
RUN \
  dnf install openssh-server wget vim telnet -y

## 运行ssh 服务
RUN \
  ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key  -N "" \
  && chmod 600 /etc/ssh/ssh_host_*_key \
  && chown root:root /etc/ssh/ssh_host_*_key

RUN \
  /usr/sbin/sshd -D &

## 安装 golang
RUN \
  wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz \
  && tar -zxvf go1.14.1.linux-amd64.tar.gz -C /usr/local \
  && rm -f go1.14.1.linux-amd64.tar.gz


ENV GOROOT=/usr/local/go
ENV PATH=$PATH:$GOROOT/bin

CMD ["/usr/sbin/sshd", "-D"]
```

3、执行如下命令进行打包

```
$ DOCKER_BUILDKIT=1 docker buildx build \
-t dockerhub.testcmzhu.top:5000/devops/test:v1_2024072502 \
-t test \
--platform linux/amd64,linux/arm64 \
. \
--push
```

DOCKER_BUILDKIT=1: 表示使用buildkit 来进行打包， 如果还是按照原来的`docker build` 来打包

- -t : 指定镜像的标签， 可以重复指定多个

- --platform： 指定架构

- .： 指定目录

- -f : 指定Dockerfile 的地址， 如果没有就默认使用当前目录下的Dockerfile

- --push: 指将镜像推送到镜像仓库