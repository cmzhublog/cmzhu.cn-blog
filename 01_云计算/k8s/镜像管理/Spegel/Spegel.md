## Spegel

[https://github.com/XenitAB/spegel](https://github.com/XenitAB/spegel)

### 安装部署

1、 首先将仓库从github 拉取下来, 并且安装spegel 

```bash
helm upgrade --install \
   --create-namespace --namespace spegel \
   --version v0.0.19 \
   spegel oci://ghcr.io/spegel-org/helm-charts/spegel

```
