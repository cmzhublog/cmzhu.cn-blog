# trivy

trivy 是一款全面且多功能的安全扫描软件, 用于寻找安全问题, 系统bug等

支持:

- Container Image (容器镜像)
- Filesystem (文件系统)
- Git Repository (remote) (Git 仓库)
- Virtual Machine Image (虚拟机镜像)
- Kubernetes (k8s)
- AWS (AWS)

github :https://github.com/aquasecurity/trivy



## 下载

二进制下载,下载后解压直接运行即可

https://github.com/aquasecurity/trivy/releases/tag/v0.34.0

Debian/ ubuntu

```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.45.1/trivy_0.45.1_Linux-64bit.deb
dpkg -i trivy_0.45.1_Linux-64bit.deb
```



Rocky Linux /Centos 

```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.45.1/trivy_0.45.1_Linux-64bit.rpm
dnf localinstall -y trivy_0.45.1_Linux-64bit.rpm
```



## 使用方式



### docker 镜像扫描

```bash
trivy image <dockerhub.cmzhu.cn:5000/aipaas-devops/userbox-config-manager:master_2d24c53_221101162110>
```



扫描镜像漏洞

![10](./trivy.assets/10.png)



这里讲解几个比较重要的用法

指定查看漏洞的安全等级

```bash
trivy image \
			--severity [HIGH,CRITICAL] \
			<dockerhub.cmzhu.cn:5000/aipaas-devops/userbox-config-manager:master_2d24c53_221101162110>
```



忽略还未修复的漏洞

```bash
trivy image \ 
      --ignore-unfixed \
      <dockerhub.cmzhu.cn:5000/aipaas-devops/userbox-config-manager:master_2d24c53_221101162110>
```



使用json格式导出镜像详细信息到result.json 

```bash
trivy image \
      --format json \
      output result.json \
      <dockerhub.cmzhu.cn:5000/aipaas-devops/userbox-config-manager:master_2d24c53_221101162110>
```

### 文件系统

使用指令可以对文件夹, 文件系统进行扫描

```bash
trivy fs --scanners vuln,secret,config Test/
```

扫描结果

```bash
root@cmzhu:~# trivy fs --scanners vuln,secret,config Test/
2023-10-12T06:06:55.337-0400	INFO	Need to update DB
2023-10-12T06:06:55.337-0400	INFO	DB Repository: ghcr.io/aquasecurity/trivy-db
2023-10-12T06:06:55.337-0400	INFO	Downloading DB...
40.34 MiB / 40.34 MiB [-------------------------------------------------------------------] 100.00% 13.49 MiB p/s 3.2s
2023-10-12T06:07:01.784-0400	INFO	Vulnerability scanning is enabled
2023-10-12T06:07:01.784-0400	INFO	Misconfiguration scanning is enabled
2023-10-12T06:07:01.784-0400	INFO	Need to update the built-in policies
2023-10-12T06:07:01.784-0400	INFO	Downloading the built-in policies...
44.66 KiB / 44.66 KiB [--------------------------------------------------------------------------] 100.00% ? p/s 100ms
2023-10-12T06:07:04.253-0400	INFO	Secret scanning is enabled
2023-10-12T06:07:04.253-0400	INFO	If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2023-10-12T06:07:04.253-0400	INFO	Please see also https://aquasecurity.github.io/trivy/v0.45/docs/scanner/secret/#recommendation for faster secret detection
2023-10-12T06:07:04.287-0400	INFO	Number of language-specific files: 0
2023-10-12T06:07:04.287-0400	INFO	Detected config files: 0
root@cmzhu:~#
```



### k8s

扫描资源

```bash
trivy k8s deployment/orion
```

