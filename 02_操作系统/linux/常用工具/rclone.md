# 使用 rclone 同步 S3 / UCloud US3 数据

本文覆盖两种场景：

1. 不同 S3 服务之间同步，例如 AWS S3 → UCloud US3、两个不同 US3 Endpoint 之间同步。
2. 同一个 S3 中不同 bucket 或不同“文件夹”（对象前缀）之间同步。

> **重要：**`rclone sync` 会删除目标端多出来的对象，使目标与源完全一致。首次执行必须加 `--dry-run`。如果不希望删除目标端任何对象，请使用 `rclone copy`。

## 1. 基本概念

rclone 路径格式如下：

```text
remote名称:bucket名称/对象前缀
```

例如：

```text
us3_shanghai:my-bucket/data/2026
```

S3 没有真正的文件夹，`data/2026` 实际上是对象 key 的前缀。同步的是该路径下的**内容**，不会自动把最末级目录名再套一层。

`copy` 与 `sync` 的区别：

| 命令 | 覆盖目标同名对象 | 复制新增对象 | 删除目标多余对象 | 推荐用途 |
|---|---:|---:|---:|---|
| `rclone copy` | 是 | 是 | 否 | 首次迁移、增量复制、保守操作 |
| `rclone sync` | 是 | 是 | **是** | 目标必须成为源的镜像 |

## 2. 安装与检查

