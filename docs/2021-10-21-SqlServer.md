---
title: 'Sql server'
layout: post
tags:
  - sqlserver
category: 数据库
---
关系型数据库sqlserver

<!--more-->

# 一、docker安装

```shell
docker pull exoplatform/sqlserver

docker run -d -e SA_PASSWORD='dalong!@123' -e SQLSERVER_DATABASE=demo -e SQLSERVER_USER=dalong -e SQLSERVER_PASSWORD='dalong!@123' -p 1444:1433 exoplatform/sqlserver
```

