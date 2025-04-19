## 固定内核或者指定软件版本

#### 方法 5 - 阻止特定版本的包（使用版本锁插件）

Versionlock 是 Yum 包管理器的插件。此插件不允许将软件包升级到比执行锁定时安装的版本更高的版本。

首先，安装版本锁。

```
$ sudo dnf install dnf-plugin-versionlock
or
$ sudo yum install dnf-plugin-versionlock
```

这还将在您的系统上创建一个文件 `/etc/yum/pluginconf.d/versionlock.list`。

要锁定系统上安装的当前版本的 `kernel.*`，请运行以下命令。

```
$ sudo dnf versionlock kernel.*
or
$ sudo yum versionlock kernel.*
```

您将获得类似的输出。

```
Last metadata expiration check: 0:01:05 ago on Mon 05 Dec 2022 12:14:16 PM UTC.
Adding versionlock on: kernel.*-3:10.3.35-1.module+el8.6.0+1005+cdf19c22.*
```

您可以一次添加多个包。

```
$ sudo dnf versionlock evolution golang
or
$ sudo yum versionlock evolution golang
```

您将获得类似的输出。

```
Last metadata expiration check: 0:01:05 ago on Mon 05 Dec 2022 12:14:16 PM UTC.
Adding versionlock on: evolution-0:3.28.5-18.el8.*
Adding versionlock on: golang-0:1.18.4-1.module+el8.7.0+1073+99e3b3cd.*
```

让我们尝试更新 `kernel.*` 包。

```
$ sudo dnf update kernel.*
or
$ sudo yum update kernel.*
```

您将获得类似的输出。

```
Last metadata expiration check: 0:02:07 ago on Mon 05 Dec 2022 12:14:16 PM UTC.
Package kernel.* available, but not installed.
No match for argument: kernel.*
Error: No packages marked for upgrade.
```

要通过 versionlock 插件检查被阻止的包列表，请使用以下命令。

```
$ dnf versionlock list
or
$ yum versionlock list
```

您将获得类似的输出。

```
Last metadata expiration check: 0:00:05 ago on Wed 07 Dec 2022 02:36:20 AM UTC.
elasticsearch-7.17.5-1.x86_64
kernel.*-3:10.3.35-1.module+el8.6.0+1005+cdf19c22.*
evolution-0:3.28.5-18.el8.*
golang-0:1.18.4-1.module+el8.7.0+1073+99e3b3cd.*
```

要从版本锁中删除包，请使用以下命令。

```
$ sudo dnf versionlock delete kernel.*
or
$ sudo yum versionlock delete kernel.*
```

您将获得以下输出。

```
Deleting versionlock for: kernel.*-3:10.3.35-1.module+el8.6.0+1005+cdf19c22.*
```

要丢弃列表并清除块，请使用以下命令。

```
$ sudo dnf versionlock clear
or
$ sudo yum versionlock clear
```

或者，您可以编辑文件 `/etc/yum/pluginconf.d/versionlock.list` 以阻止使用版本锁插件的包。

要将已安装的包添加到文件中，请使用以下命令。

```
$ sudo sh -c 'rpm -qa | grep evolution >> /etc/yum/pluginconf.d/versionlock.list'
```

上面的命令通过将 `evolution` 包添加到列表来阻止它。我们使用了 `rpm -qa | grep evolution` 获取完整的包名。和

`sudo sh -c` 命令运行一个 sudo shell，在该 shell 下运行写入文件的命令。