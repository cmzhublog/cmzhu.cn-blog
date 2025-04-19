---
title: strace
description: 
published: true
date: 2023-10-10T01:54:52.269Z
tags: linux
editor: markdown
dateCreated: 2023-10-10T01:54:52.269Z
---

## strace 

跟踪系统调用和信号

### 补充说明

**`strace`命令** 是一个集诊断、调试、统计与一体的工具，我们可以使用`strace`对应用的系统调用和信号传递的跟踪结果来对应用进行分析，以达到解决问题或者是了解应用工作过程的目的。当然`strace`与专业的调试工具比如说`gdb`之类的是没法相比的，因为它不是一个专业的调试器。

`strace`的最简单的用法就是执行一个指定的命令，在指定的命令结束之后它也就退出了。在命令执行的过程中，`strace`会记录和解析命令进程的所有系统调用以及这个进程所接收到的所有的信号值。

### 语法

```bash
strace  [  -dffhiqrtttTvxx  ] [ -acolumn ] [ -eexpr ] ...
    [ -ofile ] [-ppid ] ...  [ -sstrsize ] [ -uusername ]
    [ -Evar=val ] ...  [ -Evar  ]...
     [command [ arg ...  ] ]

strace  -c  [ -eexpr ] ...  [ -Ooverhead ] [ -Ssortby ]
    [ command [ arg...  ] ]
```

### 选项

```bash
-c 统计每一系统调用的所执行的时间,次数和出错的次数等.
-d 输出strace关于标准错误的调试信息.
-f 跟踪由fork调用所产生的子进程.
-ff 如果提供-o filename,则所有进程的跟踪结果输出到相应的filename.pid中,pid是各进程的进程号.
-F 尝试跟踪vfork调用.在-f时,vfork不被跟踪.
-h 输出简要的帮助信息.
-i 输出系统调用的入口指针.
-q 禁止输出关于脱离的消息.
-r 打印出相对时间关于,,每一个系统调用.
-t 在输出中的每一行前加上时间信息.
-tt 在输出中的每一行前加上时间信息,微秒级.
-ttt 微秒级输出,以秒了表示时间.
-T 显示每一调用所耗的时间.
-v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出.
-V 输出strace的版本信息.
-x 以十六进制形式输出非标准字符串
-xx 所有字符串以十六进制形式输出.
-a column 设置返回值的输出位置.默认 为40.
-e expr 指定一个表达式,用来控制如何跟踪.格式：[qualifier=][!]value1[,value2]...
qualifier只能是 trace,abbrev,verbose,raw,signal,read,write其中之一.value是用来限定的符号或数字.默认的 qualifier是 trace.感叹号是否定符号.例如:-eopen等价于 -e trace=open,表示只跟踪open调用.而-etrace!=open 表示跟踪除了open以外的其他调用.有两个特殊的符号 all 和 none. 注意有些shell使用!来执行历史记录里的命令,所以要使用\\.
-e trace=set 只跟踪指定的系统 调用.例如:-e trace=open,close,rean,write表示只跟踪这四个系统调用.默认的为set=all.
-e trace=file 只跟踪有关文件操作的系统调用.
-e trace=process 只跟踪有关进程控制的系统调用.
-e trace=network 跟踪与网络有关的所有系统调用.
-e strace=signal 跟踪所有与系统信号有关的 系统调用
-e trace=ipc 跟踪所有与进程通讯有关的系统调用
-e abbrev=set 设定strace输出的系统调用的结果集.-v 等与 abbrev=none.默认为abbrev=all.
-e raw=set 将指定的系统调用的参数以十六进制显示.
-e signal=set 指定跟踪的系统信号.默认为all.如 signal=!SIGIO(或者signal=!io),表示不跟踪SIGIO信号.
-e read=set 输出从指定文件中读出 的数据.例如: -e read=3,5
-e write=set 输出写入到指定文件中的数据.
-o filename 将strace的输出写入文件filename
-p pid 跟踪指定的进程pid.
-s strsize 指定输出的字符串的最大长度.默认为32.文件名一直全部输出.
-u username 以username的UID和GID执行被跟踪的命令

```

### 实例

#### 追踪系统调用

现在我们做一个很简单的程序来演示strace的基本用法。这个程序的C语言代码如下：

