## openshift 不能exec 进入后台, 和不能logs 查看日志



### 问题描述

openshift 不能进入后台, 也不能查看日志, 具体报错有如下几类

1、 get 日志时有报错websocket 错误

```bash
WebSocket connection to 'wss://console-openshift-console.apps.example.com/api/kubernetes/api/v1/namespaces/openshift-console/pods/console-79b6c7bb87-gt2ck/log?container=console&follow=true&tailLines=1000&x-csrf-token=ESx4l2bhkAyUQ8nx9f0%2FmA3qThlJEI6IOptYX2N%2FSPBDwcQuQ1K91DDjT0I3J99QYF4rogNwgleVtq6FV%2BkL7Q%3D%3D' failed: Error during WebSocket handshake: Unexpected response code: 500
```

2、 在命令行中, exec, logs 和rsh 工具都在报错

```bash
$ oc logs console-79b6c7bb87-gt2ck

Error from server: Get https://master0.example.com:10250/containerLogs/openshift-console/console-79b6c7bb87-gt2ck/console: remote error: tls: internal error
```

3、 在集群安装好之后有大量的csr在pending

```bash
$ oc exec marketplace-operator-768b99959-9pftm -n openshift-marketplace -- echo foo
Error from server: error dialing backend: remote error: tls: internal error

$ oc logs marketplace-operator-768b99959-9pftm -n openshift-marketplace
Error from server: Get https://master:10250/containerLogs/openshift-marketplace/marketplace-operator-768b99959-9pftm/marketplace-operator: remote error: tls: internal error
```

4、kube-apiserver 也会有一些日志报错信息

```bash
$ sudo crictl ps | grep kube-api
239ec13eeaf4e       beaf65fce4dc16947c5bd5d1ca7e16313234c393e8ca1c4251ac9b85094972bb   About an hour ago   Running             kube-apiserver-operator                   3                   bd197ceb6f882
6f2bdcab072ca       beaf65fce4dc16947c5bd5d1ca7e16313234c393e8ca1c4251ac9b85094972bb   About an hour ago   Running             kube-apiserver-cert-syncer-8              1                   6938a6ebc2c3d
e6b9db2994d07       0d8dcfc307048a0f0400e644fcd1c9929018103b15d0f9b23b4841f1e71937bc   About an hour ago   Running             kube-apiserver-8                          1                   6938a6ebc2c3d

$ sudo crictl logs e6b9db2994d07
...
E0725 17:38:54.707552       1 status.go:64] apiserver received an error that is not an metav1.Status: &url.Error{Op:"Get", URL:"https://master:10250/containerLogs/openshift-kube-apiserver/kube-apiserver-master/kube-apiserver-8", Err:(*net.OpError)(0xc01ec89270)}
...
```



### 解决思路

1、 批准所有pending的csr

```bash
$ oc get csr -o name | xargs oc adm certificate approve
```



### 根因

1、This has been noted in related Bugzilla: [BZ-1733331](https://bugzilla.redhat.com/show_bug.cgi?id=1733331)

### 诊断步骤

检查所有的pending 的csr 和没有批准的csr ;

```bash
$ oc get csr
NAME        AGE     REQUESTOR                          CONDITION
csr-22bmr   4h45m   system:node:node0.example.com      Pending
csr-22dln   21h     system:node:master1.example.com    Pending
```

