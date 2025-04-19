---
title: buddymirror
description: 
published: true
date: 2023-05-16T11:01:10.539Z
tags: linux
editor: markdown
dateCreated: 2023-05-16T11:01:10.539Z
---

## `BuddyMirror`

### `beegfs` 集群配置



| 节点      | `hostname` | 文件存储                    | ID   |
| --------- | ---------- | --------------------------- | ---- |
| `Mgmtd`   | `Mgmtd`    | `/data/beegfs/beegfs_mgmtd` |      |
| `Meta`    | `Meta1`    | `/data/lun_meta`            | 201  |
| `Meta`    | `Meta2`    | `/data/lun_meta`            | 202  |
| `Storage` | `Storage1` | `/mnt/lun_storage_1`        | 3011 |
| `Storage` | `Storage1` | `/mnt/lun_storage_2`        | 3012 |
| `Storage` | `Storage2` | `/mnt/lun_storage_1`        | 3021 |
| `Storage` | `Storage2` | `/mnt/lun_storage_2`        | 3022 |
| `Storage` | `Storage3` | `/mnt/lun_storage_1`        | 3031 |
| `Storage` | `Storage3` | `/mnt/lun_storage_2`        | 3032 |



> `beegfs` 的`BuddyMirror` 不是设置后就能直接按照`BuddyMirror`来写入, 而是要指定数据存储的桶才能使用

### 配置方案

使用 `beegfs-ctl` 可以指定不同的服务创建`BuddyMirror`

```bash
## 创建meta 的BuddyMirror
$ beegfs-ctl --addmirrorgroup --automatic --nodetype=meta
## 创建storage 的BuddyMirror
$ beegfs-ctl --addmirrorgroup --automatic --nodetype=storage
```

 如果要查看所有的配置, 可以使用

```bash
$ beegfs-ctl --addmirrorgroup --help

[root@bqmgmtd ~]# beegfs-ctl --addmirrorgroup --help
GENERAL USAGE:
  $ beegfs-ctl --<modename> [mode_arguments] [client_arguments]

CLIENT ARGUMENTS:
 Optional:
  --mount=<path>           Use config settings from this BeeGFS mountpoint
                           (instead of "--cfgFile").
  --cfgFile=<path>         Path to BeeGFS client config file.
                           (Default: /etc/beegfs/beegfs-client.conf)
  --logEnabled             Enable detailed logging.
  --<key>=<value>          Any setting from the client config file to override
                           the config file values (e.g. "--logLevel=5").

MODE ARGUMENTS:
 Mandatory for automatic group creation:
  --automatic             Generates the mirror buddy groups automatically with
                          respect to the location of the targets. This mode
                          aborts the creation of the mirror buddy groups if some
                          constraints are not fulfilled.
  --nodetype=<nodetype>   The node type (metadata, storage).

 Mandatory for manual group creation:
  --primary=<targetID>    The ID of the primary target for this mirror buddy
                          group.
  --secondary=<targetID>  The ID of the secondary target for this mirror buddy
                          group.
  --nodetype=<nodetype>   The node type (metadata, storage).

 Optional for automatic group creation:
  --force                 Ignore the warnings of the automatic mode.
  --dryrun                Only print the selected configuration of the
                          mirror buddy groups.
  --uniquegroupid         Use unique mirror buddy group IDs, distinct from
                          storage target IDs.

 Optional for manual group creation:
  --mirrorgroupid=<groupID>
  --groupid=<groupID>     Use the given ID for the new buddy group.
                          Only one of the two options may given.
                          (Valid range: 1..65535)

USAGE:
 This mode adds a mirror buddy group and maps two storage targets to the group.

 Example: Automatically generate mirror buddy groups from all storage targets
  # beegfs-ctl --addmirrorgroup --automatic --nodetype=storage
```

使用以下命令创建存储`BuddyMirror`

```bash
beegfs-ctl --addmirrorgroup --nodetype=storage --primary=3011 --secondary=3022 --groupid=100

beegfs-ctl --addmirrorgroup --nodetype=storage --primary=3021 --secondary=3032 --groupid=101

beegfs-ctl --addmirrorgroup --nodetype=storage --primary=3031 --secondary=3012 --groupid=102
```



查看`BuddyMirror`配置

