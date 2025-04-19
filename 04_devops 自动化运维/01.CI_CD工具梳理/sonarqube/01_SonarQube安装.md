## SonarQube 安装

**使用docker 安装SonarQube**

1、 其他数据库驱动均已支持， 如果使用了Oracle 数据库， 需要手动安装jdbc 驱动

-  启动一个带有embedded  H2 数据库的 SonarQube 容器,用于提前创建SonarQube存储目录
  ```bash
  $ docker run --rm \
      -p 9000:9000 \
      -v ${PWD}/sonarqube_extensions:/opt/sonarqube/extensions \
      sonarqube:community
  ```

- SonarQuube 正确启动后退出

- 将 Oracle JDBC 驱动程序复制到 `sonarqube_extensions/jdbc-driver/oracle`

2、启动SonarQube 容器

```bash
$ docker run -d --name sonarqube \
    -p 9000:9000 \
    -e SONAR_JDBC_URL=... \
    -e SONAR_JDBC_USERNAME=... \
    -e SONAR_JDBC_PASSWORD=... \
    -v sonarqube_data:/opt/sonarqube/data \
    -v sonarqube_extensions:/opt/sonarqube/extensions \
    -v sonarqube_logs:/opt/sonarqube/logs \
    sonarqube:community
```

3、docker-compose 启动SonarQube; 以下是docker-compose示例文件

```yaml
version: "3"

services:
  sonarqube:
    image: sonarqube:community
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - ${PWD}/sonarqube_data:/opt/sonarqube/data
      - ${PWD}/sonarqube_extensions:/opt/sonarqube/extensions
      - ${PWD}/sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"
  db:
    image: postgres:12
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
    volumes:
      - ${PWD}/postgresql:/var/lib/postgresql
      - ${PWD}/postgresql_data:/var/lib/postgresql/data

```

