## shell 代码示例

1、 实现拷贝rsync 当前目录中所有不是csv后缀的文件

```bash
for filename in $(find /mnt/bfs/yunqi/mnt/nfsdata/userstudio/notebooks -type f  | grep -v csv );do \
    dirName=$(dirname $filename); \
    partialDir=$(echo "$dirName" | awk -F "/mnt/bfs/yunqi/mnt/nfsdata/userstudio/notebooks" '{print $2}'); \
    baseName=$(basename ${filename});    \
    mkdir -p /mnt/bfs/clientbak${partialDir}; \
    cp $filename /mnt/bfs/clientbak${partialDir}/${baseName} ; \
    done
```

