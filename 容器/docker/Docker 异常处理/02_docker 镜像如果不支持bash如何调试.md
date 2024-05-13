## docker 容器的另一种调试方法(nsenter)

### 语法

下面是nsenter 的语法

```bash
$ nsenter [options] [program [arguments]]

options:

-a, --all enter all namespaces of the target process by the default /proc/[pid]/ns/* namespace paths.
-m, --mount[=<file>]：进入 mount 命令空间。如果指定了 file，则进入 file 的命名空间
-u, --uts[=<file>]：进入 UTS 命名空间。如果指定了 file，则进入 file 的命名空间
-i, --ipc[=<file>]：进入 System V IPC 命名空间。如果指定了 file，则进入 file 的命名空间
-n, --net[=<file>]：进入 net 命名空间。如果指定了 file，则进入 file 的命名空间
-p, --pid[=<file>：进入 pid 命名空间。如果指定了 file，则进入 file 的命名空间
-U, --user[=<file>：进入 user 命名空间。如果指定了 file，则进入 file 的命名空间
-t, --target <pid> # 指定被进入命名空间的目标进程的 pid
-G, --setgid gid：设置运行程序的 GID
-S, --setuid uid：设置运行程序的 UID
-r, --root[=directory]：设置根目录
-w, --wd[=directory]：设置工作目录
```



例子

**使用nsenter 进入容器的终端**

1、 获取容器的PID , 使用nsenter 调试容器, 必须要获取容器的pid

```bash
# 使用 nerdctl 获取containerd 容器的Pid
$ PID=$(nerdctl -n k8s.io  inspect  c2433b20bbaf | jq .[0].State.Pid)
624496

# 使用docker 获取容器的pid
$ PID=$(docker inspect --format {{.State.Pid}} c2433b20bbaf)
```

2、 使用nsenter 调试容器

```bash
$ nsenter -m -u -i -n -p -t $PID <command>
```

