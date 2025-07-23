## openvpn 解除用户人数限制

1、拷贝对应的包出来

```bash
$ cp /usr/local/openvpn_as/lib/python/pyovpn-2.0-py3.12.egg /data

$ unzip -q pyovpn-2.0-py3.12.egg

$ cd ./pyovpn/lic/

$ mv uprop.pyc uprop2.pyc
```

2、编辑一个新的文件 `vim uprop.py` 

```python
from pyovpn.lic import uprop2
old_figure = None
def new_figure(self, licdict):
  ret = old_figure(self, licdict)
  ret['concurrent_connections'] = 2048
  return ret

for x in dir(uprop2):
  if x[:2] == '__':
  	continue
  if x == 'UsageProperties':
    exec('old_figure = uprop2.UsageProperties.figure')
    exec('uprop2.UsageProperties.figure = new_figure')
    exec('%s = uprop2.%s' % (x, x))
```

3、编译上述的文件

```bash
$ python3 -O -m compileall /data/pyovpn/lic/uprop.py

$ zip -rq pyovpn-2.0-py3.12.egg ./pyovpn ./EGG-INFO ./common
```

4、最后替换`pyovpn-2.0-py3.12.egg`

```bash
mv pyovpn-2.0-py3.12.egg /usr/local/openvpn_as/lib/python/

systemctl restart openvpnas
```



#### 容器化破解

容器化破解Dockerfile 如下,必须使用buildkit 进行编译，步骤可参考： [buildkit编译镜像](https://wiki.cmzhu.cn/cmzhu.cn-blog/04_devops%20%E8%87%AA%E5%8A%A8%E5%8C%96%E8%BF%90%E7%BB%B4/01.CI_CD%E5%B7%A5%E5%85%B7%E6%A2%B3%E7%90%86/%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91/buildkit/)

```dockerfile
FROM openvpn/openvpn-as:2.14.3-5936bcd7-Ubuntu24

RUN apt-get update \
    && apt-get install unzip zip

RUN cd /usr/local/openvpn_as/lib/python/ \
    && echo $(ls pyovpn-*) > /tmp/var_cache \
    && unzip $(cat /tmp/var_cache) \
    && mv ./pyovpn/lic/uprop.pyc ./pyovpn/lic/uprop2.pyc

COPY <<EOF /usr/local/openvpn_as/lib/python/pyovpn/lic/uprop.py
from pyovpn.lic import uprop2
old_figure = None
def new_figure(self, licdict):
  ret = old_figure(self, licdict)
  ret['concurrent_connections'] = 2048
  return ret

for x in dir(uprop2):
  if x[:2] == '__':
    continue
  if x == 'UsageProperties':
    exec('old_figure = uprop2.UsageProperties.figure')
    exec('uprop2.UsageProperties.figure = new_figure')
    exec('%s = uprop2.%s' % (x, x))
EOF


RUN cd /usr/local/openvpn_as/lib/python/ \
  && zip -rq $(cat /tmp/var_cache) ./pyovpn ./EGG-INFO ./common
```

编译命令为：

```bash
$ DOCKER_BUILDKIT=1 docker buildx build  -t  dockerhub.cmzhu.cn:5000/3rdimages/docker.io/openvpn/openvpn-as:2 --platform linux/amd64 . --push
```

参考[dockerhub](https://hub.docker.com/r/openvpn/openvpn-as) 仓库进行手动部署

```yaml
---
version: "2.1"
services:
  openvpn-as:
    image: dockerhub.cmzhu.cn:5000/3rdimages/docker.io/openvpn/openvpn-as:2
    container_name: openvpn-as
    devices:
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - NET_ADMIN
      - MKNOD
    ports:
      - 943:943
      - 443:443
      - 1194:1194/udp
    volumes:
      - <path to data>:/openvpn
    restart: unless-stopped
```



