---
title: skopeo
description: 
published: true
date: 2022-11-22T03:53:58.585Z
tags: k8s
editor: markdown
dateCreated: 2022-11-02T07:32:36.898Z
---

# skopeo

## Sloped 介绍

- `skopeo`是一个命令行实用程序，可对容器映像和映像存储库执行各种操作。
- `skopeo`不需要用户以 root 身份运行来执行大部分操作。
- `skopeo`不需要运行守护程序来执行其操作。
- `skopeo`可以使用[OCI 映像](https://github.com/opencontainers/image-spec)以及原始 Docker v2 映像。

Skopeo 使用 API V2 容器镜像注册表，例如[docker.io](https://docker.io/)和[quay.io](https://quay.io/)注册表、私有注册表、本地目录和本地 OCI 布局目录。Skopeo 可以执行的操作包括：

- 在各种存储机制之间复制图像。例如，您可以将图像从一个注册表复制到另一个注册表，而无需特权。
- 检查显示其属性（包括其层）的远程图像，而无需您将图像拉到主机。
- 从图像存储库中删除图像。
- 将外部映像存储库同步到内部注册表以进行气隙部署。
- 当存储库需要时，skopeo 可以传递适当的凭据和证书进行身份验证。

Skopeo 在以下图像和存储库类型上运行：

- container-storage:docker-reference 位于本地容器/存储映像存储中的映像。位置和图像存储都在 /etc/containers/storage.conf 中指定。（这是[Podman](https://podman.io/)、[CRI-O](https://cri-o.io/)、[Buildah](https://buildah.io/)和朋友们的后端）
- dir:path 将清单、层 tarball 和签名存储为单独文件的现有本地目录路径。这是一种非标准化格式，主要用于调试或非侵入式容器检查。
- docker://docker-reference 实现“Docker Registry HTTP API V2”的注册表中的图像。默认情况下，使用 中的授权状态，使用`$XDG_RUNTIME_DIR/containers/auth.json`设置`skopeo login`。
- docker-archive:path[:docker-reference] 图像存储在`docker save`-formatted 文件中。docker-reference 仅在创建此类文件时使用，并且不得包含摘要。
- docker-daemon:docker-reference 存储在 docker daemon 内部存储中的镜像 docker-reference。docker-reference 必须包含标签或摘要。或者，在读取图像时，格式也可以是 docker-daemon:algo:digest（图像 ID）。
- oci:path:tag 在路径中符合“Open Container Image Layout Specification”的目录中的图像标记。

### 安装和卸载

- 安装

```bash
alias skopeo='docker run -it --rm -v /etc/docker/daemon.json:/etc/docker/daemon.json -v /root/.docker/config.json:/root/.docker/config.json -v /etc/hosts:/etc/hosts --privileged=true dockerhub.cmzhu.cn:5000/aipaas-devops/3rdparty/quay.io/skopeo/stable:v1.9.2  --insecure-policy --src-tls-verify=false --dest-tls-verify=false '
```

这个指令可以停止https 访问，应用于普通的registry
```
--insecure-policy --src-tls-verify=false --dest-tls-verify=false
```
- 卸载

```bash
unalias skopeo
```

### Copy

注意： copy 不支持http 协议，故如果协议中出现http会执行失败。

```shell
skopeo copy docker://dockerhub.cmzhu.cn:5000/cmzhu/userbox:master_3928d3a_221102142419 docker://local.dockerhub.cmzhu.cn:5000/cmzhu/userbox:master_3928d3a_221102142419
```

报错为：

```shell
[root@control-master004 ~]# skopeo copy docker://dockerhub.cmzhu.cn:5000/cmzhu/userbox:master_3928d3a_221102142419 docker://local.dockerhub.cmzhu.cn:5000/cmzhu/userbox:master_3928d3a_221102142419


Getting image source signatures
FATA[0000] trying to reuse blob sha256:2d473b07cdd5f0912cd6f1a703352c82b512407db6b05b43f2553732b55df3bc at destination: pinging container registry local.dockerhub.cmzhu.cn:5000: Get "https://local.dockerhub.cmzhu.cn:5000/v2/": http: server gave HTTP response to HTTPS client
```

如果要直接复制支持oci 协议的镜像目录，也是可行，指令类似于

```shell
skopeo copy oci:busybox_ocilayout:latest dir:existingemptydirectory
```



### Delete

删除镜像，直接传递镜像名字就行

```shell
skopeo delete docker://localhost:5000/imagename:latest
```

### 同步注册表

```shell
skopeo sync --src docker --dest dir registry.example.com/busybox /media/usb
```

### 向注册表进行身份验证

- 登录

```shell
skopeo login --username USER myregistrydomain.com:5000
```

- 登出

```bash
skopeo logout myregistrydomain.com:5000
```

### 挂载docker 信息认证

主机上有登录docker， 将主机上的/root/.docker/config.json 挂载在skopeo 容器的指定位置，也能复刻主机上的登录指令。

```bash
-v /root/.docker/config.json:/root/.docker/config.json
```
| Command                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [skopeo-copy(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-copy.1.md) | Copy an image (manifest, filesystem layers, signatures) from one location to another. |
| [skopeo-delete(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-delete.1.md) | Mark the image-name for later deletion by the registry's garbage collector. |
| [skopeo-inspect(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-inspect.1.md) | Return low-level information about image-name in a registry. |
| [skopeo-list-tags(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-list-tags.1.md) | Return a list of tags for the transport-specific image repository. |
| [skopeo-login(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-login.1.md) | Login to a container registry.                               |
| [skopeo-logout(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-logout.1.md) | Logout of a container registry.                              |
| [skopeo-manifest-digest(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-manifest-digest.1.md) | Compute a manifest digest for a manifest-file and write it to standard output. |
| [skopeo-standalone-sign(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-standalone-sign.1.md) | Debugging tool - Publish and sign an image in one step.      |
| [skopeo-standalone-verify(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-standalone-verify.1.md) | Verify an image signature.                                   |
| [skopeo-sync(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-sync.1.md) | Synchronize images between registry repositories and local directories. |

