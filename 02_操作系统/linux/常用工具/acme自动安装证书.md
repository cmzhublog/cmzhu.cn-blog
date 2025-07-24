## acme 自动安装证书

免费证书获取可以通过阿里云测试证书，cloudflare 免费证书等当方式获取；
这里介绍如何使用acme 进行证书获取

1、安装 acme.sh 脚本,并设置默认的ca证书解析为letsencrypt

```bash
$ curl https://get.acme.sh | sh
## 设置默认的ca 是letsencrypt
$ /root/.acme.sh/acme.sh --set-default-ca --server letsencrypt
```

2、使用默认--issue 方式生成单域名证书

```bash

## 生成 cdn.cmzhu.cn和www.cdn.cmzhu.cn 的证书
$ /root/.acme.sh/acme.sh --issue --nginx -d cdn.cmzhu.cn -d www.cdn.cmzhu.cn
## 服务器安装证书
$ /root/.acme.sh/acme.sh --install-cert -d cdn.cmzhu.cn -d www.cdn.cmzhu.cn \
	--key-file /etc/nginx/ssl/cdn.cmzhu.cn/cdn.cmzhu.cn.key \
	--fullchain-file /etc/nginx/ssl/cdn.cmzhu.cn/cdn.cmzhu.cn.crt
$ /root/.acme.sh/acme.sh list
```



3、使用 DNS 认证方式 生成范解析域名证书并安装

```bash
### 申请证书
$ /root/.acme.sh/acme.sh --issue -d *.cmzhu.cn --dns \
--yes-I-know-dns-manual-mode-enough-go-ahead-please
### 刷新证书
$ /root/.acme.sh/acme.sh --renew -d *.cmzhu.cn  \
--yes-I-know-dns-manual-mode-enough-go-ahead-please 

### 安装证书
/root/.acme.sh/acme.sh --install-cert -d *.cmzhu.cn  \
    --key-file /etc/nginx/ssl/acme.cmzhu.cn/acme.cmzhu.cn.key \
    --fullchain-file /etc/nginx/ssl/acme.cmzhu.cn/acme.cmzhu.cn.pem
```

