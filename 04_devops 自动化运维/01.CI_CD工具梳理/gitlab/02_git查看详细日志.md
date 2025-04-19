---
title: git 查看详细日志
description: 
published: true
date: 2023-04-17T03:32:19.196Z
tags: ci_cd
editor: markdown
dateCreated: 2023-04-17T03:32:19.195Z
---

# git clone 详细日志

背景

git 在操作的过程中,没办法看到详细的日志; 现在可以通过以下命令, 查看git clone 的详细日志

```bash
GIT_CURL_VERBOSE=1 GIT_TRACE=1 git clone ...

```

