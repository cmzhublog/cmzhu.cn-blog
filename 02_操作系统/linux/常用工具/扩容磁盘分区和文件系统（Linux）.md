## 扩容云硬盘分区和文件系统（Linux）

>  **示例说明：数据盘“/dev/vdb”原有容量100GiB，只有一个分区“/dev/vdb1”。将数据盘容量扩大至150GiB，将新增的50GB增加至已有分区“/dev/vdb1”。**



### 背景

- 系统盘 /dev/vda1
- 云平台扩容200G，但是没有扩容分区 

### 扩容步骤

1、 检查扩容工具和当前磁盘信息。

```bash
$ growpart
growpart disk partition
  rewrite partition table so that partition takes up all the space it can
  options:
  -h | --help	        print Usage and exit
       --fudge F        if part could be resized, but change would be
                        less than 'F' bytes, do not resize (default: 1048576)
  -N | --dry-run        only report what would be done, show new 'sfdisk -d'
  -v | --verbose	increase verbosity / debug
  -u | --update R	update the the kernel partition table info after growing
                        this requires kernel support and 'partx --update'
                        R is one of:
                        - 'auto': [default] update partition if possible
                        - 'force' : try despite- sanity checks (fail on failure)
                        - 'off' : do not attempt
                        - 'on'	: fail if sanity checks indicate no support
  Example:
  - growpart /dev/sda 1
    Resize partition 1 on /dev/sda
must supply disk and part it ion-number
[root@ecs-centos76 ~l#
```

2、如果没有以上回显信息，请执行以下安装命令。

```bash
$ yum install cloud-utils-growpart
```



查看数据盘“/dev/vdb”的分区信息。

**lsblk**

```
[root@ecs-centos76 ~]# lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  40G  0 disk
├vda1 253:1    0  40G  0 part /
vdb    253:16   0  150G 0 disk
├vdb1 253:17   0  100G 0 part /mnt/sdc
```

3、扩容分区

扩容分区到`/dev/vdb1`

```bash
$ growpart  /dev/vdb 1
```

4、 查看分区类型

```bash
$ parted /dev/vdb
GNU Parted 3.1
Using /dev/vdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) p
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 107GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:
 
Number  Start   End    Size     File system    Name       Flags
 1      1049KB  107GB  107GB    ext4           /dev/vdb1

(parted)
```

“Partition Table”显示分区格式为“GPT”，“File system”显示文件系统类型为ext4。

查看完成后，输入“q”，按“Enter”，退出parted模式。

4.1 当文件系统类型为ext4时，执行以下命令进行扩容。



```bash
$ resize2fs /dev/vdb1
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vdb1 is mounted on /mnt/sdc; on-line resizing required
old_desc_blocks = 13, new_desc_blocks = 19
The filesystem on /dev/vdb1 is now 39321339 blocks long.
```



4.2 如果文件系统为xfs，请执行以下命令（/mnt/sdc为/dev/vdb1的挂载点，您需要根据实际情况修改）。

```
[root@ecs-test-0001 ~]# sudo xfs_growfs /mnt/sdc
meta-data=/dev/vdb1              isize=512    agcount=4, agsize=6553536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=26214144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=12799, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 26214144 to 39321339
```