从 [rclone 官方安装页面](https://rclone.org/install/) 安装，然后确认版本：

```bash
rclone version
```

建议使用当前稳定版，因为新版 rclone 已把 `US3` 列为内置 S3 Provider。

## 3. 配置 Remote

运行交互式配置：

```bash
rclone config
```

### 3.1 配置 UCloud US3

依次选择：

```text
n                 # New remote
name: us3_source
Storage: s3
provider: US3
env_auth: false
access_key_id: <US3 Token 公钥>
secret_access_key: <US3 Token 私钥>
endpoint: <选择 US3 所在地域的 S3 Endpoint>
acl: private
storage_class: 默认
```

上海地域的 Endpoint 示例：

```text
s3-cn-sh2.ufileos.com
```

其他地域请以 [UCloud US3 地域和域名文档](https://docs.ucloud.cn/ufile/introduction/region) 以及 rclone 配置向导中列出的 Endpoint 为准。在同地域 UCloud 云主机上运行时，可考虑使用内网 Endpoint；运行机器必须能访问相应内网。

US3 使用 S3 兼容接口时，建议在高级配置中确认以下值：

```ini
provider = US3
force_path_style = true
chunk_size = 8Mi
upload_cutoff = 100Mi
copy_cutoff = 100Mi
```

原因是 US3 默认要求 8 MiB 固定分片，且单次 `CopyObject` 最大支持 100 MB。部分地域支持动态分片，具体以账号和地域能力为准。

### 3.2 配置另一个 S3

再次运行：

```bash
rclone config
```

创建 `s3_destination`。如果目标是 AWS S3，选择 `AWS`；如果是其他兼容 S3，选择对应 Provider，找不到时选择 `Other`，并填写服务商提供的：

- Access Key ID
- Secret Access Key
- Region
- S3 Endpoint
- Path style / virtual-host style 设置

不要把真实密钥写入脚本、Git 仓库或本文档。可执行以下命令加密 rclone 配置文件：

```bash
rclone config encryption set
```

查看配置文件位置和已配置的 remote：

```bash
rclone config file
rclone listremotes
```

## 4. 连接测试

分别确认两个 Remote 可以列出 bucket：

```bash
rclone lsd us3_source:
rclone lsd s3_destination:
```

查看指定目录：

```bash
rclone lsf us3_source:source-bucket/source-folder --max-depth 2
rclone lsf s3_destination:destination-bucket/destination-folder --max-depth 2
```

如果这里只读操作都失败，不要开始同步。先检查 Endpoint、密钥权限、Region、系统时间和网络连通性。

## 5. 不同 S3 之间同步

假设：

- 源：`us3_source:source-bucket/data`
- 目标：`s3_destination:destination-bucket/backup/data`

### 5.1 推荐：先执行不删除目标文件的 copy

预演：

```bash
rclone copy \
  us3_source:source-bucket/data \
  s3_destination:destination-bucket/backup/data \
  --dry-run \
  --size-only \
  --progress \
  --transfers 8 \
  --checkers 16
```

确认输出中的源、目标和变更数量正确后，删除 `--dry-run` 正式执行：

```bash
rclone copy \
  us3_source:source-bucket/data \
  s3_destination:destination-bucket/backup/data \
  --size-only \
  --progress \
  --transfers 8 \
  --checkers 16 \
  --log-level INFO \
  --log-file rclone-copy.log
```

### 5.2 镜像同步：让目标与源完全一致

先预演：

```bash
rclone sync \
  us3_source:source-bucket/data \
  s3_destination:destination-bucket/backup/data \
  --dry-run \
  --size-only \
  --progress \
  --max-delete 100
```

确认目标端将被删除的对象完全符合预期后，删除 `--dry-run` 执行。`--max-delete 100` 表示本次最多删除 100 个目标对象；超过限制时任务失败，从而降低路径写错造成的大量误删风险。请按实际情况调整，但不建议首次运行时取消限制。

```bash
rclone sync \
  us3_source:source-bucket/data \
  s3_destination:destination-bucket/backup/data \
  --size-only \
  --progress \
  --transfers 8 \
  --checkers 16 \
  --max-delete 100 \
  --log-level INFO \
  --log-file rclone-sync.log
```

不同 Remote 之间通常无法使用服务端复制，数据会从源 S3 下载到运行 rclone 的机器，再上传到目标 S3。因此需要考虑：

- 源端公网下行流量费用；
- 运行机器的带宽和磁盘无关，数据通常以流式方式传输；
- 两端 API 请求费用和限流；
- 跨境或跨地域网络稳定性。

## 6. 同一个 S3 内同步文件夹

假设同一个 US3 Remote 和 bucket 中：

- 源目录：`data/2026`
- 目标目录：`backup/2026`

### 6.1 安全复制，不删除目标多余对象

```bash
rclone copy \
  us3_source:my-bucket/data/2026 \
  us3_source:my-bucket/backup/2026 \
  --dry-run \
  --size-only \
  --progress
```

确认后去掉 `--dry-run`。

### 6.2 完全同步，删除目标多余对象

```bash
rclone sync \
  us3_source:my-bucket/data/2026 \
  us3_source:my-bucket/backup/2026 \
  --dry-run \
  --size-only \
  --progress \
  --max-delete 100
```

确认后去掉 `--dry-run`。

当源和目标使用**同一个 Remote 名称**时，rclone 会尽量请求 S3 执行服务端复制，数据不需要经过本机，通常更快且避免公网数据下载。不要为了同一个 S3 创建两个内容相同但名字不同的 Remote，否则 rclone 通常无法识别为可执行服务端复制的同一后端。

> 源目录和目标目录不能相同，也不应互相包含。例如不要把 `bucket/data` 同步到 `bucket/data/backup`，否则属于重叠路径。

### 6.3 同一 Remote、不同 bucket

```bash
rclone sync \
  us3_source:source-bucket/data \
  us3_source:destination-bucket/data \
  --dry-run \
  --size-only \
  --progress \
  --max-delete 100
```

两个 bucket 必须位于支持服务端复制的兼容范围内，通常要求同地域。否则可能失败，或需要改用两个 Remote 让数据经过本机中转。

## 7. 为什么示例使用 `--size-only`

UCloud 文档说明，US3 的 ETag 计算方式与 AWS S3 有差异，不建议依赖 ETag。因此示例使用 `--size-only` 判断对象是否需要传输。

它的代价是：如果文件内容变化但大小完全相同，rclone 可能认为文件未变化。可根据要求选择：

- 日常增量同步：使用 `--size-only`，API 请求少、兼容性较好。
- 强制重新复制所有对象：使用 `--ignore-times`，流量和请求量更大。
- 高强度内容验证：同步后运行 `rclone check --download`，它会读取两端全部对象内容，准确但会产生大量流量。

不要在不了解目标 S3 哈希语义时盲目添加 `--checksum`。

## 8. 同步后验证

快速检查对象大小和路径：

```bash
rclone check \
  us3_source:source-bucket/data \
  s3_destination:destination-bucket/backup/data \
  --size-only \
  --combined check-result.txt \
  --progress
```

`check-result.txt` 中常见标记：

```text
= 两端一致
- 只存在于目标
+ 只存在于源
* 两端对象不同
! 检查出错
```

如需逐字节验证，并接受两端下载流量：

```bash
rclone check \
  us3_source:source-bucket/data \
  s3_destination:destination-bucket/backup/data \
  --download \
  --combined check-download-result.txt \
  --progress
```

## 9. 常用参数

| 参数 | 作用 |
|---|---|
| `--dry-run` | 只显示计划，不修改数据 |
| `--interactive` / `-i` | 每项操作前交互确认 |
| `--progress` / `-P` | 显示实时进度 |
| `--size-only` | 仅按文件大小判断是否相同 |
| `--ignore-times` | 不跳过相同对象，强制传输 |
| `--transfers N` | 同时传输的对象数 |
| `--checkers N` | 并发检查数量 |
| `--max-delete N` | 限制一次同步最多删除多少对象 |
| `--bwlimit 50M` | 将传输速度限制为约 50 MiB/s |
| `--fast-list` | 减少列表请求，但会使用更多内存 |
| `--exclude PATTERN` | 排除匹配的对象 |
| `--log-file FILE` | 把日志写入文件 |

并发不是越高越好。遇到 `429`、`503 Slow Down`、连接重置或 US3 限流时，应减小 `--transfers` 和 `--checkers`。

## 10. 定时同步示例

建议先手动成功运行多次，再加入 cron。以下示例每天凌晨 02:30 执行不删除目标对象的增量复制：

```cron
30 2 * * * /usr/local/bin/rclone copy us3_source:source-bucket/data s3_destination:destination-bucket/backup/data --size-only --transfers 8 --checkers 16 --log-level INFO --log-file /var/log/rclone-s3-copy.log
```

定时任务中应使用 `rclone` 的绝对路径，并确认执行用户可以读取 rclone 配置文件。可用以下命令查看实际路径：

```bash
command -v rclone
rclone config file
```

## 11. US3 特别注意事项

- US3 Token 至少需要源端列举和读取权限，以及目标端列举、写入权限；使用 `sync` 时还需要目标端删除权限。
- US3 S3 接口默认采用 8 MiB 固定分片。超大文件可能因为 10,000 分片上限而受限；部分地域可联系 UCloud 技术支持开通动态分片。
- US3 的单次 `CopyObject` 最大为 100 MB。更大对象的同 Remote 服务端复制依赖 Multipart Copy / `UploadPartCopy` 能力；若对应地域未开通该能力，可使用两个不同 Remote 名称强制经本机下载再上传。
- 归档存储对象必须先解冻才能读取或跨 S3 迁移。
- rclone 不支持 key 中连续的双斜杠 `//`；已有此类对象应先评估并重命名。

## 12. 官方参考

- [rclone S3 后端及 US3 配置](https://rclone.org/s3/)
- [rclone sync 命令](https://rclone.org/commands/rclone_sync/)
- [rclone check 命令](https://rclone.org/commands/rclone_check/)
- [UCloud US3 S3 协议支持说明](https://docs.ucloud.cn/ufile/s3/s3_introduction)
- [UCloud US3 地域和域名](https://docs.ucloud.cn/ufile/introduction/region)