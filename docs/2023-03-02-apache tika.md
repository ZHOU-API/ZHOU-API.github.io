---
title: 'apache tika'
layout: post
tags:
  - apache
category: 框架
---
apache tika：各种格式文本解析服务

<!--more-->

# 一、安装
https://hub.docker.com/r/apache/tika


## 1、docker
```
docker pull apache/tika

docker run -d --name tika -p 9998:9998 apache/tika

docker pull logicalspark/docker-tikaserver

docker run -d --name tika -p 9998:9998 logicalspark/docker-tikaserver
```

问题：
```

[0.009s][warning][os,thread] Failed to start thread "GC Thread#0" - pthread_create failed (EPERM) for attributes: stacksize: 1024k, guardsize: 4k, detached.
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Cannot create worker GC thread. Out of system resources.
# An error report file with more information is saved as:
# /tmp/hs_err_pid1.log

```