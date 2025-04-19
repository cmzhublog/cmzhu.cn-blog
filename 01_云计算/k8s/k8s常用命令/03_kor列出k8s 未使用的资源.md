## kor

[github 地址]: https://github.com/yonahd/kor

kor 是一种用于在k8s 集群中发现未使用资源的应用,能够支持以下这些资源的查询:

- ConfigMaps
- Secrets
- Services
- ServiceAccounts
- Deployments
- StatefulSets
- Roles
- HPAs
- PVCs
- Ingresses
- PDBs

### 安装

下载二进制文件进行安装

```bash
wget https://github.com/yonahd/kor/releases/download/v0.2.4/kor_Linux_x86_64.tar.gz
tar kor_Linux_x86_64.tar.gz -C /usr/bin/kor
```

