## 修改默认的StorageClass

修改默认的storageClass 这为没有特殊需求的pvc 配置volumes

### 环境要求

- 至少有k8s 集群
- 有一个可用的StorageClass

### 改变默认的StorageClass

1、获取集群中的StorageClass

```bash
$ k get sc
NAME                         PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device               openebs.io/local   Delete          WaitForFirstConsumer   false                  7m48s
openebs-hostpath             openebs.io/local   Delete          WaitForFirstConsumer   false                  7m48s
```



 2、 判断是否是默认的storageclass 是通过修改注解变量storageclass.kubernetes.io/is-default-class来实现的; 当这个变量的值为false 时, 就不是默认sc

```bash
$ kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```



3、 如果需要将其设置为默认

```bash
$ kubectl patch storageclass <your-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

> 请注意，最多只能有一个 StorageClass 能够被标记为默认。 如果它们中有两个或多个被标记为默认，Kubernetes 将忽略这个注解， 也就是它将表现为没有默认 StorageClass。

4、 验证是否已设置成功

```bash
$ kubectl get storageclass
NAME                         PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device               openebs.io/local   Delete          WaitForFirstConsumer   false                  11m
openebs-hostpath (default)   openebs.io/local   Delete          WaitForFirstConsumer   false                  11m
```

