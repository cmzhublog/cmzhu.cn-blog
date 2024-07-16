## 清理linux 内存中的buffer 数据



```bash
$  echo <num> >/proc/sys/vm/drop_caches
```

num 可以为1-3 的数字， 不同数字分别代表不同的涵义

- 0：不释放（系统默认值）
- 1：释放页缓存
- 2：释放dentries和inodes

