---
title: build
description: 
published: true
date: 2022-10-30T05:56:03.912Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T05:56:03.912Z
---

# docker build 命令

**docker build** 命令用于使用 Dockerfile 创建镜像。

## 语法
```
docker build [OPTIONS] PATH | URL | -
```

OPTIONS说明：

- --build-arg=[] :设置镜像创建时的变量；

- --cpu-shares :设置 cpu 使用权重；

- --cpu-period :限制 CPU CFS周期；

- --cpu-quota :限制 CPU CFS配额；

- --cpuset-cpus :指定使用的CPU id；

- --cpuset-mems :指定使用的内存 id；

- --disable-content-trust :忽略校验，默认开启；

- -f :指定要使用的Dockerfile路径；

- --force-rm :设置镜像过程中删除中间容器；

- --isolation :使用容器隔离技术；

- --label=[] :设置镜像使用的元数据；

- -m :设置内存最大值；

- --memory-swap :设置Swap的最大值为内存+swap，"-1"表示不限swap；

- --no-cache :创建镜像的过程不使用缓存；

- --pull :尝试去更新镜像的新版本；

- --quiet, -q :安静模式，成功后只输出镜像 ID；

- --rm :设置镜像成功后删除中间容器；

- --shm-size :设置/dev/shm的大小，默认值是64M；

- --ulimit :Ulimit配置。

- --squash :将 Dockerfile 中所有的操作压缩为一层。

- --tag, -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。

- --network: 默认 default。在构建期间设置RUN指令的网络模式

## 实例
使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1。
```
docker build -t runoob/ubuntu:v1 . 
```

使用URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像。
```
docker build github.com/creack/docker-firefox
```

也可以通过 -f Dockerfile 文件的位置：
```
$ docker build -f /path/to/a/Dockerfile .
```
在 Docker 守护进程执行 Dockerfile 中的指令前，首先会对 Dockerfile 进行语法检查，有语法错误时会返回：
```
$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```
