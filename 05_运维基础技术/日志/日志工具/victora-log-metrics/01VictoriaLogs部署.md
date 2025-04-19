## VictoriaLogs 部署

### 背景

- 应用此方案替换 ssdb 存储日志



### 部署步骤

**Helm 部署**

1、 添加对应的Chart仓库

```bash
$ helm repo add vm https://victoriametrics.github.io/helm-charts/
"vm" has been added to your repositories
$ helm repo update
```

2、 查看可使用的版本charts

```bash
$ helm search repo vm/victoria-logs-single -l
NAME                    CHART VERSION   APP VERSION             DESCRIPTION                                       
vm/victoria-logs-single 0.3.4           v0.4.2-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.3.3           v0.4.1-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.3.2           v0.4.1-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.3.1           v0.3.0-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.3.0           v0.3.0-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.1.3           v0.3.0-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.1.2           v0.1.0-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.1.1           v0.1.0-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.1.0           v0.1.0-victorialogs     Victoria Logs Single version - high-performance...
vm/victoria-logs-single 0.0.1           v0.1.0-victorialogs     Victoria Logs Single version - high-performance...
```

3、将版本的values 导出成values 文件

```bash
$ helm show values vm/victoria-logs-single > values.yaml
```

**开始安装**

4、 测试命令是否可用

```bash
$ helm install vlsingle vm/victoria-logs-single -f values.yaml -n victoria-logs --debug --dry-run
```





### 参考地址

- https://docs.victoriametrics.com/VictoriaLogs/QuickStart.html 