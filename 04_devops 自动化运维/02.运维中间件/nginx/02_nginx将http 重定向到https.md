## `HTTP` 重定向成`HTTPS`

### 背景

在访问网站时， 用户不一定都会输入https 来访问， 这样如果使用http 访问https 得网站， 就会访问失败； 这时候，需要将HTTP请求 重定向成HTTPS的请求

### 配置方案

1、使用rewrite 方式将HTTP 重写成HTTPS

```nginx
server {
    listen 80;
    
    #填写绑定证书的域名
    server_name www.xxx.com;
    
    #强制将http的URL重写成https
    rewrite ^(.*) https://$server_name$1 permanent; 
}
```

2、使用301 重定向的方式将HTTP 指定到HTTPS 上

```nginx
server {
    listen 80;
    
    #填写绑定证书的域名
    server_name www.xxx.com;
    
    #把http的域名请求转成https
    return 301 https://$host$request_uri;
}
```

3、完整的nginx 配置如下

```nginx
#HTTP配置
server {
    listen 80;
    
    #填写绑定证书的域名
    server_name www.xxx.com;
    
    #（第一种）把http的域名请求转成https
    return 301 https://$host$request_uri;
    
    #（第二种）强制将http的URL重写成https
    rewrite ^(.*) https://$server_name$1 permanent; 
}

#HTTPS使用SSL访问的配置
server {
    #SSL使用443端口
    listen 443 ssl;
    
    #SSL证书绑定的单域名
    server_name www.xxx.com;
    
    #证书pem文件
    ssl_certificate /usr/cert/www_xxx_com.pem;
    
    #证书key文件
    ssl_certificate_key /usr/cert/www_xxx_com.key;
    
    #缓存SSL握手产生的参数和加密密钥的时长
    ssl_session_timeout 5m;
    
    #使用的加密套件的类型
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; 
    
    #表示使用的TLS协议的类型
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    
    #加密套件优先选择服务器的加密套件
    ssl_prefer_server_ciphers on; 
    
    #spa应用配置
    location / {
       root /var/www/mainApp; #配置应用的文件夹
       index index.html index.htm;
       try_files $uri $uri/ /index.html;
    }
}
```







