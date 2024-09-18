## 为什么Linux 的Buffer 和Cache 内存占用过高

### 现象

在linux 系统中，我们常用`free` 来查看内存状态

```bash
$ free -g 
              total        used        free      shared  buff/cache   available
Mem:            125          45           8           4          71          74
Swap:            31          26           5
```

从上面 `free -g ` 返回结果来看， `buff` 和`cache` 占用`71 G` , 然实际使用内存为`45 G `, 是什么原因造成这种结果呢？下面来详细分析一下

### 什么是Buff/Cache?

在开始研究之前， 需要了解什么是 `Buffer` 和 `Cache`? 

**在`Linux 2.4 ` 的版本内存管理中**，Buff 是指Linux 内存的 Buffer Cache，  Cache 是指Linux 内存管理中的Page Cache，一般解释如下

- A buffer is someting that has yet to be ‘written’ to disk.
- A cache is someting that has been ‘read’ from the disk and stored for later use.

翻译过来就是说：

1. buffer (buff) 是用来缓存尚未 “写入” 磁盘的内容。
2. cache 是用来缓存从磁盘 “读取” 出来的东西。

所以 `buffer` 被用来当成对 io 设备写的缓存。而 `cache` 被用来当作对 io 设备的读缓存。这里的 io 设备，主要指的是块设备文件和文件系统上的普通文件。 

**但是 在Linux 2.6 之后， 他们的意义发生了变化**

在 Linux 2.6 之后， 将Buffer Cache 和 Page Cache 合并到了 Page Cache 作为文件层的缓存， 而 Buffer 则被用作block 层的缓存，Block 层是什么意思呢？ 可以理解为一个Buffer 就是 一个 Physical Disk Block 的内存代表，用来将内存中的Page 映射到Disk Blocks ， 这部分用途的内存称作buffer

> buffer 里面的pages ， 指的是page Cache 里面的pages ， 所以 buffer 也被称为 Page Cache 的一部分

### Buffer 的具体职责

在当前的系统实现里，`buffer` 主要是设计用来在系统对块设备进行读写时作为缓存来使用。这意味着对块的操作会使用 `buffer` 进行缓存，比如我们在格式化文件系统的时候。

但是一般情况下两个缓存系统是一起配合使用的，比如当我们对一个文件进行写操作的时候，`cache` 的内容会被改变，而 `buffer` 则用来将 `cache` 的 `page` 标记为不同的缓冲区，并记录是哪一个缓冲区被修改了。

这样，内核在后续执行脏数据的回写（`writeback`）时，就不用将整个 `page` 写回，而只需要写回修改的部分即可。 

### Cache 的具体职责

`cache` 主要用来作为文件系统上的文件数据的缓存来用，当进程对文件有 `read/write` 操作的时候。包括将文件映射到内存的系统调用 `mmap`，就会用到 `cache`。

因为 `cache` 被作为文件类型的缓存来用，所以事实上也负责了大部分的块设备文件的缓存工作。

### 怎么回收 buff/cache？

Linux 内核会在内存将要耗尽的时候，自动触发内存回收的工作，以便释放出内存给急需内存的进程使用。

***但是这种回收的工作也并不是没有成本。\***

理解 `cache` 是干什么的就知道，`cache` 中存在着一部分 `write` 操作的数据。所以必须保证 `cache` 中的数据跟对应文件中的数据一致，才能对 `cache` 进行释放。

于是伴随着 `cache` 清除的行为的，一般都是系统 `IO` 飙高。这是因为内核要将 `cache` 中缓存的 `write` 数据进行回写。

我们可以使用下面这个文件来人工触发缓存清除的操作，Linux 提供了三种清空方式：

```bash
$ echo 1 > /proc/sys/vm/drop_caches # 仅清除页面缓存
$ echo 2 > /proc/sys/vm/drop_caches # 清除目录项和 inode
$ echo 3 > /proc/sys/vm/drop_caches # 清除页面缓存、目录项以及 inode
```

***但是这种放时只能在执行的当时起作用，过一段时间之后又会发现内存被占满，怎么办呢？\***

实际上内核提供了 `vm.vfs_cache_pressure` 参数用来控制缓冲区的回收频率，我们可以调整它。

这个参数是用来控制内核回收 VFS 缓存的频率。修改这个值会提高或者降低回收 VFS 缓存的频率。值可以设置为任意值。越大回收频率越快，可以把 `vm.vfs_cache_pressure` 赋值为 `500` 来获得最快的回收频率。这个值默认值一般为 `100`。

参考：[https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/8/html/configuring_gfs2_file_systems/con_vfs-tuning-options-gfs2-usage-considerations](https://docs.redhat.com/zh_hans/documentation/red_hat_enterprise_linux/8/html/configuring_gfs2_file_systems/con_vfs-tuning-options-gfs2-usage-considerations)

```bash
$ echo vm.vfs_cache_pressure=500  >> /etc/sysctl.conf 
$ sysctl -p 
```



另外也可以使用 `slabtop` 分析内存使用情况。一般情况下，`dentry` 和 `*_inode_cache` 值越高回收的效果越好。

```bash
$ slabtop -o | grep dentry 
 54222  28096  51%    0.19K   2582       21     10328K dentry                 
$ slabtop -o | grep _inode_cache
 83200  35132  42%    0.06K   1300       64      5200K lsm_inode_cache        
  2520   2369  94%    0.79K    126       20      2016K shmem_inode_cache      
  1826   1092  59%    0.72K     83       22      1328K proc_inode_cache       
  1530   1351  88%    0.94K     45       34      1440K sock_inode_cache       
   320    320 100%    1.00K     10       32       320K mqueue_inode_cache     
   138    138 100%    0.67K      6       23        96K hugetlbfs_inode_cache
```



为什么是 `dentry` 和 `*_inode_cache` 呢，这是因为当读写文件时内核会为该文件对象建立一个 `dentry`，并将其缓存起来，方便下一次读写时直接从内存中取出提高效率。至于 `*_inode_cache` 我就不是很清楚了，只知道是为了加快对索引节点的索引，如果有清楚的可以告诉我一下

