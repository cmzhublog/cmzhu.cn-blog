## elasticsearch 的简单使用

### curl 接口调用

1、查看集群状态

```bash
 $ curl 10.96.1.199:9200/_cat/health?v
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1711693741 06:29:01  elasticsearch yellow          1         1      1   1    0    0        1             0                  -                 50.0%
```

绿色表示一切正常, 黄色表示所有的数据可用但是部分副本还没有分配,红色表示部分数据因为某些原因不可用

2、获取集群节点列表

```bash
 $ curl 10.96.1.199:9200/_cat/nodes?v
ip            heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
100.64.231.69           30          97  13    1.81    2.19     2.02 dilmrt    *      elasticsearch-master-0
```

3、 查看所有的index

```bash
 $ curl 10.96.1.199:9200/_cat/indices?v
health status index                     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   kubernetes-log-2024.03.29 uPgHGPgeTOCjxjYGtqbTFA   1   1        217            0      2.8mb          2.8mb
```

4、查询所有的index下包含的所有type

```bash
$ curl 10.96.1.199:9200/_mapping?pretty=true
{
  "kubernetes-log-2024.03.29" : {
    "mappings" : {
      "properties" : {
        "_partial" : {
          "type" : "boolean"
        },
        "container" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "message" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "namespace" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "pod" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

5、查看某一个index 下的所有type(下面命令表示查看kubernetes-log-2024.03.29 的所有type)

```bash
 $ curl 10.96.1.199:9200/kubernetes-log-2024.03.29/_mapping?pretty=true
{
  "kubernetes-log-2024.03.29" : {
    "mappings" : {
      "properties" : {
        "_partial" : {
          "type" : "boolean"
        },
        "container" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "message" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "namespace" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "pod" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

6、查询某一个index 的所有数据

```bash
 $ curl  10.96.1.199:9200/kubernetes-log-2024.03.29/_search?pretty=true
```

7、和sql 一样查询数据

```bash
```





