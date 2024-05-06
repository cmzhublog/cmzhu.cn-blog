## nginx 修改文件上传大小限制

### 背景

- 接口上传文件时会在传输到一定之后报错403 , body size is Too Large

### 解决思路

1、 修改全体链路的nginx 的配置, 如果不设置, 默认配置为1m, 在对应接口中加上该配置`client_max_body_size`

```nginx
Syntax:	client_max_body_size size;
Default:	client_max_body_size 1m;
Context:	http, server, location
```

2、 在ingress-nginx 中修改此参数, 在ingress 注解中添加以下即可

参考: [https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md](https://github.com/kubernetes/ingress-nginx/blob/main/docs/user-guide/nginx-configuration/annotations.md)

```bash
nginx.ingress.kubernetes.io/proxy-body-size: 8m
```

