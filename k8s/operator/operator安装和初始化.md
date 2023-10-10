---
title: opertor安装和初始化
description: 
published: true
date: 2023-02-21T07:25:28.920Z
tags: k8s
editor: markdown
dateCreated: 2023-02-21T07:25:28.920Z
---

# operator 安装

## MAC

```bash
brew install operator-sdk
```



## Linux 安装

> 安装过程需要科学上网

1、 获取本平台信息

```bash
export ARCH=$(case $(uname -m) in x86_64) echo -n amd64 ;; aarch64) echo -n arm64 ;; *) echo -n $(uname -m) ;; esac)
export OS=$(uname | awk '{print tolower($0)}')
```



2、 在gitlab 下载合适的二进制文件

```bash
export OPERATOR_SDK_DL_URL=https://github.com/operator-framework/operator-sdk/releases/download/v1.27.0
curl -LO ${OPERATOR_SDK_DL_URL}/operator-sdk_${OS}_${ARCH}
```



3、验证下载的二进制文件

从以下位置导入 operator-sdk 发布 GPG 密钥`keyserver.ubuntu.com`：

```bash
gpg --keyserver keyserver.ubuntu.com --recv-keys 052996E2A20B5C7E
```

4、下载校验和文件及其签名，然后验证签名：

```bash
curl -LO ${OPERATOR_SDK_DL_URL}/checksums.txt
curl -LO ${OPERATOR_SDK_DL_URL}/checksums.txt.asc
gpg -u "Operator SDK (release) <cncf-operator-sdk@cncf.io>" --verify checksums.txt.asc
```



5、确保校验匹配

```bash
grep operator-sdk_${OS}_${ARCH} checksums.txt | sha256sum -c -
```

6、安装二进制文件

```bash
chmod +x operator-sdk_${OS}_${ARCH} && sudo mv operator-sdk_${OS}_${ARCH} /usr/local/bin/operator-sdk

```

