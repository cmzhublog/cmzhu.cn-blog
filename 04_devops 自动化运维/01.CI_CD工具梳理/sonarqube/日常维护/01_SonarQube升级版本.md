## SonarQube 升级版本

### 升级方案

1、备份pg 数据库

```bash
$ pg_dumpall > sonar_qube.sql
```

2、开始升级SonarQube时，先将原来的sonarQube 暂停,

```bash
$ docker-compose stop 
```



> 注意： 下面时原来的SonarQube 的docker-compose 安装配置
>
> ```yaml
> version: "3"
> 
> services:
>   sonarqube:
>     image: sonarqube:community
>     depends_on:
>       - db
>     environment:
>       SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
>       SONAR_JDBC_USERNAME: sonar
>       SONAR_JDBC_PASSWORD: sonar
>     volumes:
>       - ${PWD}/sonarqube_data:/opt/sonarqube/data
>       - ${PWD}/sonarqube_extensions:/opt/sonarqube/extensions
>       - ${PWD}/sonarqube_logs:/opt/sonarqube/logs
>     ports:
>       - "9000:9000"
>   db:
>     image: postgres:12
>     environment:
>       POSTGRES_USER: sonar
>       POSTGRES_PASSWORD: sonar
>     volumes:
>       - ${PWD}/postgresql:/var/lib/postgresql
>       - ${PWD}/postgresql_data:/var/lib/postgresql/data
> 
> ```

3、修改镜像为最新的镜像

```yaml
version: "3"

services:
  sonarqube:
    image: sonarqube:9.9.6-communit
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

4、镜像修改完成之后， 启动容器

```bash
$ docker-compose up -d 
```

会发现访问主页变成如下图所示， 这时不要担心，只需要访问`IP:PORT/setup`

![img](./01_SonarQube%E5%8D%87%E7%BA%A7%E7%89%88%E6%9C%AC.assets/6c0e97f6aea78607cd01a8f197f9c3d8.png)

5、然后会发现界面维护中， 浏览器访问`IP:PORT/setup`, 并点击升级即可,等待几分钟，升级完成之后， 就可以正常登录

![img](./01_SonarQube%E5%8D%87%E7%BA%A7%E7%89%88%E6%9C%AC.assets/23738ea67891f2c6305cf65ecbc0ba42.png)

6、如果在升级过程中， 出现任何异常， 可重新`psql`导入数据库进行恢复

