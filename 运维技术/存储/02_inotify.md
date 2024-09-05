## Inotify

​	`	inotify` 是 Linux 内核提供的一种文件系统事件监控机制，可以用于监视文件和目录的变化。它允许应用程序实时跟踪文件的创建、删除、修改和其他事件，而无需循环查询文件状态，从而提高了效率和性能

- **实时监控**：`inotify` 提供对文件和目录的实时监控，能够及时捕捉到文件系统的变化
- **事件通知**：当监控的文件或目录发生变化时，`inotify` 会产生相应的事件通知，应用程序可以快速响应
- **低开销**：与传统的轮询机制相比，`inotify` 的资源开销较低，能够更高效地利用系统资源

使用 `inotify` 监控的事件包括：

- `IN_MODIFY`：文件被修改
- `IN_CREATE`：文件或目录被创建
- `IN_DELETE`：文件或目录被删除
- `IN_MOVE`：文件或目录被移动
- `IN_OPEN`：文件被打开
- `IN_CLOSE`：文件被关闭

### 如何使用

`inotify` 的使用可以通过命令行工具 `inotifywait` 和 `inotifywatch`，或者通过编程（如 C/C++、Python 等）实现

1、 安装 `inotify-tools`

```bash
# 在 Debian/Ubuntu 上安装
$ sudo apt-get install inotify-tools

# 在 CentOS/RHEL 上安装
$ sudo yum install inotify-tools

# 在Rocky Linux 上安装
$ sudo dnf install inotify-tools -y
```

2、`inotifywait` 是一个命令行工具，可以用于等待文件系统事件并输出信息。例如，监视某个目录的文件创建和修改事件

```bash
$ inotifywait -m /path/to/directory -e create -e modify
```

- `-m`：持续监视。
- `-e`：指定要监视的事件类型

3、`inotifywatch` 也可以用于统计文件系统事件。例如，监视某个目录一段时间内的事件数量

```bash
$ inotifywatch -v -r /path/to/directory
```

- `-v`：显示详细信息
- `-r`：递归监视子目录

`inotify` 是一个强大的工具，用于实时监控文件系统的变化。无论是通过命令行工具还是编程接口，都可以有效地跟踪文件和目录的变化，从而实现高效的文件处理和监控

### 例子

使用`inotify` + `rsync` 实现实时同步文件夹的数据 , 脚本 `inotify_rsync_backup.sh`

```bash
#!/bin/bash
SRC='/data/www'
DEST='rsyncuser@rsync服务器IP::backup'
rpm -q rsync &> /dev/null || yum -y install rsync

inotifywait -mrq --exclude=".*\.swp" --timefmt '%Y-%m-%d %H:%M:%S' --format '%T %w %f' -e create,delete,moved_to,close_write,attrib ${SRC} | while read DATE TIME DIR FILE; do
        FILEPATH=${DIR}${FILE}
        if rsync -az --delete --password-file=/etc/rsync.pass "$SRC" "$DEST"; then
            echo "At ${TIME} on ${DATE}, file $FILEPATH was backed up via rsync" >>var/log/changelist.log
        else
            echo "Backup failed for $FILEPATH at ${TIME} on ${DATE}" >> /var/log/changelist.log
        fi
done
```

主要实现：

- 检查是否安装了 `rsync`，如果未安装，则通过 `yum` 安装

- 使用inotifywait监控SRC目录的变化
  - `-m`：持续监控
  - `-r`：递归监控子目录
  - `-q`：安静模式，只输出事件
  - `--exclude`：排除特定文件（如 `.swp` 文件）
  - `--timefmt` 和 `--format`：格式化输出时间和文件信息
  - 监控的事件包括：创建、删除、移动到、关闭写入和属性变化
- 使用 `rsync` 将文件从源目录备份到目标目录
  - `-a`：归档模式，表示递归复制并保持所有文件属性
  - `-z`：在传输过程中压缩文件
  - `--delete`：在目标中删除源目录中不存在的文件
  - `--password-file`：指定存储 rsync 密码的文件