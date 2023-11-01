## yum 

yum 是 centos 主流的包管理工具, 但后续Rocky Linux 改用了dnf 用于替代yum

直接通过yum 下载安装包

```bash
$ yum install --downloadonly --downloaddir=/tmp/ sysstat
```



完成之后如何离线安装呢? localinstall 命令会协助安装包时所需要的依赖

```bash
$ yum localinstall -y *.rpm
```

### reposync

该命令更加强大，可以将远端yum仓库里面的包全部下载到本地。这样构建自己的yum仓库，就不会遇到网络经常更新包而头痛的事情了。 该命令也是来自与 yum-utils 里面。

```bash
$ yum install yum-utils -y
```

参数: 

```bash
-r    指定已经本地已经配置的 yum 仓库的 repo源的名称。
-p    指定下载的路径
```

例子: 

```bash
$ reposync -r epel -p /opt/local_epel
```