```bash
# 查看storage buddy mirror
$ beegfs-ctl --listmirrorgroups --nodetype=storage
# 查看meta buddy mirror
$ beegfs-ctl --listmirrorgroups --nodetype=meta

[root@bqmgmtd ~]# beegfs-ctl --listmirrorgroups --nodetype=storage
     BuddyGroupID   PrimaryTargetID SecondaryTargetID
     ============   =============== =================
              102              3011              3021
              103              3012              3031
              203              3022              3032
              
# 查看 mirror 详细信息
[root@bqmgmtd ~]# beegfs-ctl --listtargets --mirrorgroups
MirrorGroupID MGMemberType TargetID   NodeID
============= ============ ========   ======
          102      primary     3011      301
          102    secondary     3021      302
          103      primary     3012      301
          103    secondary     3031      303
          203      primary     3022      302
          203    secondary     3032      303

```



启用`Mirror`

> `Storage ` 节点支持 `per-directory basis` , 意思是在文件系统中可以支持一部分数据使用`BuddyMirror`, 一部分数据不使用`BuddyMirror`
>
> `meta` 不同, 如果需要使用`BuddyMirror`, 就必须整个路径全量使用;

```bash
beegfs-ctl --setpattern --buddymirror --help

[root@bqmgmtd ~]# beegfs-ctl --setpattern --buddymirror --help
GENERAL USAGE:
  $ beegfs-ctl --<modename> [mode_arguments] [client_arguments]

CLIENT ARGUMENTS:
 Optional:
  --mount=<path>           Use config settings from this BeeGFS mountpoint
                           (instead of "--cfgFile").
  --cfgFile=<path>         Path to BeeGFS client config file.
                           (Default: /etc/beegfs/beegfs-client.conf)
  --logEnabled             Enable detailed logging.
  --<key>=<value>          Any setting from the client config file to override
                           the config file values (e.g. "--logLevel=5").

MODE ARGUMENTS:
 Mandatory:
  <path>                       Path to a directory.
                               Specify "-" here to read multiple paths from
                               stdin (separated by newline).

 Optional:
  --storagepoolid=<poolID>     Use poolID as storage pool for new files in this
                               directory. Can be set to "default" to use 
                               the default pool.
  --chunksize=<bytes>          Block size for striping (per storage target).
                               Suffixes 'k' (kilobytes) and 'm' (megabytes) are
                               allowed.
  --pattern=<pattern>          Set the stripe pattern type to use. Can be
                               raid0 (default) or buddymirror. Buddy mirroring
                               mirrors file contents across targets.
                               Each target will be mirrored on a corresponding
                               mirror target (its mirror buddy).
                               This is an enterprise feature. See end-user
                               license agreement for definition and usage
                               limitations of enterprise features.
  --numtargets=<number>        Number of stripe targets (per file).
                               If the stripe pattern is set to 'buddymirror',
                               this is the number of mirror groups.
  --force                      Allow buddy mirror pattern to be set, even if no
                               groups have been defined.
  --unmounted                  If this is specified then the given path is
                               relative to the root directory of a possibly
                               unmounted BeeGFS. (Symlinks will not be resolved
                               in this case.)

USAGE:
 This mode sets a new striping configuration for a certain directory. The new
 striping configuration will only be effective for new files and new
 subdirectories of the given directory.

 This mode can be used by non-root users only if the administrator
 enabled `sysAllowUserSetPattern` in the metadata server config.
 This enables normal users to change the number of stripe targets
 (--numtargets) and the chunk size (--chunksize) for files that they
 own themselves. All other parameters can only be changed by root.

 Example: Set directory pattern to 1 megabyte chunks and 4 targets per file
  # beegfs-ctl --setpattern --chunksize=1m --numtargets=4 /mnt/beegfs/mydir

 Example: Apply new pattern to all subdirectories of /mnt/beegfs/mydir
  # find /mnt/beegfs/mydir -type d | \
     beegfs-ctl --setpattern --chunksize=1m --numtargets=4 -
```

启用 `storage` 的`BuddyMirror`

> --numtargets 设置成和存储的mirror桶数量一致

```bash
mkdir /mnt/beegfs/buddymirror
beegfs-ctl --setpattern --pattern=buddymirror --chunksize=1m --numtargets=3 /mnt/beegfs/buddymirror
```



 配置`meta `节点的`BuddyMirror`

```bash
beegfs-ctl --addmirrorgroup --nodetype=meta --primary=201 --secondary=202 --groupid=102
```

 启用`meta `节点 的`buddymirror`

```bash
## 激活meta
[root@bqmgmtd ~]# beegfs-ctl --mirrormd
Operation succeeded.

NOTE:
 To complete activating metadata mirroring, please remount any clients and
 restart all metadata services now.
## stop 所有 client 和重启meta 节点
systemctl stop beegfs-client
systemctl restart beegfs-meta

```

