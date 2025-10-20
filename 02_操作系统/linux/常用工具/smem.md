## Linux 内存监控工具 SMEM

1、 下载安装

```bash
$ cd $(mktemp -d )
$ wget https://rpmfind.net/linux/epel/9/Everything/x86_64/Packages/s/smem-1.5-11.el9.x86_64.rpm 
$ yum localinstall -y smem-1.5-11.el9.x86_64.rpm
```



使用方法

```bash
$ smem -tk 

```

