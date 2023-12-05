## 了解 velero 

Velero（前身为 Heptio Ark）为您提供了备份和恢复 Kubernetes 集群资源和持久卷的工具。您可以使用云提供商或本地运行 Velero。Velero 可让您：

- 对群集进行备份，并在丢失时进行还原
- 将集群资源迁移到其他集群
- 将生产群集复制到开发和测试群集

Velero 包括:

- 在群集上运行的服务器
- 在本地运行的命令行客户端

### 安装

安装步骤依赖

- minio
- kubernetes 1.16 >

1、下载需要使用到的文件

```bash
 $ wget  https://github.com/vmware-tanzu/velero/releases/download/v1.12.2-rc.1/velero-v1.12.2-rc.1-linux-amd64.tar.gz
 
 $ tar -zxvf velero-v1.12.2-rc.1-linux-amd64.tar.gz
velero-v1.12.2-rc.1-linux-amd64/LICENSE
velero-v1.12.2-rc.1-linux-amd64/examples/minio/00-minio-deployment.yaml
velero-v1.12.2-rc.1-linux-amd64/examples/nginx-app/README.md
velero-v1.12.2-rc.1-linux-amd64/examples/nginx-app/base.yaml
velero-v1.12.2-rc.1-linux-amd64/examples/nginx-app/with-pv.yaml
velero-v1.12.2-rc.1-linux-amd64/velero

 $ mv velero-v1.12.2-rc.1-linux-amd64/velero /usr/local/bin
 $ rm -rf velero*
```



2、添加minio 配置文件

```bash
$ cat <<EOF>> minio.credentials
[default]
aws_access_key_id=minio
aws_secret_access_key=minio123
EOF
```

3、 安装文件

```bash
$ velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.7.0 \
  --bucket bak \
  --secret-file ./minio.credentials \
  --use-volume-snapshots=false \
  --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://minio.minio:9000 
```

4、使用 下面命令查看安装进度

```bash
$ kubectl logs deployment/velero -n velero
```

