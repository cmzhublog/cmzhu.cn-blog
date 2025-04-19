## scp

Linux scp 命令用于 Linux 之间复制文件和目录。

scp 是 secure copy 的缩写, scp 是 linux 系统下基于 ssh 登陆进行安全的远程文件拷贝命令。

scp 是加密的，[rcp](https://www.runoob.com/linux/linux-comm-rcp.html) 是不加密的，scp 是 rcp 的加强版。

### 语法

```bash
$ scp [-1246BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file]
[-l limit] [-o ssh_option] [-P port] [-S program]
[[user@]host1:]file1 [...] [[user@]host2:]file2
```

简易写法:

```bash
$ scp [可选参数] file_source file_target
```

### 参数说明

- -1： 强制scp命令使用协议ssh1
- -2： 强制scp命令使用协议ssh2
- -4： 强制scp命令只使用IPv4寻址
- -6： 强制scp命令只使用IPv6寻址
- -B： 使用批处理模式（传输过程中不询问传输口令或短语）
- -C： 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
- -p：保留原文件的修改时间，访问时间和访问权限。
- -q： 不显示传输进度条。
- -r： 递归复制整个目录。
- -v：详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
- -c cipher： 以cipher将数据传输进行加密，这个选项将直接传递给ssh。
- -F ssh_config： 指定一个替代的ssh配置文件，此参数直接传递给ssh。
- -i identity_file： 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
- -l limit： 限定用户所能使用的带宽，以Kbit/s为单位。
- -o ssh_option： 如果习惯于使用ssh_config(5)中的参数传递方式，
- -P port：注意是大写的P, port是指定数据传输用到的端口号
- -S program： 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。

### 例子

#### 1、从本地复制到远程

命令格式：

```bash
$ scp local_file remote_username@remote_ip:remote_folder 
或者 
$ scp local_file remote_username@remote_ip:remote_file 
或者 
$ scp local_file remote_ip:remote_folder 
或者 
$ scp local_file remote_ip:remote_file 
```

复制目录

```bash
$ scp -r local_folder remote_username@remote_ip:remote_folder
```

#### 2、从远程复制到本地

```bash
$ scp root@www.runoob.com:/home/root/others/music /home/space/music/1.mp3 
$ scp -r www.runoob.com:/home/root/others/ /home/space/music/
```

> **说明：**
>
> 1.如果远程服务器防火墙有为scp命令设置了指定的端口，我们需要使用 -P 参数来设置命令的端口号，命令格式如下：
>
> ```bash
> #scp 命令使用端口号 4588
> $ scp -P 4588 remote@www.runoob.com:/usr/local/sin.sh /home/administrator
> ```
>
> 2.使用scp命令要确保使用的用户具有可读取远程服务器相应文件的权限，否则scp命令是无法起作用的。



3、保留用户权限配置和时间

```bash
$ scp  -vp -r www.runoob.com:/home/root/others/ /home/space/music/
```

