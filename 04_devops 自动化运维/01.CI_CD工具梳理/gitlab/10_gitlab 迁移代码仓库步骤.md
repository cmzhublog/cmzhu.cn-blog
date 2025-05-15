## 代码仓迁移

#### 手动迁移

1、 克隆云效代码镜像，检出命令：

```bash
$ git clone --mirror
```

2、 进入本地仓库目录  

```bash
$ cd <本地仓库目录>
```

3、 绑定gitlab新仓库地址：

```bash
$ git remote add gitlab <gitlab_url>
```

4、 检查

```bash
$ git remote -v
```



5、正确无误的情况下推送代码至gitlab，如有其他异常需处理完毕后推送

推送命令： 

```bash
$ git push --all gitlab --force
```

