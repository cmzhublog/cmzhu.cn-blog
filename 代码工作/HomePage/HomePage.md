## HomePage 的搭建

### 背景

- 当前博客网站没有一个合适的主页， 通过此方法为博客网站添加一个主页
- 主页来自： [https://github.com/Tomotoes/HomePage](https://github.com/Tomotoes/HomePage)
- 通过开源项目将项目打包到镜像中

### 部署方案

1、 docker 构建时需要的Dockerfile

```Dockerfile
FROM docker.io/dockette/nodejs:v12 as base

## install the packages
RUN apk add vim git python3

## clone the registry and install it 
RUN git clone https://github.com/Tomotoes/HomePage.git
COPY config.json /HomePage/config.json
RUN  cd HomePage \
    && npm install \
    && npm run build


## 部署阶段
FROM nginx:alpine3.18-perl as nginx
COPY --from=base /HomePage/dist  /usr/share/nginx/html
```

2、 注入特殊配置config.json

```json
{
        "head": {
                "title": "Congming Zhu's Home Page",
                "description": "Author:Congming Zhu,Category:Personal Home Page",
                "favicon": "favicon.ico"
        },
        "intro": {
                "title": "Congming Zhu's Home Page",
                "subtitle": "Front back left right end engineer",
                "enter": "回车",
                "supportAuthor": true,
                "background": true
        },
        "main": {
                "name": "Congming Zhu",
                "signature": "Code & Blog & Resume",
                "avatar": {
                        "link": "assets/avatar.jpg",
                        "height": "100",
                        "width": "100"
                },
                "ul": {
                        "first": {
                                "href": "https://wiki.cmzhu.cn",
                                "icon": "bokeyuan",
                                "text": "Blog"
                        },
                        "second": {
                                "href": "https://resume.cmzhu.cn",
                                "icon": "xiaolian",
                                "text": "About"
                        },
                        "third": {
                                "href": "zhucongming20@gmail.com",
                                "icon": "email",
                                "text": "Email"
                        },
                        "fourth": {
                                "href": "https://github.com/cmzhublog",
                                "icon": "github",
                                "text": "Github"
                        }
                }
        }
}

```

3、 将项目打包成镜像

```bash
$ docker build -t dockerhub.cmzhu.cn:5000/cmzhu/homepage:v1 .
```

4、在k8s 中创建deployment和service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: homepage
 namespace: cmzhu
spec:
  replicas: 2
  selector:
    matchLabels:
      app: homepage
  template:
    metadata:
      labels:
        app: homepage
    spec:
      containers:
      - name: homepage
        image: dockerhub.cmzhu.cn:5000/cmzhu/homepage:v1
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: homepage
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  selector:
    app: homepage
```

5、创建ingress 将服务暴露出来

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mkdocs
  namespace: cmzhu
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  rules:
  - host: cmzhu.cn
    http:
      paths:
      - backend:
          service:
            name: homepage
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - cmzhu.cn
    secretName: homepage
```

6、到此， 外部就可以通过[https://cmzhu.cn](https://cmzhu.cn) 访问主站了

![image-20240309202525260](Untitled.assets/image-20240309202525260.png)