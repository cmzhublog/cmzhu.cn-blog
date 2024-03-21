# 1_背景说明

现在业务系统接入Prometheus的exporter较多，存在大量不需要的指标，这些指标会大量占用存储空间和cpu、内存资源，同时导致Prometheus能够保存的数据较少。

为了解决该问题，可以对现在的监控指标进行精简和过滤。

# 2_配置方式

先登录Prometheus容器或者Prometheus服务页面，查看scrape_configs中对应服务的job_name对应内容，确定现有配置情况：

![image-20240321101031641](./01_prometheus%E8%BF%87%E6%BB%A4%E4%B8%8D%E9%9C%80%E8%A6%81%E7%9A%84%E6%8C%87%E6%A0%87.assets/image-20240321101031641.png)

编辑job_name对应的servicemonitor，例如编辑biggateway-exportor：

```bash
$ kubectl -n monitoring edit servicemonitor biggateway-exportor
```

![image-20240321101118142](./01_prometheus%E8%BF%87%E6%BB%A4%E4%B8%8D%E9%9C%80%E8%A6%81%E7%9A%84%E6%8C%87%E6%A0%87.assets/image-20240321101118142.png)

添加如下配置，请注意yaml缩进：

```yaml
    metricRelabelings:
    - action: drop
      regex: nginx_connections_accepted2222
      sourceLabels:
      - __name__
```

配置中：action指定对相关指标进行丢弃；regex为sourceLabels的正则表达式匹配；sourceLabels为需要丢弃的指标的label，这里使用__name__，即指标名称进行pip。

修改完servicemonitor后，直接保存相关配置，等待Prometheus Operator重新加载配置即可。

配置后，可以看到配置生效后对相应指标进行了drop：

![image-20240321101234018](./01_prometheus%E8%BF%87%E6%BB%A4%E4%B8%8D%E9%9C%80%E8%A6%81%E7%9A%84%E6%8C%87%E6%A0%87.assets/image-20240321101234018.png)



# 3_注意事项

- 修改servicemonitor后，Prometheus重新加载配置存在延迟，许耐心等待
- 如果修改servicemonitor后配置无法生效，请检查Prometheus Operator是否正常运行
- 如果需要drop的指标较多，可以将action设置为keep进行反向选择，减少配置量



# 4_参考链接

https://www.tencentcloud.com/zh/document/product/457/47004

https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/user-guides/running-exporters.md