## Ingress 迁移到Gateway

参考链接：[https://mp.weixin.qq.com/s/xdIcXJh5BduXxUMLyyioLQ](https://mp.weixin.qq.com/s/xdIcXJh5BduXxUMLyyioLQ) 感谢大佬分享

### 1. 背景

1. 2024 年K8S 社区提出明确信号，Ingress  API已进入维护模式，Gateway API才是未来

2. Ingress2Gateway 1.0的正式发布，标志着迁移工具链已经成熟。但"工具成熟"不等于"迁移无痛"——本文结合社区真实案例，整理出5个高频踩坑点，并给出完整的迁移实战方案。

### 2. 为什么要从Ingress迁移到Gateway API

#### 2.1 Ingress的历史局限

Ingress 仅仅支持HTTP和HTTPS 的简单路由场景

```yaml
# 传统Ingress：功能有限，扩展靠注解
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /      # 靠注解扩展
    nginx.ingress.kubernetes.io/canary: "true"         # 灰度发布也靠注解
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: api-v1
            port:
              number: 80
```

痛点：

- 注解是字符串，无类型检查，配置错误只在运行时暴露
- 不同Ingress Controller的注解互不兼容（nginx vs traefik vs istio）
- 不支持TCP/UDP路由（只能用workaround）
- 多租户场景下权限边界模糊

#### 2.2 Gateway API的设计哲学

Gateway API通过角色分离解决了这些问题：

```tex
基础设施管理员 → GatewayClass（定义网关实现）
集群管理员     → Gateway（定义监听端口和协议）
应用开发者     → HTTPRoute/TCPRoute（定义路由规则）
```

```yaml 
# Gateway API：结构清晰，类型安全
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-route
  namespace: production
spec:
  parentRefs:
  - name: prod-gateway
  hostnames:
  - "api.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /v1
    filters:
    - type: RequestHeaderModifier
      requestHeaderModifier:
        add:
        - name: X-Version
          value: "v1"
    backendRefs:
    - name: api-v1
      port: 80
      weight: 80      # 流量权重直接在spec中定义，无需注解
    - name: api-v2
      port: 80
      weight: 20
```

### 3.使用ingress2gateway一键迁移

#### 3.1 安装

```bash
# 方法1：直接下载二进制
$ curl -Lo ingress2gateway https://github.com/kubernetes-sigs/ingress2gateway/releases/latest/download/ingress2gateway-linux-amd64
$ chmod +x ingress2gateway && mv ingress2gateway /usr/local/bin/

# 方法2：通过krew安装
$ kubectl krew install ingress2gateway
```

 #### 3.2 迁移步骤

```bash
# Step 1：预览转换结果（不修改集群）
$ ingress2gateway print \
  --namespace production \
  --providers ingress-nginx

# Step 2：转换并输出到文件
$ ingress2gateway print \
  --namespace production \
  --providers ingress-nginx \
  > gateway-resources.yaml

# Step 3：检查生成的资源
$ cat gateway-resources.yaml

# Step 4：应用到集群（灰度阶段先不删除Ingress）
$ kubectl apply -f gateway-resources.yaml
```

#### 3.3 转换例子

转换前的ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /api(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

转换后的gateway 

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-ingress
spec:
  parentRefs:
  - name: nginx
    namespace: nginx-gateway
  hostnames:
  - api.example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    filters:
    - type: URLRewrite
      urlRewrite:
        path:
          type: ReplacePrefixMatch
          replacePrefixMatch: /
    backendRefs:
    - name: api-service
      port: 8080
```

 ### 4. 踩坑点

#### 4.1 正则表达式路径不被支持

现象： 使用了正则路径的Ingress转换后路由匹配失效。 

原因： Gateway API HTTPRoute不支持正则路径匹配（只支持Exact、PathPrefix、RegularExpression，但最后一种实现依赖具体Gateway）。

解决方案：

```yaml
# 将正则路由拆分为多个精确路由
rules:
- matches:
  - path:
      type: PathPrefix
      value: /api/v1
  backendRefs:
  - name: api-v1-service
    port: 8080
- matches:
  - path:
      type: PathPrefix
      value: /api/v2
  backendRefs:
  - name: api-v2-service
    port: 8080
```

#### 4.2 跨命名空间引用需要显式授权

现象： HTTPRoute引用其他namespace的Service，访问报403。

原因： Gateway API引入了ReferenceGrant资源控制跨namespace访问。

解决方案：

```yaml
# 在目标namespace创建ReferenceGrant
apiVersion: gateway.networking.k8s.io/v1beta1
kind: ReferenceGrant
metadata:
  name: allow-production-routes
  namespace: backend-services    # 被引用的namespace
spec:
  from:
  - group: gateway.networking.k8s.io
    kind: HTTPRoute
    namespace: frontend           # 允许来自frontend namespace的引用
  to:
  - group: ""
    kind: Service
```

#### 4.3 TLS证书配置方式变化

现象： 迁移后HTTPS访问返回证书错误。

原因： Ingress的TLS证书在Ingress资源中定义；Gateway API的TLS证书在Gateway资源中定义，HTTPRoute不感知TLS。

解决方案：

```yaml
# Gateway API TLS配置（在Gateway中）
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: prod-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    port: 443
    protocol: HTTPS
    tls:
      mode: Terminate
      certificateRefs:
      - name: api-tls-cert    # 证书在Gateway层配置
        namespace: cert-manager
```



#### 4.4 自定义Header注解迁移不完整

ingress2gateway工具对复杂注解的转换支持有限，需要手动补充：

```bash
# 检查哪些注解未被转换
$ ingress2gateway print --namespace production 2>&1 | grep -i "warning\|unsupported\|skip"
```

#### 4.5 超时与重试配置缺失

原Ingress的超时注解需要在Gateway API中用BackendLBPolicy或HTTPRoute的timeout字段重新配置：

```yaml
rules:
- matches:
  - path:
      type: PathPrefix
      value: /api
  timeouts:
    request: 30s        # 请求超时
    backendRequest: 10s # 后端响应超时
  backendRefs:
  - name: api-service
    port: 8080
```

### 5. 灰度迁移与回滚方案

```bash
# 灰度方案：Ingress和Gateway同时运行，通过DNS权重切流
# 1. 部署Gateway API资源（新入口）
kubectl apply -f gateway-resources.yaml

# 2. 验证Gateway API路由正常
curl -H "Host: api.example.com" http://<gateway-ip>/api/health

# 3. 逐步将DNS流量切到Gateway（先5%，再50%，再100%）
# 使用云厂商DNS权重路由功能

# 4. 确认无误后删除Ingress
kubectl delete ingress api-ingress -n production

# 回滚：如果Gateway API出现问题，切回DNS指向原Ingress IP
```

### 6. 总结

Gateway API不是Ingress的"升级版"，而是一次架构重构。迁移的核心要点：

1. 使用ingress2gateway工具完成80%的机械转换
2. 重点检查：正则路径、跨namespace引用、TLS配置、超时重试
3. 采用灰度+DNS权重的方式实现零停机迁移
4. 迁移完成后充分利用Gateway API的流量分割、多协议支持等新特性

迁移周期建议预留2-4周，充分测试后再全量切换。
