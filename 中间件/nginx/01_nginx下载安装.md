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

