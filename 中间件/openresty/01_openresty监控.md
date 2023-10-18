## openresty 监控

### nginx-lua-prometheus

github 地址: 

[https://github.com/knyar/nginx-lua-prometheus]: https://github.com/knyar/nginx-lua-prometheus

这是一个 lua 的库, 可以和nginx 一起来跟踪指标, 并将他们公布在单独的网页上, 由prometheus 拉取.



#### 安装

安装此库, 需要安装ngx_lua 模块, 可以简单的安装 libnginx-mod-http-lua

库文件 - `prometheus.lua` - 需要在 中 `LUA_PATH` 可用。如果这是您使用的唯一 Lua 库，则只需指向 `lua_package_path` 签出此 git 存储库的目录（请参阅下面的示例）。

openresty 在 opm 中可用, 他可以通过[luarocks](https://luarocks.org/modules/knyar/nginx-lua-prometheus).获取

```bash
$ opm get knyar/nginx-lua-prometheus
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
        LANGUAGE = "en_US.UTF-8",
        LC_ALL = "en_US.UTF-8",
        LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
* Fetching knyar/nginx-lua-prometheus  
  Downloading https://opm.openresty.org/api/pkg/tarball/knyar/nginx-lua-prometheus-0.20230607.opm.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 19265  100 19265    0     0   7056      0  0:00:02  0:00:02 --:--:--  7056
Package knyar/nginx-lua-prometheus 0.20230607 installed successfully under /usr/local/openresty/site/ .
```

查看文件是否存在:

```bash
$ ls /usr/local/openresty/site/lualib/prometheus.lua
/usr/local/openresty/site/lualib/prometheus.lua
```

#### 配置使用

1、 先在nginx.conf 中的 http 模块中加入如下配置

```nginx
    lua_shared_dict prometheus_metrics 10M; 
    lua_package_path "/usr/local/openresty/site/lualib/?.lua;;";

    init_worker_by_lua_block {
      prometheus = require("prometheus").init("prometheus_metrics")

      metric_requests = prometheus:counter(
        "nginx_http_requests_total", "Number of HTTP requests", {"host", "status"})
      metric_latency = prometheus:histogram(
        "nginx_http_request_duration_seconds", "HTTP request latency", {"host"})
      metric_connections = prometheus:gauge(
        "nginx_http_connections", "Number of HTTP connections", {"state"})
    }

    log_by_lua_block {
      metric_requests:inc(1, {ngx.var.server_name, ngx.var.status})
      metric_latency:observe(tonumber(ngx.var.request_time), {ngx.var.server_name})
    } # https://cdn.console.aliyun.com/domain/detail/cdn.bigquant.com/backSrc

```

2、配置监控使用的server

```nginx
server {
  listen 9145;
  allow 192.168.0.0/16;
  deny all;
  location /metrics {
    content_by_lua_block {
      metric_connections:set(ngx.var.connections_reading, {"reading"})
      metric_connections:set(ngx.var.connections_waiting, {"waiting"})
      metric_connections:set(ngx.var.connections_writing, {"writing"})
      prometheus:collect()
    }
  }
}
```

 