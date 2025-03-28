## acme 自动安装证书

免费证书获取可以通过阿里云测试证书，cloudflare 免费证书等当方式获取；
这里介绍如何使用acme 进行证书获取

```bash
## 下载acme 安装脚本并执行
$ curl https://get.acme.sh | sh
## 设置默认的ca 是letsencrypt
$ acme.sh --set-default-ca --server letsencrypt
## 生成 cdn.cmzhu.cn和www.cdn.cmzhu.cn 的证书
$ /root/.acme.sh/acme.sh --issue --nginx -d cdn.cmzhu.cn -d www.cdn.cmzhu.cn
## 服务器安装证书
$ /root/.acme.sh/acme.sh --install-cert -d cdn.sukredito.mx -d www.cdn.sukredito.mx \
	--key-file /etc/nginx/ssl/cdn.sukredito.mx/cdn.sukredito.mx.key \
	--fullchain-file /etc/nginx/ssl/cdn.sukredito.mx/cdn.sukredito.mx.crt
$ /root/.acme.sh/acme.sh list
```

