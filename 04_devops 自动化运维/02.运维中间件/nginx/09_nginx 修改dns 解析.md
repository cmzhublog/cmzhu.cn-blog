## nginx 出现dns 解析IP 缓存情况



nginx 的upsteam 指定到了域名和实际不一致的IP地址，这个时候大概率是nginx 的解析缓存，可以重新nginx -s reload 

```bash
$ nginx -t 
$ nginx -s reload 
```

