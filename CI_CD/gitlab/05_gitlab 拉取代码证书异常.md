## gitlab 拉取代码证书异常

### 报错详情

```bash
$ git clone https://gitlab.cmzhu.cn/develop/testin.git

Cloning into 'testin'...
fatal: unable to access 'https://gitlab.cmzhu.cn/develop/testin.git/': SSL certificate problem: self-signed certificate
```



### 原因解释

1、 证书不可信导致



### 解决思路

1、修改为可信证书

2、 终端信任该证书

```bash
$ git config --global http.sslVerify false
```

