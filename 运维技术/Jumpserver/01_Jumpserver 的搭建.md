## Jumpserver 搭建步骤

参考项目： [https://github.com/jumpserver/Dockerfile/tree/master](https://github.com/jumpserver/Dockerfile/tree/master)

### 1.0 docker-compose 方案搭建 

参考上述文档， 可以知道docker-compose.yml 文件如下， 项目中属于将文件分开， 我则将所有的compose 配置文件全部合并在一起

```yaml
services:
  redis:
    image: redis:7.0
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_redis
    restart: always
    command: redis-server --requirepass $REDIS_PASSWORD --loglevel warning --maxmemory-policy allkeys-lru
    environment:
      TZ: ${TZ:-Asia/Shanghai}
      REDIS_PORT: $REDIS_PORT
      REDIS_PASSWORD: $REDIS_PASSWORD
    healthcheck:
      test: "redis-cli -h 127.0.0.1 -p $$REDIS_PORT -a $$REDIS_PASSWORD info Replication"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
      - ${VOLUME_DIR}/redis/data:/data
    networks:
      - net

  mysql:
    image: mariadb:10.6
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_mysql
    restart: always
    command: --character-set-server=utf8 --collation-server=utf8_general_ci
    environment:
      TZ: ${TZ:-Asia/Shanghai}
      DB_PORT: $DB_PORT
      MARIADB_ROOT_PASSWORD: $DB_PASSWORD
      MARIADB_DATABASE: $DB_NAME
    healthcheck:
      test: "mysql -h127.0.0.1 -P$$DB_PORT -uroot -p$$MARIADB_ROOT_PASSWORD -e 'SHOW DATABASES;'"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
    volumes:
      - ${VOLUME_DIR}/mariadb/data:/var/lib/mysql
    networks:
      - net

  core:
    image: jumpserver/jms_core:${VERSION}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_core
    command: start web
    deploy:
      mode: replicated
      replicas: 1
      # resources:
        # limits:
          # cpus: "1"
          # memory: 1536M
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - "node.role==worker"
    env_file: .env
    healthcheck:
      test: "check http://localhost:8080/api/health/"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 90s
    volumes:
      - ${VOLUME_DIR}/core/data:/opt/jumpserver/data
    networks:
      - net

  celery:
    image: jumpserver/jms_core:${VERSION}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_celery
    command: start task
    deploy:
      mode: replicated
      replicas: 1
      # resources:
        # limits:
          # cpus: "0.2"
          # memory: 1024M
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - "node.role==worker"
    env_file: .env
    healthcheck:
      test: "bash /opt/jumpserver/utils/check_celery.sh"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
    volumes:
      - ${VOLUME_DIR}/core/data:/opt/jumpserver/data
    networks:
      - net

  koko:
    image: jumpserver/jms_koko:${VERSION}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_koko
    privileged: true
    deploy:
      mode: replicated
      replicas: 1
      # resources:
        # limits:
          # cpus: "0.1"
          # memory: 128M
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - "node.role==worker"
    env_file: .env
    healthcheck:
      test: "check http://localhost:5000/koko/health/"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
      - ${VOLUME_DIR}/koko/data:/opt/koko/data
    ports:
      - ${SSH_PORT:-2222}:${SSH_PORT:-2222}
    networks:
      - net

  guacd:
    image: jumpserver/guacd:1.5.5-bookworm
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_guacd
    user: root
    deploy:
      mode: replicated
      replicas: 1
      # resources:
        # limits:
          # cpus: "0.1"
          # memory: 128M
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - "node.role==worker"
    env_file: .env
    volumes:
      - ${VOLUME_DIR}/lion/data:/opt/lion/data
    networks:
      - net

  lion:
    image: jumpserver/jms_lion:${VERSION}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_lion
    deploy:
      mode: replicated
      replicas: 1
      # resources:
        # limits:
          # cpus: "0.1"
          # memory: 128M
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - "node.role==worker"
    env_file: .env
    healthcheck:
      test: "check http://localhost:8081/lion/health/"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
      - ${VOLUME_DIR}/lion/data:/opt/lion/data
    networks:
      - net

  chen:
    image: jumpserver/jms_chen:${VERSION}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_chen
    deploy:
      mode: replicated
      replicas: 1
      # resources:
        # limits:
          # cpus: "0.1"
          # memory: 128M
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - "node.role==worker"
    env_file: .env
    volumes:
      - ${VOLUME_DIR}/chen/data:/opt/chen/data
    healthcheck:
      test: "check http://localhost:8082/chen"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 60s
    networks:
      - net

  web:
    image: jumpserver/jms_web:${VERSION}
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 1Gi
        reservations:
          cpus: '0.25'
          memory: 200M
    container_name: jms_web
    deploy:
      mode: replicated
      replicas: 1
      # resources:
        # limits:
          # cpus: "0.1"
          # memory: 128M
      restart_policy:
        condition: on-failure
      placement:
        constraints:
          - "node.role==worker"
    env_file: .env
    healthcheck:
      test: "check http://localhost/api/health/"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
      - ${VOLUME_DIR}/core/data:/opt/jumpserver/data
      - ${VOLUME_DIR}/web/data/logs:/var/log/nginx
      - ${VOLUME_DIR}/web/download:/opt/download
    ports:
      - ${HTTP_PORT:-80}:80
    networks:
      - net


networks:
  net:
    driver: ${NETWORK_DRIVER:-bridge}
    ipam:
      driver: default
      config:
        - subnet: $DOCKER_SUBNET

```

2、 合并完成之后， 发现上述还应用了一个环境变量文件 `.env`

```bash
# 版本号可以自己根据项目的版本修改
VERSION=v4.0.1

# 构建参数, 支持 amd64, arm64, ppc64le, s390x
TARGETARCH=amd64

# Compose, Swarm 模式下修改 NETWORK_DRIVER=overlay
COMPOSE_PROJECT_NAME=jms
# COMPOSE_HTTP_TIMEOUT=3600
# DOCKER_CLIENT_TIMEOUT=3600
DOCKER_SUBNET=192.168.250.0/24
NETWORK_DRIVER=bridge

# 持久化存储
VOLUME_DIR=/opt/jumpserver

# 时区
TZ=Asia/Shanghai
TIME_ZONE=Asia/Shanghai

# DB, 支持 MySQL, PostgreSQL
DB_ENGINE=mysql
DB_HOST=mysql
DB_PORT=3306
DB_USER=root
DB_PASSWORD=nu4x599Wq7u0Bn8EABh3J91G
DB_NAME=jumpserver

# Redis, 支持 Sentinel
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=8URXPL2x3HZMi7xoGTdk3Upj

# Core
SECRET_KEY=B3f2w8P2PfxIAS7s4URrD9YmSbtqX4vXdPUL217kL9XPUOWrmy
BOOTSTRAP_TOKEN=7Q11Vz6R2J6BLAdO
LOG_LEVEL=ERROR
DOMAINS=

# 组件通信
CORE_HOST=http://core:8080

# Lion
GUACD_LOG_LEVEL=error
GUA_HOST=guacd
GUA_PORT=4822

# Web
HTTP_PORT=20080
SSH_PORT=2222

##
# SECRET_KEY 保护签名数据的密匙, 首次安装请一定要修改并牢记, 后续升级和迁移不可更改, 否则将导致加密的数据不可解密。
# BOOTSTRAP_TOKEN 为组件认证使用的密钥, 仅组件注册时使用。组件指 koko, lion, chen ...

```

3、 都配置好之后， 使用docker-compose up -d 就能启动

```bash
$ docker-compose up -d 
```

4、发现所有服务全部运行之后，jumpserver 就搭建完成了

```bash
$ docker-compose ps 
```

5、搭建完成后， 使用默认账户进行登录

```bash
默认账户: admin
默认密码：ChangeMe
```

