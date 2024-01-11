## nginx 反向代理

### 背景

1、 nginx 反向代理之后, 发现websocket 建立连接会pending, 查看nginx 会发现websocket 会报错

网络架构如下

```Mermaid
graph LR
  A{Start} --> B["nginx服务器(test02.com)"]
  B --反向代理域名--> C[LB 服务器]
  C -- TCP代理 --> D["Ingress 的控制节点(master节点: test01.com)"]
  D --> E[service]
  E --> F[Pod]
    
  
```



### 解决

1、 调整nginx 配置如下

```nginx
# langchain-ras
upstream websocketest {
  server 10.0.0.1;
  keepalive 200;
}
map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}
server {
        listen       80;
        server_name  test02.com;
        #ssl on;
        autoindex off;
location / {
			proxy_pass http://websocketest;
            proxy_set_header Host test01.com;
            proxy_set_header Origin test01.com;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "$connection_upgrade";
        }
}
```



