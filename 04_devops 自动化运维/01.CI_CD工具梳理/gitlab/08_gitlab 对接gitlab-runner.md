## gitlab 对接gitlab-runner 

### 背景

- Jenkins 对于gitlab的流水线设计，各种使用还不是很完善，日志回传等还需要打开账户在jenkins 上去排查

### 方案

- 启动gitlab-runner 来进行流水线打包



### 配置步骤

1、 使用docker-compose 启动一个gitlab-runner 实例

```yaml
version: '3'
services:
  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    restart: unless-stopped
    depends_on:
      - gitlab
    privileged: true
    volumes:
      - ./config/gitlab-runner:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
      - /bin/docker:/bin/docker
```

配置完成之后， 需要对gitlab-runner 进行初始化, 对于注册时的token， 可以直接从gitlab上， 创建runner 时获得

```bash
# 进入gitlab-runner 的容器环境
$ docker exec -it gitlab-runner bash

# 对gitlab-runner 进行注册
$ gitlab-runner register --tls-ca-file /etc/gitlab-runner/test.crt --url https://test.cmzhu.cn --token glrt-U1UPjDsyg5h1wjGonym6
```



参考： 

gitlab-runner 的docker配置：[https://docs.gitlab.cn/runner/executors/docker.html](https://docs.gitlab.cn/runner/executors/docker.html)