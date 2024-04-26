## dd 命令

Linux dd 命令用于读取、转换并输出数据。

dd 可从标准输入或文件中读取数据，根据指定的格式来转换数据，再输出到文件、设备或标准输出。

**参数说明:**

- if=文件名：输入文件名，默认为标准输入。即指定源文件。
- of=文件名：输出文件名，默认为标准输出。即指定目的文件。
- ibs=bytes：一次读入bytes个字节，即指定一个块大小为bytes个字节。
  obs=bytes：一次输出bytes个字节，即指定一个块大小为bytes个字节。
  bs=bytes：同时设置读入/输出的块大小为bytes个字节。
- cbs=bytes：一次转换bytes个字节，即指定转换缓冲区大小。
- skip=blocks：从输入文件开头跳过blocks个块后再开始复制。
- seek=blocks：从输出文件开头跳过blocks个块后再开始复制。
- count=blocks：仅拷贝blocks个块，块大小等于ibs指定的字节数。
- conv=<关键字>，关键字可以有以下11种：
  - conversion：用指定的参数转换文件。
  - ascii：转换ebcdic为ascii
  - ebcdic：转换ascii为ebcdic
  - ibm：转换ascii为alternate ebcdic
  - block：把每一行转换为长度为cbs，不足部分用空格填充
  - unblock：使每一行的长度都为cbs，不足部分用空格填充
  - lcase：把大写字符转换为小写字符
  - ucase：把小写字符转换为大写字符
  - swap：交换输入的每对字节
  - noerror：出错时不停止
  - notrunc：不截短输出文件
  - sync：将每个输入块填充到ibs个字节，不足部分用空（NUL）字符补齐。
- --help：显示帮助信息
- --version：显示版本信息

### 实例

在Linux 下制作启动盘，可使用如下命令：

```bash
$ dd if=boot.img of=/dev/fd0 bs=1440k 
```

将testfile文件中的所有英文字母转换为大写，然后转成为testfile_1文件，在命令提示符中使用如下命令：

```bash
$  dd if=testfile_2 of=testfile_1 conv=ucase 
```

其中testfile_2 的内容为：

```
$ cat testfile_2 #testfile_2的内容  
HELLO LINUX!  
Linux is a free unix-type opterating system.  
This is a linux testfile!  
Linux test 
```

转换完成后，testfile_1 的内容如下：

```
$ dd if=testfile_2 of=testfile_1 conv=ucase #使用dd 命令，大小写转换记录了0+1 的读入  
记录了0+1 的写出  
95字节（95 B）已复制，0.000131446 秒，723 KB/s  
cmd@hdd-desktop:~$ cat testfile_1 #查看转换后的testfile_1文件内容  
HELLO LINUX!  
LINUX IS A FREE UNIX-TYPE OPTERATING SYSTEM.  
THIS IS A LINUX TESTFILE!  
LINUX TEST #testfile_2中的所有字符都变成了大写字母 
```

由标准输入设备读入字符串，并将字符串转换成大写后，再输出到标准输出设备，使用的命令为：

```
dd conv=ucase 
```

输入以上命令后按回车键，输入字符串，再按回车键，按组合键Ctrl+D 退出，出现以下结果：

```
$ dd conv=ucase 
Hello Linux! #输入字符串后按回车键  
HELLO LINUX! #按组合键Ctrl+D退出，转换成大写结果  
记录了0+1 的读入  
记录了0+1 的写出  
13字节（13 B）已复制，12.1558 秒，0.0 KB/s 
```

当前目录下创建10个5G 的文件

```bash
$ for i in {0..9};do dd if=/dev/zero of=file${i} bs=1G count=5;done
```

dd 磁盘测试

1、 生成大文件用于读写

```bash
$ dd if=/dev/zero of=io.file bs=1024M count=50 
```

2、使用大文件来读写

```bash
$ dd if=io.file if=/dev/null bs=1024M
```



文档来自: [https://www.runoob.com/linux/linux-comm-dd.html](https://www.runoob.com/linux/linux-comm-dd.html)