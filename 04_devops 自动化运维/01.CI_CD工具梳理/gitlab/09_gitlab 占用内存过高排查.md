## gitlab 占用内存过高排查

1、 使用top 命令排查操作系统中使用内存最多的进程, 不难发现， 其中除了mongo和java 进程外， 就属于bundle 进程占用的内存最多，随便拿一个进程出来查看详细信息

> top + M(top 之后输入大写的M)

```bash
$ top
top - 09:58:36 up 24 days, 19:49,  3 users,  load average: 0.42, 0.40, 0.40
Tasks: 752 total,   3 running, 749 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.3 sy,  0.0 ni, 98.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 65452408 total,  6542512 free, 41964956 used, 16944940 buff/cache
KiB Swap: 32833532 total, 22398156 free, 10435376 used. 22425068 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18167 polkitd   20   0   15.0g  11.2g  44612 S   1.3 17.9 151:32.48 mongod
24343 root      20   0   34.5g   4.5g  22896 S   0.7  7.3  74:02.20 java
13322 3000      20   0   21.0g   1.3g  67776 S   9.6  2.1   1492:33 java
16127 libstor+  20   0 3144000   1.2g  10364 S   0.3  1.8   2690:07 bundle
19750 3000      20   0 8014184   1.0g   5328 S   9.9  1.6   3663:12 java
16872 libstor+  20   0 2262712 983.6m   6712 S   0.0  1.5  52:59.42 bundle
16854 libstor+  20   0 2281896 982.0m   6780 S   0.0  1.5  49:11.86 bundle
16874 libstor+  20   0 2319484 997504   6744 S   0.3  1.5  54:48.88 bundle
16856 libstor+  20   0 2289560 984248   6740 S   0.3  1.5  51:58.10 bundle
16826 libstor+  20   0 2286932 969724   6708 S   0.3  1.5  56:58.93 bundle
16822 libstor+  20   0 2324176 969532   6968 S   0.0  1.5  52:22.46 bundle
16838 libstor+  20   0 2242776 968920   6560 S   0.3  1.5  49:45.16 bundle
16848 libstor+  20   0 2327236 959200   6668 S   0.0  1.5  56:33.53 bundle
16842 libstor+  20   0 2355120 946112   6916 S   0.3  1.4  53:29.03 bundle
16864 libstor+  20   0 2227912 943408   6784 S   0.3  1.4  49:58.24 bundle
16832 libstor+  20   0 2381788 935984   6948 S   0.3  1.4  53:02.63 bundle
16850 libstor+  20   0 2319600 934500   6832 S   0.0  1.4  52:13.95 bundle
16868 libstor+  20   0 2244820 932508   6688 S   0.0  1.4  46:49.48 bundle
16824 libstor+  20   0 2281768 913308   6756 S   0.0  1.4  52:55.04 bundle
16870 libstor+  20   0 2275016 899844   6600 S   0.0  1.4  48:30.13 bundle
16840 libstor+  20   0 2288456 894524   6660 S   0.3  1.4  51:46.02 bundle
15865 libstor+  20   0  950612 631712   4024 S   0.3  1.0  81:09.70 bundle
 2846 root      20   0   30.2g 625396  14732 S   0.0  1.0   3:24.53 java
16858 libstor+  20   0 2141336 577268   6580 S   0.0  0.9  49:05.52 bundle
16836 libstor+  20   0 2266384 568368   6660 S   0.0  0.9  51:08.64 bundle
16820 libstor+  20   0 2344680 561860   6892 S   0.0  0.9  57:20.38 bundle
16834 libstor+  20   0 2438820 560268   6636 S   0.0  0.9  56:05.37 bundle
16830 libstor+  20   0 2257944 559620   6720 S   0.0  0.9  51:13.87 bundle
16866 libstor+  20   0 2308568 557944   6628 S   0.0  0.9  50:49.88 bundle
16844 libstor+  20   0 2313268 548840   6668 S   0.0  0.8  52:32.21 bundle
16862 libstor+  20   0 2312360 548752   6692 S   0.0  0.8  52:46.76 bundle
16880 libstor+  20   0 2285528 547848   6884 S   0.0  0.8  47:27.69 bundle
16828 libstor+  20   0 2319124 542720   6776 S   0.0  0.8  61:01.00 bundle
16846 libstor+  20   0 2314220 539168   6932 S   0.0  0.8  51:54.55 bundle
16818 libstor+  20   0 2382440 534812   6756 S   0.0  0.8  51:52.04 bundle
16876 libstor+  20   0 2320524 530564   6712 S   0.0  0.8  52:56.33 bundle
16878 libstor+  20   0 2397660 521856   6860 S   0.3  0.8  58:23.09 bundle
16852 libstor+  20   0 2281528 516956   6664 S   0.0  0.8  48:27.68 bundle
16860 libstor+  20   0 2334140 512640   6904 S   0.3  0.8  58:04.05 bundle
```

2、使用ps -ef  查看进程详细信息, 发现是puma 进程，参考官方文档，是gitlab 的puma 占用的内存；

```bash
$ ps -ef | grep 16858
libstor+ 16858 15865  0 6月29 ?       00:49:05 puma: cluster worker 20: 360 [gitlab-puma-worker]
root     28798 12803  0 09:58 pts/0    00:00:00 grep --color=auto 16858
```

3、修复方案

1、 根据实际情况调整puma 的进程数量,worker_processes =2 

```bash
puma['worker_processes'] = 2
puma['min_threads'] = 4
puma['max_threads'] = 4
```



参考文档：[https://docs.gitlab.cn/jh/administration/operations/puma.html#reducing-memory-use](https://docs.gitlab.cn/jh/administration/operations/puma.html#reducing-memory-use)

