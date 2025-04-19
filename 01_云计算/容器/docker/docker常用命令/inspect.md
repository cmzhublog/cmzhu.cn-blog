---
title: inspect
description: 
published: true
date: 2022-10-30T01:23:49.884Z
tags: docker
editor: markdown
dateCreated: 2022-10-30T01:23:49.884Z
---

# docker inspect 命令



**docker inspect** : 获取容器/镜像的元数据。

## 语法

```
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

OPTIONS说明：

- -f :指定返回值的模板文件。

- -s :显示总的文件大小。

- --type :为指定类型返回JSON。

## 实例
获取镜像mysql:5.6的元信息。

```
runoob@runoob:~$ docker inspect mysql:5.6
[
    {
        "Id": "sha256:2c0964ec182ae9a045f866bbc2553087f6e42bfc16074a74fb820af235f070ec",
        "RepoTags": [
            "mysql:5.6"
        ],
        "RepoDigests": [],
        "Parent": "",
        "Comment": "",
        "Created": "2016-05-24T04:01:41.168371815Z",
        "Container": "e0924bc460ff97787f34610115e9363e6363b30b8efa406e28eb495ab199ca54",
        "ContainerConfig": {
            "Hostname": "b0cf605c7757",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3306/tcp": {}
            },
...
```

获取正在运行的容器mymysql的 IP。

```
runoob@runoob:~$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mymysql
172.17.0.3
```