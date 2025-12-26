---
title: 'OLAP数据库 Clickhouse'
layout: post
tags:
  - clickhouse
category: 数据库
---
OLAP数据库 Clickhouse 安装和使用。

<!--more-->

# 一、docker部署

```shell
docker pull yandex/clickhouse-server

docker run -d --name clickhouse-server --ulimit nofile=262144:262144 -p 8123:8123 -p 9000:9000 -p 9009:9009 yandex/clickhouse-server

docker run -itd --restart=always --privileged --name clickhouse-server --ulimit nofile=262144:262144 --net=host --volume=/data/clickhouse:/var/lib/clickhouse yandex/clickhouse-server

docker run --rm -e CLICKHOUSE_DB=my_database -e CLICKHOUSE_USER=username -e CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT=1 -e CLICKHOUSE_PASSWORD=password -p 9000:9000/tcp clickhouse/clickhouse-server


```

## 修改密码
```shell
docker exec -it clickhouse-server /bin/bash

apt-get update
apt-get install vim -y

PASSWORD=$(base64 < /dev/urandom | head -c8); echo "你的密码"; echo -n "你的密码" | sha256sum | tr -d '-'

## /etc/clickhouse-server/users.xml
<password> 密码 </password>
<password_sha256_hex> 密码密文 </password_sha256_hex>

clickhouse-client -h 127.0.0.1 -d default -m -u default --password '你的密码'
```

## 注意
driver有专门驱动包，mysql driver端口连接默认为9004，postgresql是9005