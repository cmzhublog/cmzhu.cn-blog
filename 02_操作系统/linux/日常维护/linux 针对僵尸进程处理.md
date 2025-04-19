## 处理僵尸进程

1、如何确定系统存在僵尸进程,其中zombie 代表僵尸进程，发现这前面不是0 时，代表有僵尸进程

```bash
$ top -b -n 1 | head -10 
top - 22:23:23 up 11 days,  9:57,  2 users,  load average: 41.24, 41.80, 41.72
Tasks: 570 total,   1 running, 569 sleeping,   0 stopped,   0 zombie
%Cpu(s): 11.1 us,  7.4 sy,  0.0 ni, 70.4 id, 11.1 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :  15908.5 total,    920.0 free,  11310.6 used,   4256.9 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   4597.9 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
  50928 root      20   0   12980   6036   3924 R  21.1   0.0   0:00.07 top
   3179 root      20   0 2391064   1.3g  42116 S  10.5   8.3     53,24 kube-ap+
   3260 root      20   0   10.8g 146004  39420 S  10.5   0.9     25,36 etcd
```

2、处理僵尸进程，一般就是找到僵尸进程并kill 掉

```bash
$ ps -e -o stat,ppid,pid,cmd | egrep -i '^z'
```

在kill 进程时，最好先看下进程是什么，排查进程出现的原因