```c
# filename test.c
#include <stdio.h>

int main()
{
    int a;
    scanf("%d", &a);
    printf("%09d\n", a);
    return 0;
}
```

然后我们用`gcc -o test test.c`编译一下，得到一个可执行的文件test。然后用strace调用执行：

```bash
root@cmzhu:~/Test# gcc -o test test.c
root@cmzhu:~/Test# ls
test  test.c
root@cmzhu:~/Test# ls
test  test.c
root@cmzhu:~/Test# ./test
a
000032761
root@cmzhu:~/Test#
```

使用strace 追踪一下调用试试

```bash
strace ./test
```

结果如下, 在 `read(0,` 的时候等待输入, 

```bash
root@cmzhu:~/Test# strace ./test
execve("./test", ["./test"], 0x7ffe4742c170 /* 19 vars */) = 0
brk(NULL)                               = 0x56064189f000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc426cf2000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=28726, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 28726, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fc426cea000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\220s\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=1922136, ...}, AT_EMPTY_PATH) = 0
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 1970000, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fc426b09000
mmap(0x7fc426b2f000, 1396736, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x26000) = 0x7fc426b2f000
mmap(0x7fc426c84000, 339968, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x17b000) = 0x7fc426c84000
mmap(0x7fc426cd7000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1ce000) = 0x7fc426cd7000
mmap(0x7fc426cdd000, 53072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fc426cdd000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc426b06000
arch_prctl(ARCH_SET_FS, 0x7fc426b06740) = 0
set_tid_address(0x7fc426b06a10)         = 15300
set_robust_list(0x7fc426b06a20, 24)     = 0
rseq(0x7fc426b07060, 0x20, 0, 0x53053053) = 0
mprotect(0x7fc426cd7000, 16384, PROT_READ) = 0
mprotect(0x56064163c000, 4096, PROT_READ) = 0
mprotect(0x7fc426d24000, 8192, PROT_READ) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
munmap(0x7fc426cea000, 28726)           = 0
newfstatat(0, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}, AT_EMPTY_PATH) = 0
getrandom("\xa3\x9a\x16\x07\xa6\x76\xb5\x2b", 8, GRND_NONBLOCK) = 8
brk(NULL)                               = 0x56064189f000
brk(0x5606418c0000)                     = 0x5606418c0000
read(0, a
"a\n", 1024)                    = 2
newfstatat(1, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}, AT_EMPTY_PATH) = 0
write(1, "000032708\n", 10000032708
)             = 10
lseek(0, -2, SEEK_CUR)                  = -1 ESPIPE (Illegal seek)
exit_group(0)                           = ?
+++ exited with 0 +++
root@cmzhu:~/Test#
```

