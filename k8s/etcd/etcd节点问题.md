---
title: etcd数据异常
description: 
published: true
date: 2023-02-13T02:43:06.057Z
tags: k8s
editor: markdown
dateCreated: 2023-02-13T02:35:38.609Z
---

# etcd 生产问题记录

## 背景

1、 集群在创建aiflow 的任务pod 的时候,发现创建失败.
2、 检查etcd日志发现集群etcd db_size 超过限制;
3、 集群状态出现异常

## 产生原因

1、 集群db_size 值超过额定的限制; (DB SIZE 默认最大为2GB)

![1.png](/image/etcd/1.png)

2、 在etcd 操作扩容DB SIZE 之后, 出现etcd 集群数据不同步问题, slave 节点的DB SIZE 比master 的DB SIZE 大;
![2.png](/image/etcd/2.png)



## 处理步骤

### 处理DB SIZE 超出默认值问题

1、先调整配额限制; 修改每一个etcd 节点的启动配置(/etc/kubernetes/manifests/etcd.yaml)

```bash
- command:
    - etcd
    - --advertise-client-urls=https://192.168.148.116:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.148.116:2380
    - --initial-cluster=control-master004=https://192.168.148.116:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.148.116:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.148.116:2380
    - --name=control-master004
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --quota-backend-bytes=3221225472   # 修改值变成3GB
    - --auto-compaction-mode=revision
    - --auto-compaction-retention=1000000
```



2、 重启之后会陆续运行成功;



### 处理etcd 数据同步异常问题

>  **处理思路**
>
> 将同步异常的节点踢出etcd 集群,数据备份之后再重新加入集群;

1 、 停止异常节点的kubelet

```bash
systemctl stop kubelet
```

2、 etcd 执行前缀

```bash
export ETCDCTL_API=3
export HOST_1=https://192.168.148.116   # etcd 节点
export HOST_2=https://192.168.148.117
export HOST_3=https://192.168.148.115
export ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379

# etcd 执行前缀
etcdctl --endpoints=$ENDPOINTS  \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key
   
```

3、 删除etcd list

```bash
# 查看所有节点
etcdctl --endpoints=$ENDPOINTS  \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key \
   member list 

# 清除节点
etcdctl --endpoints=$ENDPOINTS  \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key \
   member remove <MEMBER ID>
   

```

![3.png](/image/etcd/3.png)



4、去到数据异常的节点



5、将异常节点重新加入集群(目前是异常的,加入集群效果是报错)

```bash
export NAME=control-master006

etcdctl --endpoints=${HOST_1}:2379,${HOST_2}:2379 \
	member add ${NAME} \
	--peer-urls=http://${HOST_3}:2380
```



6、修改启动参数

```bash
# 主要修改两个
--initial-cluster-state existing # 标志着是新成员

# 集群状态参数也需要修改
NAME_1=control-master004
HOST_1=192.168.148.116
...

--initial-cluster=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_4}=http://${HOST_4}:2380
```



7 、启动完成重新同步etcd 数据,使集群正常(自动完成)

8、 恢复服务