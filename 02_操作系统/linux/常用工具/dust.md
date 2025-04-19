## dust 

Dust 下载安装, 可在github 的这个地址选择对应版本下载

[https://github.com/bootandy/dust/releases](https://github.com/bootandy/dust/releases)

### 参数

```bash
Usage: dust
Usage: dust <dir>
Usage: dust <dir>  <another_dir> <and_more>
Usage: dust -p (full-path - Show fullpath of the subdirectories)
Usage: dust -s (apparent-size - shows the length of the file as opposed to the amount of disk space it uses)
Usage: dust -n 30  (Shows 30 directories instead of the default [default is terminal height])
Usage: dust -d 3  (Shows 3 levels of subdirectories)
Usage: dust -D (Show only directories (eg dust -D))
Usage: dust -F (Show only files - finds your largest files)
Usage: dust -r (reverse order of output)
Usage: dust -H (si print sizes in powers of 1000 instead of 1024)
Usage: dust -X ignore  (ignore all files and directories with the name 'ignore')
Usage: dust -x (Only show directories on the same filesystem)
Usage: dust -b (Do not show percentages or draw ASCII bars)
Usage: dust -B (--bars-on-right - Percent bars moved to right side of screen])
Usage: dust -i (Do not show hidden files)
Usage: dust -c (No colors [monochrome])
Usage: dust -f (Count files instead of diskspace)
Usage: dust -t (Group by filetype)
Usage: dust -z 10M (min-size, Only include files larger than 10M)
Usage: dust -e regex (Only include files matching this regex (eg dust -e "\.png$" would match png files))
Usage: dust -v regex (Exclude files matching this regex (eg dust -v "\.png$" would ignore png files))
Usage: dust -L (dereference-links - Treat sym links as directories and go into them)
Usage: dust -P (Disable the progress indicator)
Usage: dust -R (For screen readers. Removes bars/symbols. Adds new column: depth level. (May want to use -p for full path too))
Usage: dust -S (Custom Stack size - Use if you see: 'fatal runtime error: stack overflow' (default allocation: low memory=1048576, high memory=1073741824)"),
Usage: dust --skip-total (No total row will be displayed)
Usage: dust -z 4000000 (Exclude output files/directories below size 4MB)
```



### 使用



1、 统计当前目录中的所有文件和所有文件夹的大小

```bash
dust -d 1
```



