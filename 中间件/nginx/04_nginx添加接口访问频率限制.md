## nginx 添加接口访问频率限制

部分接口如果出现较高访问频率, 可能会影响到其他服务的访问, 为此; 可以在nginx 的配置中可以使用`ngx_http_limit_req_module`模块对部分接口进行限速

### 配置方式

配置示例

```
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

    server {
        location / {
            limit_req zone=one burst=5;
            # 其他配置项...
        }
    }
}
```

```nginx
...
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
...
...

limit_req zone=one burst=5;
...
```

其中: 

- limit_req_zone: 表示使用`ngx_http_limit_req_module` 模块
- $binary_remote_addr: 为nginx 的内置变量, 表示请求的客户端的IP
- zone=one:10m : 表示创建一个名字为one 的限制区域, 分配配one区域的内存大小为10m
- rate=10r/s : 表示访问速率, 此配置为每秒钟最多访问10次
- limit_req : 在location 引用前面配置的限制区域, 表示此类匹配的接口都会应用限速
- zone=one : 指定限制区域
- burst=5: 表示突变值为5, 在访问中如果达到10次每秒, 可以额外突发五次