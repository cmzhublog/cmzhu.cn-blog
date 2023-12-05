## Proxy Protocol 

### 什么是Proxy Protocol?

是一种在代理服务器和客户端之间传递信息的标准协议。它可以在不影响客户端和服务器之间原有通信方式的情况下，向服务器传递客户端的真实IP地址和端口等信息

当一个客户端通过代理服务器向服务器发送请求时，代理服务器会将客户端的IP地址和端口等信息添加到Proxy Protocol协议头部，并将请求转发给服务器。当服务器收到请求时，会从Proxy Protocol协议头部解析出客户端的IP地址和端口等信息，并使用这些信息来响应客户端的请求。这样，即使客户端和服务器之间通过代理服务器进行通信，服务器仍然可以准确地知道客户端的真实IP地址和端口等信息，从而可以更好地处理请求

Proxy Protocol协议主要有两个版本：

- Proxy Protocol v1
- Proxy Protocol v2。



其中，Proxy Protocol v1使用文本格式传输信息，而Proxy Protocol v2使用二进制格式传输信息，并支持加密和压缩等功能。目前，Proxy Protocol协议已经被广泛应用于各种代理服务器和负载均衡设备中，例如HAProxy、Nginx、ELB等

### nginx 中使用 Proxy Protocol

通过如下配置, Nginx可以实现在TCP/UDP协议和HTTP/HTTPS协议同时支持Proxy Protocol

```nginx
http {
    #...
    server {
        listen 80   proxy_protocol;
        listen 443  ssl proxy_protocol;
        #...
    }
}
   
stream {
    #...
    server {
        listen 112233 proxy_protocol;
        #...
    }
}

```

