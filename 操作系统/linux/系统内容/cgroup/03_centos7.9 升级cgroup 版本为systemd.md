## centos 7 升级cgroup 版本为 systemd

### 背景

- 内核版本

```bash
$ uname -r
3.10.0-1160.71.1.el7.x86_64
```

- 操作系统版本

```bash
$ cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
```

- cgroup banbe

```bash
$ stat -fc %T /sys/fs/cgroup/
tmpfs
```



### 升级步骤

因为支持cgroup 的system 需要使用 `>4.15` 的内核版本, 因此第一步应该升级内核

1、 升级内核

查看内核是安装elrepo

```bash
$ yum  --disablerepo="*"  --enablerepo="elrepo-kernel"  list  available
Loaded plugins: fastestmirror


Error getting repository data for elrepo-kernel, repository not found
```

升级安装包, 并安装elrepo

```bash
$ yum update
$ rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

或者:

```bash
$ yum install -y https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
$ rpm -Uvh https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
```

载入elrepo-kernel元数据

```bash
$ yum --disablerepo=\* --enablerepo=elrepo-kernel repolist
```

查看可用的rpm 包

```bash
$ yum --disablerepo=\* --enablerepo=elrepo-kernel list kernel*
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Could not retrieve mirrorlist http://mirrors.elrepo.org/mirrors-elrepo-kernel.el7 error was
14: curl#52 - "Empty reply from server"
 * elrepo-kernel: mirrors.coreix.net
Installed Packages
kernel.x86_64                                                                                                         3.10.0-1160.71.1.el7                                                                                       @anaconda
kernel.x86_64                                                                                                         3.10.0-1160.102.1.el7                                                                                      @updates
kernel-tools.x86_64                                                                                                   3.10.0-1160.102.1.el7                                                                                      @updates
kernel-tools-libs.x86_64                                                                                              3.10.0-1160.102.1.el7                                                                                      @updates
Available Packages
kernel-lt.x86_64                                                                                                      5.4.260-1.el7.elrepo                                                                                       elrepo-kernel
kernel-lt-devel.x86_64                                                                                                5.4.260-1.el7.elrepo                                                                                       elrepo-kernel
kernel-lt-doc.noarch                                                                                                  5.4.260-1.el7.elrepo                                                                                       elrepo-kernel
kernel-lt-headers.x86_64                                                                                              5.4.260-1.el7.elrepo                                                                                       elrepo-kernel
kernel-lt-tools.x86_64                                                                                                5.4.260-1.el7.elrepo                                                                                       elrepo-kernel
kernel-lt-tools-libs.x86_64                                                                                           5.4.260-1.el7.elrepo                                                                                       elrepo-kernel
kernel-lt-tools-libs-devel.x86_64                                                                                     5.4.260-1.el7.elrepo                                                                                       elrepo-kernel
kernel-ml.x86_64                                                                                                      6.6.1-1.el7.elrepo                                                                                         elrepo-kernel
kernel-ml-devel.x86_64                                                                                                6.6.1-1.el7.elrepo                                                                                         elrepo-kernel
kernel-ml-doc.noarch                                                                                                  6.6.1-1.el7.elrepo                                                                                         elrepo-kernel
kernel-ml-headers.x86_64                                                                                              6.6.1-1.el7.elrepo                                                                                         elrepo-kernel
kernel-ml-tools.x86_64                                                                                                6.6.1-1.el7.elrepo                                                                                         elrepo-kernel
kernel-ml-tools-libs.x86_64                                                                                           6.6.1-1.el7.elrepo                                                                                         elrepo-kernel
kernel-ml-tools-libs-devel.x86_64                                                                                     6.6.1-1.el7.elrepo                                                                                         elrepo-kernel
[root@localhost ~]# yum  list kernel*
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
```

或者

```bash
$ yum  --disablerepo="*"  --enablerepo="elrepo-kernel"  list  available
```

内核升级到最新版本

```bash
$ yum  --enablerepo=elrepo-kernel  install  -y  kernel-lt
```

查看目前可用的内核版本

```bash
$ awk -F\' '$1=="menuentry " {print i++ " : " $2}' /boot/grub2/grub.cfg
0 : CentOS Linux (5.4.260-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.102.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1160.71.1.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-38e1a36180b14a3a943f710e5b01db6f) 7 (Core)
```

安装辅助工具

```bash
$ yum install -y grub2-pc
```

设置默认的系统加载内核

```bash
$ grub2-set-default 0
```

修改内核文件

```bash
#编辑/etc/default/grub文件
$ vim /etc/default/grub 
设置   GRUB_DEFAULT=0   # 只需要修改这里即可

GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved  #---将这里的saved修改成0----
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto spectre_v2=retpoline rhgb quiet net.ifnames=0 console=tty0 console=ttyS0,115200n8 noibrs"
GRUB_DISABLE_RECOVERY="true"
```

生成新的配置文件

```bash
$ grub2-mkconfig -o /boot/grub2/grub.cfg
```

安装最新的内核工具包

```bash
$ yum --disablerepo=\* --enablerepo=elrepo-kernel install -y kernel-lt-tools.x86_64
```

2、调整系统的cgroup 版本

使用yum 进行升级

```bash
$ [root@localhost ~]# wget https://copr.fedorainfracloud.org/coprs/jsynacek/systemd-backports-for-centos-7/repo/epel-7/jsynacek-systemd-backports-for-centos-7-epel-7.repo -O /etc/yum.repos.d/jsynacek-systemd-centos-7.repo --no-check-certificate
 
[root@localhost ~]# yum update systemd
```

```bash
$ yum  update
$ yum list systemd
```

开启V2 版本

```bash
$ cat /etc/default/grub

```

编译 /etc/default/grub 文件，创建 GRUB_CMDLINE_LINUX 变量并添加 cgroup_no_v1=all systemd.unified_cgroup_hierarchy=1 参数，执行以下命令重建 grub 文件并重启服务器。

```bash
$ grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg

```

