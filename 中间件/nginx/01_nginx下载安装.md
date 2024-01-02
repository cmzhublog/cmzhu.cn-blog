## nginx 安装方式

nginx 是支持大量请求的nginx 服务器, 其占用内存少、并发能力强、能支持高达 5w 个并发连接数, 深受开发者喜爱

### nginx 安装方式

#### Rech

1、 在安装nginx 之前, 需要安装nginx 所在的包存储仓库, Redhat , Rocky linux , Centos 类似

```bash
$ yum install yum-utils
```

2、 创建yum 仓库配置文件/etc/yum.repos.d/nginx.repo

```bash
$ cat /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

3、 一般情况下, 下载使用都是稳定版本的nginx 包, 如果想使用最新的开发版本, 可以执行下面命令

```bash
$ yum-config-manager --enable nginx-mainline
```

4、 然后开始安装nginx

```bash
$ yum install nginx
```

#### debian

1、 先安装依赖

```bash
$ apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring
```

2、导入官方签名的密钥, 用于apt 验证包的真实性

```bash
$ curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

3、 验证密钥的正确性

```bash
$ gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

4、 输出对应的密钥573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62

```bash
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```

5、 将稳定版仓库地址配置进apt 配置文件中/etc/apt/sources.list.d/nginx.list

```bash
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
  http://nginx.org/packages/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

6、配置最新main分支(开发)版本

```bash
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
  http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

7、 设置该nginx 配置为机器优先提供nginx 安装包

```bash
$ echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

8、 开始安装 nginx

```bash
$ sudo apt update
$ sudo apt install nginx
```

#### Ubuntu

1、安装依赖

```bash
$ apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

2、导入官方签名

```bash
$ curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

3、 验证官方密钥的正确性

```bash
$ gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```

4、 设置稳定版的包管理仓库

```bash
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
  http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

5、设置非稳定版(开发版)的包管理仓库(设置了4 就不用设置5)

```bash
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
  http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

6、 设置优先使用配置的包仓库

```bash
$ echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

7、 安装nginx

```bash
$ sudo apt update
$ sudo apt install nginx
```

###  nginx 编译安装

nginx 支持编译安装, 可以按照如下步骤进行安装, 安装所有需要的依赖, 和换国内下载源



1、 安装指定编译时需要的依赖

```bash
$ apt -y install  build-essential \
	zlib1g-dev \
	libpcre3-dev \
	libssl-dev \
	wget 
```

修改国内源

```bash
$ cat << EOF | sudo tee /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
EOF

$ apt-get update

```



2、 在官网查找对应的nginx 的版本, 以下按照nginx1.24.0 版本来安装

```bash
$ wget https://nginx.org/download/nginx-1.24.0.tar.gz
$ tar -zxvf  nginx-1.24.0.tar.gz && cd nginx-1.24.0
```

3、开始编译安装

```bash
$ ls
CHANGES  CHANGES.ru  LICENSE  README  auto  conf  configure  contrib  html  man  src
```

```bash
$ ./configure
...
...
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
  

$ make && make install 

```

