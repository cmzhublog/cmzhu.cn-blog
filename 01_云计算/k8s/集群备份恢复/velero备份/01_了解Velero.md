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

- kubernetes 1.16 >

1、下载需要使用到的文件

```bash
 $ cd $(mktemp -d )
 $ wget https://github.com/vmware-tanzu/velero/releases/download/v1.17.0/velero-v1.17.0-linux-amd64.tar.gz
 
 $tar -zxvf velero-v1.17.0-linux-amd64.tar.gz
velero-v1.17.0-linux-amd64/LICENSE
velero-v1.17.0-linux-amd64/examples/minio/00-minio-deployment.yaml
velero-v1.17.0-linux-amd64/examples/nginx-app/README.md
velero-v1.17.0-linux-amd64/examples/nginx-app/base.yaml
velero-v1.17.0-linux-amd64/examples/nginx-app/with-pv.yaml
velero-v1.17.0-linux-amd64/velero
 $ cp velero-v1.17.0-linux-amd64/velero /usr/bin/
 $ rm -rf velero*
```



2、添加minio 配置文件

```bash
$ cat >>aliyun.credentials << EOF
ALIBABA_CLOUD_ACCESS_KEY_ID=<ALIBABA_CLOUD_ACCESS_KEY_ID>
ALIBABA_CLOUD_ACCESS_KEY_SECRET=<ALIBABA_CLOUD_ACCESS_KEY_SECRET>
EOF
```

3、 安装文件

```bash
$ BUCKET=moonrise-velero-bucket
$ REGINE=ap-southeast-1
$ velero install \
  --provider alibabacloud \
  --image velero/velero:release-1.17-dev \
  --bucket $BUCKET \
  --secret-file ./aliyun.credentials \
  --use-volume-snapshots=false \
  --backup-location-config region=${REGION} \
  --plugins kubeoperator/velero-plugin-alibabacloud:v1.0.0-2d33b89 \
  --wait
 
```

执行完成之后，可以看到如下配置即为创建成功：

```bash
$ velero install   --provider alibabacloud   --image velero/velero:release-1.17-dev   --bucket $BUCKET   --secret-file ./aliyun.credentials   --use-volume-snapshots=false   --backup-location-config region=${REGION}      --plugins velero/velero-plugin-alibabacloud:v1.0.0-2d33b89   --wait
CustomResourceDefinition/backuprepositories.velero.io: attempting to create resource
CustomResourceDefinition/backuprepositories.velero.io: attempting to create resource client
CustomResourceDefinition/backuprepositories.velero.io: created
CustomResourceDefinition/backups.velero.io: attempting to create resource
CustomResourceDefinition/backups.velero.io: attempting to create resource client
CustomResourceDefinition/backups.velero.io: created
CustomResourceDefinition/backupstoragelocations.velero.io: attempting to create resource
CustomResourceDefinition/backupstoragelocations.velero.io: attempting to create resource client
CustomResourceDefinition/backupstoragelocations.velero.io: created
CustomResourceDefinition/deletebackuprequests.velero.io: attempting to create resource
CustomResourceDefinition/deletebackuprequests.velero.io: attempting to create resource client
CustomResourceDefinition/deletebackuprequests.velero.io: created
CustomResourceDefinition/downloadrequests.velero.io: attempting to create resource
CustomResourceDefinition/downloadrequests.velero.io: attempting to create resource client
CustomResourceDefinition/downloadrequests.velero.io: created
CustomResourceDefinition/podvolumebackups.velero.io: attempting to create resource
CustomResourceDefinition/podvolumebackups.velero.io: attempting to create resource client
CustomResourceDefinition/podvolumebackups.velero.io: created
CustomResourceDefinition/podvolumerestores.velero.io: attempting to create resource
CustomResourceDefinition/podvolumerestores.velero.io: attempting to create resource client
CustomResourceDefinition/podvolumerestores.velero.io: created
CustomResourceDefinition/restores.velero.io: attempting to create resource
CustomResourceDefinition/restores.velero.io: attempting to create resource client
CustomResourceDefinition/restores.velero.io: created
CustomResourceDefinition/schedules.velero.io: attempting to create resource
CustomResourceDefinition/schedules.velero.io: attempting to create resource client
CustomResourceDefinition/schedules.velero.io: created
CustomResourceDefinition/serverstatusrequests.velero.io: attempting to create resource
CustomResourceDefinition/serverstatusrequests.velero.io: attempting to create resource client
CustomResourceDefinition/serverstatusrequests.velero.io: created
CustomResourceDefinition/volumesnapshotlocations.velero.io: attempting to create resource
CustomResourceDefinition/volumesnapshotlocations.velero.io: attempting to create resource client
CustomResourceDefinition/volumesnapshotlocations.velero.io: created
CustomResourceDefinition/datadownloads.velero.io: attempting to create resource
CustomResourceDefinition/datadownloads.velero.io: attempting to create resource client
CustomResourceDefinition/datadownloads.velero.io: created
CustomResourceDefinition/datauploads.velero.io: attempting to create resource
CustomResourceDefinition/datauploads.velero.io: attempting to create resource client
CustomResourceDefinition/datauploads.velero.io: created
Waiting for resources to be ready in cluster...
Namespace/velero: attempting to create resource
Namespace/velero: attempting to create resource client
Namespace/velero: created
ClusterRoleBinding/velero: attempting to create resource
ClusterRoleBinding/velero: attempting to create resource client
ClusterRoleBinding/velero: created
ServiceAccount/velero: attempting to create resource
ServiceAccount/velero: attempting to create resource client
ServiceAccount/velero: created
Secret/cloud-credentials: attempting to create resource
Secret/cloud-credentials: attempting to create resource client
Secret/cloud-credentials: created
BackupStorageLocation/default: attempting to create resource
BackupStorageLocation/default: attempting to create resource client
BackupStorageLocation/default: created
Deployment/velero: attempting to create resource
Deployment/velero: attempting to create resource client
Deployment/velero: created
Waiting for Velero deployment to be ready.
Velero is installed! ⛵ Use 'kubectl logs deployment/velero -n velero' to view the status.
```



4、使用 下面命令查看安装进度

```bash
$ kubectl logs deployment/velero -n velero
```



### velero 使用

1、 备份完整集群

```bash
$ velero backup create testk8s-01-20251029 --include-namespaces '*' --wait
```

2、查询备份进度和存在哪些备份

```bash
$ velero backup get
```

3、 自动化备份方案,每小时0 分备份一次，过期时间为1d

```bash
$ velero schedule create dailybackup-01 --schedule="0 * * * *"  --ttl 24h
```

4、通过下面命令查询调度状态

```bash
$ velero schedule get
```

