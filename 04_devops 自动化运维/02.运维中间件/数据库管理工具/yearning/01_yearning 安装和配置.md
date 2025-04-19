## yearning 的安装和配置



Docker-compose 安装,首次运行时，使用`yearning install 进行安装`

参考：[https://hub.docker.com/r/yeelabs/yearning](https://hub.docker.com/r/yeelabs/yearning)





```yaml
version: '3'

services:
    yearning:
        image: docker.io/yeelabs/yearning:v3.1.9
        environment:
           MYSQL_USER: yearning
           MYSQL_PASSWORD: dsfsfddecdcece
           MYSQL_ADDR: mysql
           MYSQL_DB: yearning
           Y_LANG: zh_CN   ## zh_CN / en_US
           SECRET_KEY: dbcjqheupqjsuwsm
           IS_DOCKER: is_docker
        ports:
           - 30080:8000
        # 首次使用请先初始化
        #command: /bin/bash -c "./Yearning install && ./Yearning run"
        depends_on:
           - mysql
        restart: always

    mysql:
        image: docker.io/library/mariadb:11.4.5
        environment:
           MARIADB_ROOT_PASSWORD: dsfsfddecdcece
           MARIADB_DATABASE: yearning
           MARIADB_USER: yearning
           MARIADB_PASSWORD: dsfsfddecdcece
        command:
           - --character-set-server=utf8mb4
           - --collation-server=utf8mb4_general_ci
        volumes:
           - ./data/mysql:/var/lib/mysql
```

修改为k8s 时为可以创建`yearning.yaml`

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yearning
  namespace: cmzhu
  labels:
    app: yearning
spec:
  replicas: 1
  selector:
    matchLabels:
      app: yearning
  template:
    metadata:
      labels:
        app: yearning
    spec:
      nodeSelector:
        yearning: "true"
      initContainers:
        - name: check-mariadb-running
          image: hub.test.com:8443/3rdimages/docker.io/library/mariadb:11.4.5
          command: 
            - /bin/bash
            - -c
          args:
            - | 
              while true; do
                mariadb -h mariadb.cmzhu -u root -p${MARIADB_ROOT_PASSWORD} -e "SELECT 1;" > /dev/null 2>&1
                if [ $? -eq 0 ]; then
                  echo "MySQL 服务正常运行！"
                  break
                else
                  echo "MySQL 服务未运行，正在尝试重新连接..."
                  sleep 5
                fi
              done
          env: 
            - name: MARIADB_ROOT_PASSWORD
              value: dsfsfddecdcece
            - name: MARIADB_DATABASE
              value: yearning
            - name: MARIADB_USER
              value: yearning
            - name: MARIADB_PASSWORD
              value: dsfsfddecdcece
          resources: #k8s资源限制
            requests:
              memory: "256Mi"
              cpu: "64m"
            limits:
              memory: "512Mi"
              cpu: "2000m"
        - name: init-yearning
          image: hub.test.com:8443/3rdimages/docker.io/yeelabs/yearning:v3.1.9
          command: 
            - /bin/bash
            - -c
          args:
            -  ./Yearning install
          env: 
            - name: MYSQL_USER
              value: yearning
            - name: MYSQL_PASSWORD
              value: dsfsfddecdcece
            - name: MYSQL_ADDR
              value: mariadb.cmzhu
            - name: MYSQL_DB
              value: yearning
            - name: Y_LANG
              value: zh_CN
            - name: SECRET_KEY
              value: dbcjqheupqjsuwsm
            - name: IS_DOCKER
              value: is_docker  
          resources: #k8s资源限制
            requests:
              memory: "256Mi"
              cpu: "64m"
            limits:
              memory: "512Mi"
              cpu: "2000m"
      containers:
        - name: yearning
          image: hub.test.com:8443/3rdimages/docker.io/yeelabs/yearning:v3.1.9
          imagePullPolicy: IfNotPresent # 拉取镜像策略
          command:
            - /bin/bash
            - -c 
          args: 
            - ./Yearning run
          env: 
            - name: MYSQL_USER
              value: yearning
            - name: MYSQL_PASSWORD
              value: dsfsfddecdcece
            - name: MYSQL_ADDR
              value: mariadb.cmzhu
            - name: MYSQL_DB
              value: yearning
            - name: Y_LANG
              value: zh_CN
            - name: SECRET_KEY
              value: dbcjqheupqjsuwsm
            - name: IS_DOCKER
              value: is_docker  
          ports:
            - containerPort: 8000
          resources: #k8s资源限制
            requests:
              memory: "256Mi"
              cpu: "64m"
            limits:
              memory: "512Mi"
              cpu: "2000m"
---
apiVersion: v1
kind: Service
metadata:
  name: yearning
  namespace: cmzhu
spec:
  selector:
    app: yearning
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000


---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb
  namespace: cmzhu
  labels:
    app: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      nodeSelector:
        yearning: "true"
      containers:
        - name: mariadb
          image: hub.test.com:8443/3rdimages/docker.io/library/mariadb:11.4.5
          args:
            - "--character-set-server=utf8mb4"
            - "--collation-server=utf8mb4_general_ci"
          imagePullPolicy: IfNotPresent # 拉取镜像策略
          env: 
            - name: MARIADB_ROOT_PASSWORD
              value: dsfsfddecdcece
            - name: MARIADB_DATABASE
              value: yearning
            - name: MARIADB_USER
              value: yearning
            - name: MARIADB_PASSWORD
              value: dsfsfddecdcece
          ports:
            - containerPort: 3306
          resources: #k8s资源限制
            requests:
              memory: "256Mi"
              cpu: "64m"
            limits:
              memory: "512Mi"
              cpu: "2000m"
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: mariadb-data
      volumes:
        - name: mariadb-data
          hostPath:
            path: /data/mariadb_data # directory location on host
            type: Directory # this field is optional
---
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  namespace: cmzhu
spec:
  selector:
    app: mariadb
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
  name: yearning-ingress
  namespace: cmzhu
spec:
  rules:
  - host: yearning.cmzhu.cn
    http:
      paths:
      - backend:
          service:
            name: yearning
            port:
              number: 8000
        path: /
        pathType: Prefix
```





