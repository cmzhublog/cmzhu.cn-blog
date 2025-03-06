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