从`trace`结构可以看到，系统首先调用`execve`开始一个新的进行，接着进行些环境的初始化操作，最后停顿在”read(0,”上面，这也就是执行到了我们的`scanf`函数，等待我们输入数字呢，在输入完`a`之后，在调用write函数将格式化后的数值`000032708`输出到屏幕，最后调用`exit_group`退出进行，完成整个程序的执行过程。

#### 跟踪信号传递

我们还是使用上面的那个test程序，来观察进程接收信号的情况。还是先`strace ./test`，等到等待输入的画面的时候不要输入任何东西，然后打开另外一个窗口，输入如下的命令

```bash
root@cmzhu:~/Test# ps -ef | grep test
root       15613   12422  0 02:51 pts/1    00:00:00 strace ./test
root       15616   15613  0 02:51 pts/1    00:00:00 ./test
root       15644   15629  0 02:52 pts/3    00:00:00 grep test
```

```bash
kill -9 15616

brk(0x55eb4e111000)                     = 0x55eb4e111000
read(0,  <unfinished ...>)              = ?
+++ killed by SIGKILL +++
Killed
```

#### 系统调用统计

`strace`不光能追踪系统调用，通过使用参数`-c`，它还能将进程所有的系统调用做一个统计分析给你，下面就来看看`strace`的统计，这次我们执行带`-c`参数的`strace`：

```bash
strace -c ./test
```

```bash
root@cmzhu:~/Test# strace -c ./test
abc
000032734
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0         2           read
  0.00    0.000000           0         1           write
  0.00    0.000000           0         2           close
  0.00    0.000000           0         1         1 lseek
  0.00    0.000000           0         8           mmap
  0.00    0.000000           0         3           mprotect
  0.00    0.000000           0         1           munmap
  0.00    0.000000           0         3           brk
  0.00    0.000000           0         2           pread64
  0.00    0.000000           0         1         1 access
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         1           arch_prctl
  0.00    0.000000           0         1           set_tid_address
  0.00    0.000000           0         2           openat
  0.00    0.000000           0         4           newfstatat
  0.00    0.000000           0         1           set_robust_list
  0.00    0.000000           0         1           prlimit64
  0.00    0.000000           0         1           getrandom
  0.00    0.000000           0         1           rseq
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000           0        37         2 total
```



这里很清楚的告诉你调用了那些系统函数，调用次数多少，消耗了多少时间等等这些信息，这个对我们分析一个程序来说是非常有用的。

### 常用参数说明

除了-c参数之外，strace还提供了其他有用的参数给我们，让我们能很方便的得到自己想要的信息，下面就对那些常用的参数一一做个介绍。

#### 重定向输出

参数-o用在将strace的结果输出到文件中，如果不指定-o参数的话，默认的输出设备是STDERR，也就是说使用”-o filename”和” 2>filename”的结果是一样的。



```bash
# 这两个命令都是将strace结果输出到文件test.txt中
strace -c -o test.txt ./test
strace -c ./test  2>test.txt

```

#### 对系统调用进行计时

strace可以使用参数-T将每个系统调用所花费的时间打印出来，每个调用的时间花销现在在调用行最右边的尖括号里面。

```bash
strace -T ./test
```

```bash
root@cmzhu:~/Test# strace -T ./test
execve("./test", ["./test"], 0x7fffaee70d58 /* 19 vars */) = 0 <0.000522>
brk(NULL)                               = 0x55a51e58f000 <0.000069>
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f6e374ba000 <0.000059>
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory) <0.000054>
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3 <0.000045>
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=28726, ...}, AT_EMPTY_PATH) = 0 <0.000046>
mmap(NULL, 28726, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f6e374b2000 <0.000066>
close(3)                                = 0 <0.000042>
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3 <0.000045>
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\220s\2\0\0\0\0\0"..., 832) = 832 <0.000045>
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784 <0.000072>
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=1922136, ...}, AT_EMPTY_PATH) = 0 <0.000043>
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784 <0.000038>
mmap(NULL, 1970000, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f6e372d1000 <0.000065>
mmap(0x7f6e372f7000, 1396736, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x26000) = 0x7f6e372f7000 <0.000072>
mmap(0x7f6e3744c000, 339968, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x17b000) = 0x7f6e3744c000 <0.000097>
mmap(0x7f6e3749f000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1ce000) = 0x7f6e3749f000 <0.000068>
mmap(0x7f6e374a5000, 53072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f6e374a5000 <0.000063>
close(3)                                = 0 <0.000035>
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f6e372ce000 <0.000062>
arch_prctl(ARCH_SET_FS, 0x7f6e372ce740) = 0 <0.000030>
set_tid_address(0x7f6e372cea10)         = 16751 <0.000047>
set_robust_list(0x7f6e372cea20, 24)     = 0 <0.000076>
rseq(0x7f6e372cf060, 0x20, 0, 0x53053053) = 0 <0.000033>
mprotect(0x7f6e3749f000, 16384, PROT_READ) = 0 <0.000061>
mprotect(0x55a51c7c4000, 4096, PROT_READ) = 0 <0.000037>
mprotect(0x7f6e374ec000, 8192, PROT_READ) = 0 <0.000046>
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0 <0.000031>
munmap(0x7f6e374b2000, 28726)           = 0 <0.000078>
newfstatat(0, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}, AT_EMPTY_PATH) = 0 <0.000075>
getrandom("\xd1\x63\xf3\xf2\xdf\x35\x31\x4a", 8, GRND_NONBLOCK) = 8 <0.000045>
brk(NULL)                               = 0x55a51e58f000 <0.000052>
brk(0x55a51e5b0000)                     = 0x55a51e5b0000 <0.000043>
read(0, test
"test\n", 1024)                 = 5 <12.363778>
newfstatat(1, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}, AT_EMPTY_PATH) = 0 <0.000145>
write(1, "000032622\n", 10000032622
)             = 10 <0.000092>
lseek(0, -5, SEEK_CUR)                  = -1 ESPIPE (Illegal seek) <0.000044>
exit_group(0)                           = ?
+++ exited with 0 +++
root@cmzhu:~/Test#
```

#### 系统调用的时间

这是一个很有用的功能，strace会将每次系统调用的发生时间记录下来，只要使用-t/tt/ttt三个参数就可以看到效果了，具体的例子可以自己去尝试。

| 参数名 | 输出样式                        | 说明                                 |
| ------ | ------------------------------- | ------------------------------------ |
| -t     | 10:33:04 exit_group(0)          | 输出结果精确到秒                     |
| -tt    | 10:33:48.159682 exit_group(0)   | 输出结果精确到微妙                   |
| -ttt   | 1262169244.788478 exit_group(0) | 精确到微妙，而且时间表示为unix时间戳 |

```bash
root@cmzhu:~/Test# strace -t test
21:45:03 execve("/usr/bin/test", ["test"], 0x7ffd57bb84d8 /* 19 vars */) = 0
21:45:03 brk(NULL)                      = 0x557cf1676000
21:45:03 mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fa34ea96000
21:45:03 access("/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory)
21:45:03 openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
21:45:03 newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=28726, ...}, AT_EMPTY_PATH) = 0
21:45:03 mmap(NULL, 28726, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fa34ea8e000
21:45:03 close(3)                       = 0
21:45:03 openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
21:45:03 read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\220s\2\0\0\0\0\0"..., 832) = 832
21:45:03 pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
21:45:03 newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=1922136, ...}, AT_EMPTY_PATH) = 0
21:45:03 pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
21:45:03 mmap(NULL, 1970000, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fa34e8ad000
21:45:03 mmap(0x7fa34e8d3000, 1396736, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x26000) = 0x7fa34e8d3000
21:45:03 mmap(0x7fa34ea28000, 339968, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x17b000) = 0x7fa34ea28000
21:45:03 mmap(0x7fa34ea7b000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1ce000) = 0x7fa34ea7b000
21:45:03 mmap(0x7fa34ea81000, 53072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fa34ea81000
21:45:03 close(3)                       = 0
21:45:03 mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fa34e8aa000
21:45:03 arch_prctl(ARCH_SET_FS, 0x7fa34e8aa740) = 0
21:45:03 set_tid_address(0x7fa34e8aaa10) = 30538
21:45:03 set_robust_list(0x7fa34e8aaa20, 24) = 0
21:45:03 rseq(0x7fa34e8ab060, 0x20, 0, 0x53053053) = 0
21:45:03 mprotect(0x7fa34ea7b000, 16384, PROT_READ) = 0
21:45:03 mprotect(0x557cf0683000, 4096, PROT_READ) = 0
21:45:03 mprotect(0x7fa34eac8000, 8192, PROT_READ) = 0
21:45:03 prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
21:45:03 munmap(0x7fa34ea8e000, 28726)  = 0
21:45:03 getrandom("\x12\xc2\x5b\x42\xb5\xf1\x0e\x1a", 8, GRND_NONBLOCK) = 8
21:45:03 brk(NULL)                      = 0x557cf1676000
21:45:03 brk(0x557cf1697000)            = 0x557cf1697000
21:45:03 openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
21:45:03 newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=3048928, ...}, AT_EMPTY_PATH) = 0
21:45:03 mmap(NULL, 3048928, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fa34e400000
21:45:03 close(3)                       = 0
21:45:03 close(1)                       = 0
21:45:03 close(2)                       = 0
21:45:03 exit_group(1)                  = ?
21:45:03 +++ exited with 1 +++
```

#### 截断输出

-s参数用于指定trace结果的每一行输出的字符串的长度，下面看看test程序中-s参数对结果有什么影响，现指定-s为20，然后在read的是是很我们输入一个超过20个字符的数字串

```bash
strace -s 20 ./test
```

```bash
read(0, 222222222222222222222222222222222222222222222222222222
"2222222222"..., 1024)          = 55
```

#### trace一个现有的进程

strace不光能自己初始化一个进程进行trace，还能追踪现有的进程，参数-p就是取这个作用的，用法也很简单，具体如下。

```bash
strace -p pid
```

