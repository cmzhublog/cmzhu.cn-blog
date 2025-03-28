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

4、最后替换`pyovpn-2.0-py3.10.egg`

```bash
mv pyovpn-2.0-py3.10.egg /usr/local/openvpn_as/lib/python/

systemctl restart openvpnas
```