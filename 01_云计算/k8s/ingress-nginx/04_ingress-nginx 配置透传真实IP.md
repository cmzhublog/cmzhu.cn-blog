## ingress-nginx 配置X-Forwarded-for

1、 配置时可参考ingress-nginx 的configmap 配置，具体配置如下

```yaml
data:
  compute-full-forwarded-for: "true"
  forwarded-for-header: "X-Forwarded-For"
  use-forwarded-headers: "true"
```

2、详细的cm 配置如下

```yaml
apiVersion: v1
data:
  compute-full-forwarded-for: "true"
  forwarded-for-header: X-Forwarded-For
  use-forwarded-headers: "true"
kind: ConfigMap
metadata:
  labels:
    app: ingress-nginx
  name: nginx-configuration
  namespace: ingress-nginx
...
```

3、配置完成之后重启ingress-nginx-controller 容器

4、X-Forwarded-for 会带有中间所有层的IP，比如经过nginx -> ingress 所存储的IP 形式为

```bash
ClientIP(真实IP) -> Proxy01IP -> Proxy02IP ... 
likes: 222.209.85.222[,192.168.101.122]
```

