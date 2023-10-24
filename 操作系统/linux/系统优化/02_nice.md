

# linux 系统优化

## 改变进程优先级(nice 和renice)

什么是进程优先级?

当 Linux 内核尝试决定哪些运行中的进程可以访问 CPU 时，其中一个需要考虑的因素就是进程优先级的值（也称为 nice 值）。每个进程都有一个介于 -20 到 19 之间的 nice 值。默认情况下，进程的 nice 值为 0。

### nice 命令

使用ps 命令可以查看nice 值

```bash
ps -le 
```



nice命令可以在进程运行时赋予进程优先级值(NI)

```bash
nice [-n NI值] 命令
```

-n NI值：给命令赋予 NI 值，该值的范围为 -20~19；

例子:

```bash
nice -n -20 bash test.py
```



### renice

renice 可以在进程运行时, 进程优先级;

```bash
renice < 进程优先级 > < 进程号 >
```

例子:

```bash
renice -20 2120
```

