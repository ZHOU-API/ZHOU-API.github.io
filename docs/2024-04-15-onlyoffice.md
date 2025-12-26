---
title: 'onlyoffice'
layout: post
tags:
  - office
category: 服务
---
onlyoffice:用于在一处管理文档、项目、客户关系和电子邮件的协作系统

<!--more-->

## 一、Docker 部署
### 1、前置环境
```
#MYSQL
mkdir -p "/data/onlyoffice/mysql/conf.d";
mkdir -p "/data/onlyoffice/mysql/data";
mkdir -p "/data/onlyoffice/mysql/initdb";

#服务器数据
mkdir -p "/data/onlyoffice/CommunityServer/data";
mkdir -p "/data/onlyoffice/CommunityServer/logs";
mkdir -p "/data/onlyoffice/CommunityServer/letsencrypt";

#文档数据
mkdir -p "/data/onlyoffice/DocumentServer/data";
mkdir -p "/data/onlyoffice/DocumentServer/logs";

#邮件数据
mkdir -p "/data/onlyoffice/MailServer/data/certs";
mkdir -p "/data/onlyoffice/MailServer/logs";

# 控制面板
mkdir -p "/data/onlyoffice/ControlPanel/data";
mkdir -p "/data/onlyoffice/ControlPanel/logs";

```

### 2、构建
```
# 文档编辑器
docker run -itd -p 8063:80 --name onlyoffice-document-server \
    -e JWT_SECRET=false \
    -v /data/onlyoffice/DocumentServer/logs:/var/log/onlyoffice  \
    -v /data/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data  onlyoffice/documentserver:7.1

# community社区版
docker run --net onlyoffice -i -t -d --privileged --restart=always --name onlyoffice-community-server -p 8063:80 -p 5222:5222 \
 -e MYSQL_SERVER_ROOT_PASSWORD=qazwsx123! \
 -e MYSQL_SERVER_DB_NAME=onlyoffice \
 -e MYSQL_SERVER_HOST=172.17.180.145 \
 -e MYSQL_SERVER_USER=onlyoffice_user \
 -e MYSQL_SERVER_PASS=onlyoffice_pass \
 -v /data/onlyoffice/CommunityServer/data:/var/www/onlyoffice/Data \
 -v /data/onlyoffice/CommunityServer/logs:/var/log/onlyoffice \
 -v /data/onlyoffice/CommunityServer/letsencrypt:/etc/letsencrypt \
 -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
 onlyoffice/communityserver


```

### 3、访问
```
host:port
```

### 4、初始化