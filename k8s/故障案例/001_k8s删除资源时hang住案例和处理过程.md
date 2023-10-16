## k8s 删除资源时hang 住

### 背景描述

- 删除pods、namespace等资源时，hang在删除命令界面无法自动退出

- [参考链接]: https://blog.csdn.net/weixin_40449300/article/details/108348420

  

### 解决过程

- 尝试用strace kubectl delete ...命令追踪命令执行过程中的hang死信息，无太多帮助，放弃此线路

- 尝试重启kube-dns，问题仍存在

- 查看docker日志，无太多发现

- 查看kubelet日志，有以下重要提示：unable to retrieve the complete list of server API

### 解决方案

#### k8s 

```bash
#1.查找到出问题的apiservice
$ kubectl get apiservice
#可以看到其中有出现false状态的apiservice,删除出问题的apiservice故障即可解决。
#2. 删除出问题的apiservice
$ kubectl delete apiservce <service-name>
```

#### OpenShift

```bash
# kubectl get ns 查看处于Terminating的ns
[root@VM_1_4_centos ~]# kubectl get ns | grep testns
testns Terminating 21d
# 将处于Terminating的ns的描述文件保存下来
[root@VM_1_4_centos ~]# kubectl get ns testns -o json > tmp.json
[root@VM_1_4_centos ~]# cat tmp.json 
{
 "apiVersion": "v1",
 "kind": "Namespace",
  "metadata": {
 "creationTimestamp": "2020-10-13T14:28:07Z",
 "name": "testns",
 "resourceVersion": "13782744400",
 "selfLink": "/api/v1/namespaces/testns",
 "uid": "9ff63d71-a4a1-43bc-89e3-78bf29788844"
 },
 "spec": {
 "finalizers": [
 "kubernetes"
 ]
 },
 "status": {
 "phase": "Terminating"
 }
}
# 删除上述json文件的spec和status部分
 "spec": {
 "finalizers": [
 "kubernetes"
 ]
 },
 "status": {
 "phase": "Terminating"
 } 
# 本地启动kube proxy
kubectl proxy --port=8081
# 新开窗口执行删除操作
curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://
127.0.0.1:8081/api/v1/namespaces/testns/finalize
```

