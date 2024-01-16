## squid

Squid是一种开源的代理服务器软件，它充当了客户端和互联网之间的中介。它可以缓存常用的网页内容，提高访问速度，并且可以控制和监控网络流量。Squid还可以进行访问控制，允许或禁止特定的客户端访问特定的网站或资源。它通常用于企业、学校、机构等组织的网络环境中，用于提供更快速、安全和可控制的网络访问



### squid 搭建(docker)



```bash
$ cat docker-compose.yml
version: '3.4'

services:
  squid:
    image: b4tman/squid
    container_name: squid
    ports:
      - "3128:3128"
    volumes:
      - ./cache:/var/spool/squid
      - ./squid.conf:/etc/squid/squid.conf
```

配置文件如下: 



```bash
$ cat ./squid.conf
http_access allow all
http_port 3128
via off
forwarded_for off
follow_x_forwarded_for deny all
request_header_access X-Forwarded-For deny all
header_access X_Forwarded_For deny all
cache_store_log none
cache_access_log /dev/null
cache_log /dev/null
```

