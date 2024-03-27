## elasticsearch 环境搭建

### 依赖

- version : 7.8.1
- kubernetes: v1.25.0



参考链接: [https://artifacthub.io/packages/helm/elastic/elasticsearch/7.8.1](https://artifacthub.io/packages/helm/elastic/elasticsearch/7.8.1)



### 安装步骤

安装过程使用helm 部署

1、拉取相应的`helm-charts`包

```bash
 $ helm repo add elastic https://helm.elastic.co
"elastic" has been added to your repositories
```

2、 查找对应的helm-charts包,并下载解压下来

```bash
$ helm pull elastic/elasticsearch --version 7.8.1 --untar
```

3、 将elasticsearch 的values 文件拷贝出来, 用于修改

```bash
 $ cp elasticsearch/values.yaml .
```

yaml 文件修改为: 

```yaml
replicas: 1
```

4、 使用命令将创建对应的helm , 说明对应的charts 和k8s 1.25.0 版本不兼容

```bash
$ helm \
upgrade --install \
elasticsearch \
-n elastic-system \
--create-namespace \
./elasticsearch -f ./values.yaml

Release "elasticsearch" does not exist. Installing it now.
Error: unable to build kubernetes objects from release manifest: unable to recognize "": no matches for kind "PodDisruptionBudget" in version "policy/v1beta1"
```

5、 将配置导出成模板

```bash
$ helm \
template \
elasticsearch \
-n elastic-system \
--create-namespace \
./elasticsearch -f ./values.yaml > elaticsearch.yaml

```

6、查看相关的配置elaticsearch.yaml

```yaml
 ---
# Source: elasticsearch/templates/poddisruptionbudget.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: "elasticsearch-master-pdb"
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: "elasticsearch-master"
---
# Source: elasticsearch/templates/service.yaml
kind: Service
apiVersion: v1
metadata:
  name: elasticsearch-master
  labels:
    heritage: "Helm"
    release: "elasticsearch"
    chart: "elasticsearch"
    app: "elasticsearch-master"
  annotations:
    {}
spec:
  type: ClusterIP
  selector:
    release: "elasticsearch"
    chart: "elasticsearch"
    app: "elasticsearch-master"
  ports:
  - name: http
    protocol: TCP
    port: 9200
  - name: transport
    protocol: TCP
    port: 9300
---
# Source: elasticsearch/templates/service.yaml
kind: Service
apiVersion: v1
metadata:
  name: elasticsearch-master-headless
  labels:
    heritage: "Helm"
    release: "elasticsearch"
    chart: "elasticsearch"
    app: "elasticsearch-master"
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
spec:
  clusterIP: None # This is needed for statefulset hostnames like elasticsearch-0 to resolve
  # Create endpoints also if the related pod isn't ready
  publishNotReadyAddresses: true
  selector:
    app: "elasticsearch-master"
  ports:
  - name: http
    port: 9200
  - name: transport
    port: 9300
---
# Source: elasticsearch/templates/statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch-master
  labels:
    heritage: "Helm"
    release: "elasticsearch"
    chart: "elasticsearch"
    app: "elasticsearch-master"
  annotations:
    esMajorVersion: "7"
spec:
  serviceName: elasticsearch-master-headless
  selector:
    matchLabels:
      app: "elasticsearch-master"
  replicas: 1
  podManagementPolicy: Parallel
  updateStrategy:
    type: RollingUpdate
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-master
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 30Gi
  template:
    metadata:
      name: "elasticsearch-master"
      labels:
        heritage: "Helm"
        release: "elasticsearch"
        chart: "elasticsearch"
        app: "elasticsearch-master"
      annotations:

    spec:
      securityContext:
        fsGroup: 1000
        runAsUser: 1000
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - "elasticsearch-master"
            topologyKey: kubernetes.io/hostname
      terminationGracePeriodSeconds: 120
      volumes:
      enableServiceLinks: true
      initContainers:
      - name: configure-sysctl
        securityContext:
          runAsUser: 0
          privileged: true
        image: "docker.elastic.co/elasticsearch/elasticsearch:7.8.1"
        imagePullPolicy: "IfNotPresent"
        command: ["sysctl", "-w", "vm.max_map_count=262144"]
        resources:
          {}

      containers:
      - name: "elasticsearch"
        securityContext:
          capabilities:
            drop:
            - ALL
          runAsNonRoot: true
          runAsUser: 1000
        image: "docker.elastic.co/elasticsearch/elasticsearch:7.8.1"
        imagePullPolicy: "IfNotPresent"
        readinessProbe:
          exec:
            command:
              - sh
              - -c
              - |
                #!/usr/bin/env bash -e
                # If the node is starting up wait for the cluster to be ready (request params: "wait_for_status=green&timeout=1s" )
                # Once it has started only check that the node itself is responding
                START_FILE=/tmp/.es_start_file

                http () {
                  local path="${1}"
                  local args="${2}"
                  set -- -XGET -s

                  if [ "$args" != "" ]; then
                    set -- "$@" $args
                  fi

                  if [ -n "${ELASTIC_USERNAME}" ] && [ -n "${ELASTIC_PASSWORD}" ]; then
                    set -- "$@" -u "${ELASTIC_USERNAME}:${ELASTIC_PASSWORD}"
                  fi

                  curl --output /dev/null -k "$@" "http://127.0.0.1:9200${path}"
                }

                if [ -f "${START_FILE}" ]; then
                  echo 'Elasticsearch is already running, lets check the node is healthy'
                  HTTP_CODE=$(http "/" "-w %{http_code}")
                  RC=$?
                  if [[ ${RC} -ne 0 ]]; then
                    echo "curl --output /dev/null -k -XGET -s -w '%{http_code}' \${BASIC_AUTH} http://127.0.0.1:9200/ failed with RC ${RC}"
                    exit ${RC}
                  fi
                  # ready if HTTP code 200, 503 is tolerable if ES version is 6.x
                  if [[ ${HTTP_CODE} == "200" ]]; then
                    exit 0
                  elif [[ ${HTTP_CODE} == "503" && "7" == "6" ]]; then
                    exit 0
                  else
                    echo "curl --output /dev/null -k -XGET -s -w '%{http_code}' \${BASIC_AUTH} http://127.0.0.1:9200/ failed with HTTP code ${HTTP_CODE}"
                    exit 1
                  fi

                else
                  echo 'Waiting for elasticsearch cluster to become ready (request params: "wait_for_status=green&timeout=1s" )'
                  if http "/_cluster/health?wait_for_status=green&timeout=1s" "--fail" ; then
                    touch ${START_FILE}
                    exit 0
                  else
                    echo 'Cluster is not yet ready (request params: "wait_for_status=green&timeout=1s" )'
                    exit 1
                  fi
                fi
          failureThreshold: 3
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 3
          timeoutSeconds: 5
        ports:
        - name: http
          containerPort: 9200
        - name: transport
          containerPort: 9300
        resources:
          limits:
            cpu: 1000m
            memory: 2Gi
          requests:
            cpu: 1000m
            memory: 2Gi
        env:
          - name: node.name
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: cluster.initial_master_nodes
            value: "elasticsearch-master-0,"
          - name: discovery.seed_hosts
            value: "elasticsearch-master-headless"
          - name: cluster.name
            value: "elasticsearch"
          - name: network.host
            value: "0.0.0.0"
          - name: ES_JAVA_OPTS
            value: "-Xmx1g -Xms1g"
          - name: node.data
            value: "true"
          - name: node.ingest
            value: "true"
          - name: node.master
            value: "true"
        volumeMounts:
          - name: "elasticsearch-master"
            mountPath: /usr/share/elasticsearch/data
---
# Source: elasticsearch/templates/test/test-elasticsearch-health.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "elasticsearch-bwqng-test"
  annotations:
    "helm.sh/hook": test-success
spec:
  securityContext:
    fsGroup: 1000
    runAsUser: 1000
  containers:
  - name: "elasticsearch-uesyh-test"
    image: "docker.elastic.co/elasticsearch/elasticsearch:7.8.1"
    imagePullPolicy: "IfNotPresent"
    command:
      - "sh"
      - "-c"
      - |
        #!/usr/bin/env bash -e
        curl -XGET --fail 'elasticsearch-master:9200/_cluster/health?wait_for_status=green&timeout=1s'
  restartPolicy: Never
```

7、测试是否搭建完成

```bash
 $ k get pod  -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP              NODE    NOMINATED NODE   READINESS GATES
elasticsearch-bwqng-test   0/1     Error     0          4m41s   100.64.231.94   cmzhu   <none>           <none>
elasticsearch-master-0     1/1     Running   0          4m41s   100.64.231.69   cmzhu   <none>           <none>
```

```bash
 $ kubectl get svc
NAME                            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)             AGE
elasticsearch-master            ClusterIP   10.96.1.199   <none>        9200/TCP,9300/TCP   6m18s
elasticsearch-master-headless   ClusterIP   None          <none>        9200/TCP,9300/TCP   6m18s
```

```bash
 $ telnet 10.96.1.199 9200
Trying 10.96.1.199...
Connected to 10.96.1.199.
Escape character is '^]'.
```

```bash
 $ curl 10.96.1.199:9200/_cat
=^.^=
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/thread_pool/{thread_pools}
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates
/_cat/ml/anomaly_detectors
/_cat/ml/anomaly_detectors/{job_id}
/_cat/ml/trained_models
/_cat/ml/trained_models/{model_id}
/_cat/ml/datafeeds
/_cat/ml/datafeeds/{datafeed_id}
/_cat/ml/data_frame/analytics
/_cat/ml/data_frame/analytics/{id}
/_cat/transforms
/_cat/transforms/{transform_id}
```

