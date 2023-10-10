---
title: openshift 证书升级
description: 
published: true
date: 2022-12-20T07:29:53.048Z
tags: k8s
editor: markdown
dateCreated: 2022-12-20T07:29:53.048Z
---

# openshift 证书过期问题解决

## 背景

1、 openshift 证书默认为1年1签,在每年都会遇到一次证书过期的问题, 当遇到这类问题时的处理方案如下

## 处理步骤

1、 首先确定是否是证书过期

```bash
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -v -i <inventory_file> \
    playbooks/openshift-checks/certificate_expiry/easy-mode.yaml
```

该指令最终会生成一套合适的报告,大概会指出xxx 证书已过期

2、 处理过期

该playbook 是openshift 官方给出的证书升级方案,直接通过下面方案执行即可,在执行过程中会遇到一些报错,在找到报错的task ,直接将其注释掉,继续执行,就能解决;

```bash
$ cd /usr/share/ansible/openshift-ansible
$ ansible-playbook -i <inventory_file> \
    playbooks/redeploy-certificates.yml
```

3、 执行完成后会发现origin-node 不能重启,原因是csr 没有被接受,这时需要在master节点上执行,手动接受.

批准所有csr

```bash
oc get csr -o name | xargs oc adm certificate approve
```



4、 备注,查看证书时详细信息

```bash
openssl x509 -noout -text -in /etc/origin/node/client-ca.crt

openssl x509 -noout -text -in /etc/origin/node/certificates/kubelet-server-current.pem


openssl x509 -noout -text -in /etc/origin/node/certificates/kubelet-client-current.pem
```

