## nginx 例子

### 01 通过IP访问但是要将域名提供给后端

背景： 国内域名需要备案，但是测试环境没有那个多备案的需求，公司测试服务在公网，据通过IP访问，并在服务后端的nginx 上带上域名,具体写法如下：

```nginx
server {
    listen       9011 default_server;
    server_name  _;
    client_max_body_size 1024M;
    location /api1 {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host api1.cmzhu.cn;
        proxy_set_header Origin api1.cmzhu.cn;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
    location /api2 {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host api2.cmzhu.cn;
        proxy_set_header Origin api2.cmzhu.cn;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
    location /api3 {
        proxy_pass http://127.0.0.1:30004;
        proxy_set_header Host api3.cmzhu.cn;
        proxy_set_header Origin api3.cmzhu.cn;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
    location /api4 {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host api4.cmzhu.cn;
        proxy_set_header Origin api4.cmzhu.cn;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

> 通过上述的配置，访问127.0.0.1:9091/api1， header带给后端服务的域名是api1.cmzhu.cn , 以此类推

### 02 通过location替换掉后端的地址的 URI 部分

背景：

