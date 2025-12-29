## Linux 新增SWAP 分区

### 添加步骤

1、 创建2G 的swap 文件，使用dd 命令创建全为0 的空文件

```bash
$ dd if=/dev/zero of=/dev/swap bs=1M count=2048 
```

2、初始化文件`/dev/swap`和为`/dev/swap`授权

```bash
$ chmod 0600 /mnt/swap
# 文件系统初始化
$ mkswap /mnt/swap
```

3、 创建swap 分区

```bash
$ swapon /mnt/swap
```

4、查询swap 分区

```bash
$ swapon -f
NAME      TYPE SIZE USED PRIO
/mnt/swap file   2G 1.8M   -2
```

5、 持久化挂载swap

```bash
$ echo "/mnt/swap swap swap defaults 0 0 " >> /etc/fstab && mount -a 
```

