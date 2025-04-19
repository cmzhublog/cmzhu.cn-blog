## nerdctl 工具

参考仓库: [https://github.com/containerd/nerdctl](https://github.com/containerd/nerdctl)

### 背景

- kubernetes 升级成1.25.0 之后, runc 换成了containerd 

- nerdctl 就能像docker 一样来管理containerd

### 安装和部署

1、 从github 下载指定版本的nerdctl

```bash
 $ wget https://github.com/containerd/nerdctl/releases/download/v2.0.0-beta.2/nerdctl-2.0.0-beta.2-linux-amd64.tar.gz

```

2、解压缩包

```bash
 $ tar -zxvf nerdctl-2.0.0-beta.2-linux-amd64.tar.gz
nerdctl
containerd-rootless-setuptool.sh
containerd-rootless.sh
```

3、拷贝

```bash
$ mv nerdctl /usr/local/bin/
```

4、配置nerdctl 参考 [https://github.com/containerd/nerdctl/blob/main/docs/config.md](https://github.com/containerd/nerdctl/blob/main/docs/config.md)

```bash
 $ vim /etc/nerdctl/nerdctl.toml
 
 namespace      = "k8s.io"
```

5、 其余使用方式和docker 类似

