---
title: zstd用法
description: 
published: true
date: 2023-08-18T07:24:36.751Z
tags: linux
editor: markdown
dateCreated: 2023-08-18T07:22:21.737Z
---

## zstd 压缩工具

> github 地址: https://github.com/facebook/zstd

表示 `zstd` 和 其他几项压缩工具的对比, 其中, `zstd` 压缩率最高, 压缩和解压时间靠前时间

| 压缩工具                             | 压缩率 | 压缩速度 | 解压速度  |
| ------------------------------------ | ------ | -------- | --------- |
| **zstd 1.5.1 -1**              | 2.887  | 530 MB/s | 1700 MB/s |
| [zlib](https://www.zlib.net/) 1.2.11 -1 | 2.743  | 95 MB/s  | 400 MB/s  |
| brotli 1.0.9 -0                      | 2.702  | 395 MB/s | 450 MB/s  |
| **zstd 1.5.1 --fast=1**        | 2.437  | 600 MB/s | 2150 MB/s |
| **zstd 1.5.1 --fast=3**        | 2.239  | 670 MB/s | 2250 MB/s |
| quicklz 1.5.0 -1                     | 2.238  | 540 MB/s | 760 MB/s  |
| **zstd 1.5.1 --fast=4**        | 2.148  | 710 MB/s | 2300 MB/s |
| lzo1x 2.10 -1                        | 2.106  | 660 MB/s | 845 MB/s  |
| [lz4](https://lz4.github.io/lz4/) 1.9.3 | 2.101  | 740 MB/s | 4500 MB/s |
| lzf 3.6 -1                           | 2.077  | 410 MB/s | 830 MB/s  |
| snappy 1.1.9                         | 2.073  | 550 MB/s | 1750 MB/s |

### `zstd` 用法

1、以下操作均在 `zstd` 版本为 1.5.1 下进行, 其余版本可能会有所出入

```bash
[root@cmzhu ~]# zstd --version
*** zstd command line interface 64-bits v1.5.1, by Yann Collet ***

```

2、 压缩命令相关配置

```bash
[root@cmzhu ~]# zstd --help
*** zstd command line interface 64-bits v1.5.1, by Yann Collet ***
Usage :
      zstd [args] [FILE(s)] [-o file]

FILE    : a filename
          with no FILE, or when FILE is - , read standard input
Arguments :
 -#     : # compression level (1-19, default: 3)
 -d     : decompression
 -D DICT: use DICT as Dictionary for compression or decompression
 -o file: result stored into `file` (only 1 output file)
 -f     : disable input and output checks. Allows overwriting existing files,
          input from console, output to stdout, operating on links,
          block devices, etc.
--rm    : remove source file(s) after successful de/compression
 -k     : preserve source file(s) (default)
 -h/-H  : display help/long help and exit

Advanced arguments :
 -V     : display Version number and exit
 -c     : write to standard output (even if it is the console)
 -v     : verbose mode; specify multiple times to increase verbosity
 -q     : suppress warnings; specify twice to suppress errors too
--[no-]progress : forcibly display, or never display the progress counter.
                  note: any (de)compressed output to terminal will mix with progress counter text.
 -r     : operate recursively on directories
--filelist FILE : read list of files to operate upon from FILE
--output-dir-flat DIR : processed files are stored into DIR
--output-dir-mirror DIR : processed files are stored into DIR respecting original directory structure
--[no-]check : during compression, add XXH64 integrity checksum to frame (default: enabled). If specified with -d, decompressor will ignore/validate checksums in compressed frame (default: validate).
--trace FILE : log tracing information to FILE.
--      : All arguments after "--" are treated as files

Advanced compression arguments :
--ultra : enable levels beyond 19, up to 22 (requires more memory)
--long[=#]: enable long distance matching with given window log (default: 27)
--fast[=#]: switch to very fast compression levels (default: 1)
--adapt : dynamically adapt compression level to I/O conditions
--[no-]row-match-finder : force enable/disable usage of fast row-based matchfinder for greedy, lazy, and lazy2 strategies
--patch-from=FILE : specify the file to be used as a reference point for zstd's diff engine.
 -T#    : spawns # compression threads (default: 1, 0==# cores)
 -B#    : select size of each job (default: 0==automatic)
--single-thread : use a single thread for both I/O and compression (result slightly different than -T1)
--auto-threads={physical,logical} (default: physical} : use either physical cores or logical cores as default when specifying -T0
--rsyncable : compress using a rsync-friendly method (-B sets block size)
--exclude-compressed: only compress files that are not already compressed
--stream-size=# : specify size of streaming input from `stdin`
--size-hint=# optimize compression parameters for streaming input of approximately this size
--target-compressed-block-size=# : generate compressed block of approximately targeted size
--no-dictID : don't write dictID into header (dictionary compression only)
--[no-]compress-literals : force (un)compressed literals
--format=zstd : compress files to the .zst format (default)
--format=gzip : compress files to the .gz format
--format=xz : compress files to the .xz format
--format=lzma : compress files to the .lzma format
--format=lz4 : compress files to the .lz4 format

Advanced decompression arguments :
 -l     : print information about zstd compressed files
--test  : test compressed file integrity
 -M#    : Set a memory usage limit for decompression
--[no-]sparse : sparse mode (default: enabled on file, disabled on stdout)

Dictionary builder :
--train ## : create a dictionary from a training set of files
--train-cover[=k=#,d=#,steps=#,split=#,shrink[=#]] : use the cover algorithm with optional args
--train-fastcover[=k=#,d=#,f=#,steps=#,split=#,accel=#,shrink[=#]] : use the fast cover algorithm with optional args
--train-legacy[=s=#] : use the legacy algorithm with selectivity (default: 9)
 -o DICT : DICT is dictionary name (default: dictionary)
--maxdict=# : limit dictionary to specified size (default: 112640)
--dictID=# : force dictionary ID to specified value (default: random)

Benchmark arguments :
 -b#    : benchmark file(s), using # compression level (default: 3)
 -e#    : test all compression levels successively from -b# to -e# (default: 1)
 -i#    : minimum evaluation time in seconds (default: 3s)
 -B#    : cut file into independent blocks of size # (default: no block)
 -S     : output one benchmark result per input file (default: consolidated result)
--priority=rt : set process priority to real-time
```

3、 `zstd` 相关命令介绍
创建压缩字典

```bash
zstd --train FullPathToTrainingSet/* -o dictionaryName
```

压缩字典

```bash
zstd -D dictionaryName FILE
```

解压字典

```bash
zstd -D dictionaryName --decompress FILE.zst
```

4、压缩单个文件

压缩文件, 并使用 time 记录耗时

```
bash

[root@cmzhu ~]# time zstd test
test                 :  0.00%   (  10.0 GiB =>    329 KiB, test.zst)

real	0m3.927s
user	0m4.282s
sys	0m1.225s
[root@cmzhu ~]#
```

解压文件

```
bash
[root@cmzhu ~]# time zstd -d test.zst
test.zst            : 10737418240 bytes

real	0m2.023s
user	0m2.015s
sys	0m0.003s
[root@cmzhu ~]#
```

5、压缩多个文件, 在压缩多个文件时需要先打包,在压缩

```
bash
[root@cmzhu ~]# time $(tar -cvf - kubekey/ | zstd  -o kubekey1.zst)
real	0m4.749s
user	0m4.760s
sys	0m1.155s
[root@cmzhu ~]#
```

### `zstd` 相关压缩配置对比



在压缩部分重复率足够大时, 可以使用 `--long `于提高窗口大小,适用 了之后会将轮训窗口增大为 ` 128MIB`,  从下图也可以看懂 , 添加了  `--long` 配置的压缩, 无论压缩率还是压缩时间, 都有巨大的提高

| 压缩方式            |     压缩率 |       压缩速度 |        解压速度 |
| :------------------ | ---------: | -------------: | --------------: |
| `zstd -1`         |  `5.065` | `284.8 MB/s` |  `759.3 MB/s` |
| `zstd -5`         |  `5.826` | `124.9 MB/s` |  `674.0 MB/s` |
| `zstd -10`        |  `6.504` |  `29.5 MB/s` |  `771.3 MB/s` |
| `zstd -1 --long`  | `17.426` | `220.6 MB/s` | `1638.4 MB/s` |
| `zstd -5 --long`  | `19.661` | `165.5 MB/s` | `1530.6 MB/s` |
| `zstd -10 --long` | `21.949` |  `75.6 MB/s` | `1632.6 MB/s` |

还有一种极端, 就是重复率并不适用于长窗口压缩时,这种提高了窗口查询时就会降低性能


| 压缩方式            |    压缩率 |       压缩速度 |       解压速度 |
| :------------------ | --------: | -------------: | -------------: |
| `zstd -1`         | `2.878` | `231.7 MB/s` | `594.4 MB/s` |
| `zstd -1 --long`  | `2.929` | `106.5 MB/s` | `517.9 MB/s` |
| `zstd -5`         | `3.274` |  `77.1 MB/s` | `464.2 MB/s` |
| `zstd -5 --long`  | `3.319` |  `51.7 MB/s` | `371.9 MB/s` |
| `zstd -10`        | `3.523` |  `16.4 MB/s` | `489.2 MB/s` |
| `zstd -10 --long` | `3.566` |  `16.2 MB/s` | `415.7 MB/s` |