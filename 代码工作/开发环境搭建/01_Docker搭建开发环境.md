## Docker 搭建开发环境

Docker 搭建开发环境， 是使用Docker 搭建开发环境的镜像， 用于针对Golang、 C 、Python 等编程语言的开发； 本节以搭建golang 开发环境为例子， 来记录如何搭建开发环境的步骤

### 搭建步骤

1、 确定基础镜像

- 基础镜像： rockylinux:9.2.20230513-minimal

```dockerfile
FROM rockylinux:9.2.20230513 as baseimg

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

2、build 成镜像

```bash
$ docker build -t dockerhub.cmzhu.cn:5000/infrastructure/golang:v1.14.1 -f Dockerfile .
```

3、测试环境

```bash
$ docker run -it --rm  dockerhub.cmzhu.cn:5000/infrastructure/golang:v1.14.1 bash
```

