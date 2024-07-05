## 域控中添加samba 服务器

域控中提供合适的文件存储服务器

参考文档：[https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/assembly_setting-up-samba-as-an-ad-domain-member-server_assembly_using-samba-as-a-server](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/assembly_setting-up-samba-as-an-ad-domain-member-server_assembly_using-samba-as-a-server)

环境需求

- OS : Rock Linux 9.3

- DNS： 域控服务器

功能支持

- 访问其他域成员上的域资源
- 对本地服务（如 `sshd`）验证域用户
- 托管在服务器上的共享目录和打印机，以充当文件和打印服务器



### 前置准备

1、 关闭防火墙

```bash
$ systemctl disable --now firewalld
```

2、 关闭selinux, 执行如下配置， 然后重启服务器

```bash
$ sed "s/SELINUX=*/SELINUX=disabled/g" /etc/selinux/config
```

3、 windows 关闭防火墙

### 配置步骤

Samba Winbind 是系统安全服务守护进程(SSSD)的替代品，用于将 Red Hat Enterprise Linux(RHEL)系统与 活动目录(AD)连接。您可以使用 `realmd` 将 RHEL 系统加入到 AD 域，来配置 Samba Winbind。

1、如果AD 需要弃用RC4 加密进行 Kerberos 验证，请在 RHEL 中启用对这些密码的支持：

```bash
$ update-crypto-policies --set DEFAULT:AD-SUPPORT
```

2、rocky 安装需要使用到的安装包

```bash
$  install realmd oddjob-mkhomedir oddjob samba-winbind-clients \
       samba-winbind samba-common-tools samba-winbind-krb5-locator -y 
```

3、要在域成员中共享目录或打印机，请安装`samba` 软件包：

```bash
$ dnf install -y install samba
```

4、 备份samba 的配置文件

```bash
$ mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

5、 修改本机的DNS为域控的DNS，必须要将DNS 指向域控才能将服务器加入域控

```tex
$ cat /etc/resolv.conf

nameserver [域控服务器地址]
```

6、加入域。例如，要加入名为`cmzhu.cn`的域：

```bash
$ realm join --membership-software=samba --client-software=winbind cmzhu.cn
```

执行上述命令后， 会自动做如下的步骤

使用上面的命令，`realm`工具会自动：

- 为`cmzhu.cn`域中的成员创建`/etc/samba/smb.conf`文件
- 将用于用户和组查找的`winbind`模块添加到`/etc/nsswitch.conf`文件中
- 更新`/etc/pam.d/`目录中的可插拔验证模块(PAM)配置文件
- 启动`winbind`服务，并使服务在系统引导时启动

另外，在`/etc/samba/smb.conf`文件中设置备用的 ID 映射后端或自定义 ID 映射设置。详情请参阅 [了解和配置 Samba ID 映射](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/assembly_understanding-and-configuring-samba-id-mapping_assembly_using-samba-as-a-server)。

6、验证winbind 服务是否正常运行

```bash
$ systemctl status winbind
```

7、针对实际情况修改samba配置并启动samba 服务器

```bash
$ cat /etc/samba/smb.conf
[global]
kerberos method = secrets and keytab
realm = cmzhu.cn
template homedir = /home/%U@%D
workgroup = cmzhu
template shell = /bin/bash
security = ads
idmap config cmzhu : range = 2000000-2999999
idmap config cmzhu : backend = rid
idmap config * : range = 10000-999999
idmap config * : backend = tdb
winbind use default domain = no
winbind refresh tickets = yes
winbind offline logon = yes
winbind enum groups = no
winbind enum users = no
log file = /var/log/samba/log.%m
log level = 1 auth:3

[homes]
 comment = Home Directories
 browseable = no
 writable = yes

#[share]
#        comment = Share Directories
#        path = /share
#        public = yes
#        valid users = %U
#        browseable = yes
#        writable = yes
#        create mask = 0664
#        directory mask = 0775
[share]
comment = samba home directory
path = /home/share
public = yes
browseable = yes
public = yes
read only = no
writable = yes
create mask = 0777
directory mask = 0777
available = yes
security = share


[share2]
comment = samba home directory
path = /home/share2
public = yes
browseable = yes
public = yes
read only = no
writable = yes
create mask = 0777
directory mask = 0777
available = yes
security = share
valid users = cmzhu\bc

```

启动samba 服务器

```bash
$ systemctl enable --now smb 
```

### 验证

1、 显示同步的所有账户

```bash
$ wbinfo -u
cmzhu\administrator
cmzhu\guest
cmzhu\krbtgt
cmzhu\bctest
cmzhu\ldapadmin
cmzhu\test1
```

2 、显示所有组

```bash
$ wbinfo -g
```

3、 windows 电脑上连接samba 尝试, 输入下面地址，输入提供的域控的用户名和密码， 可以实现共享

```powershell
$ \\[samba 地址]\share
```

