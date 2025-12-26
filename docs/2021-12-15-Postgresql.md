---
title: 'PostgreSql安装'
layout: post
tags:
  - PostgreSql
category: 数据库
---
PostgreSql安装.

<!--more-->

# 一、PostgreSql安装

```shell
docker pull postgres

docker run -it --name postgres --restart always --privileged=true -e POSTGRES_PASSWORD='root' -e ALLOW_IP_RANGE=0.0.0.0/0 -v /home/postgres:/var/lib/postgresql  -p 5432:5432 -d postgres

#--privileged=true使用该参数，container内的root拥有真正的root权限
#POSTGRES_PASSWORD：数据库密码
#-e ALLOW_IP_RANGE=0.0.0.0/0，这个表示允许所有ip访问，如果不加，则非本机 ip 访问不了
```

