## 卸载Docker

### 背景

在安装k8s 时， 有时会要求卸载docker 等其他的runc， 这时 需要将原来安装的runc 卸载， 这样才能更好的安装k8s

### 卸载步骤

1、 首先删除掉所有的容器

```bash
docker ps -qa |xargs -i docker rm -f {}
```

2、卸载掉docker-ce docker-ee, 并且删除掉docker 数据目录

```bash
# 查看docker存储目录
root@cmzhu:/data/clash# docker info | grep -i root
 Docker Root Dir: /var/lib/docker
 
# 卸载Docker CE
sudo apt-get purge docker-ce
# 卸载Docker EE
sudo apt-get purge docker-ee
# 删除Docker镜像、容器、数据卷等文件
sudo rm -rf /var/lib/docker
```

3、 可以使用find 命令查询， 还有多少文件没有删除， 会发现有很多

```bash
sudo find / -name '*docker*'
```

4、删除安装时自动安装的所有包

```bash
sudo apt-get autoremove docker docker-ce docker-engine docker.io containerd runc
```

5、查看是否还有没有卸载干净的包

```bash
dpkg -l | grep docker
```

6、卸载相应查询到的包

```bash
sudo apt-get autoremove docker-ce-*
```

7、 删除掉docker 相关的目录

```bash
sudo rm -rf /etc/systemd/system/docker.service.d
sudo rm -rf /var/lib/docker
sudo rm /etc/apparmor.d/docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

8、确定docker 卸载完成

```bash
docker --version
```

9、 最后执行一遍，并删除所有相关的文件（**尽量不要执行**）

```bash
sudo find / -name "*docker*" -exec `rm -rf` {} +
```



