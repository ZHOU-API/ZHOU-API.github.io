---
title: 'docker+spug+springboot快速部署'
layout: post
tags:
  - java
  - docker
category: 运维
---
docker+spug+springboot实现快速搭建自动化部署环境快速上线。

<!--more-->

# 一、前置环境
1、docker
2、spug
3、gitlab（版本管理）
思路：
先在docker中创建应用容器，再通过spug从git拉取代码、并发送到对应主机，编程成可执行文件后，替换容器内的代码，最后重启服务实现自动化部署。

## 构建容器

参考springboot构建docker容器应用：
1、保证容器能使用命令启动：docker restart appname
2、构建容器使用-v 映射程序文件到宿主机，便于脚本执行替换文件命令

## 配置spug

1、配置应用
![infoflow_2022-4-21_10-43-31](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/infoflow_2022-4-21_10-43-31.png)

2、新建发布
确定目标主机、git地址、发布模式
![infoflow_2022-4-21_11-6-14](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/infoflow_2022-4-21_11-6-14.png)

3、代码检出触发执行脚本
![infoflow_2022-4-21_11-7-54](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/infoflow_2022-4-21_11-7-54.png)
```
#代码检出后执行
mvn clean package
```

4、确定目标主机配置、发布代码后执行脚本

![infoflow_2022-4-21_11-9-30](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/infoflow_2022-4-21_11-9-30.png)

部署路径指的目标主机的地址
存储路径指目标主机存放历史发布文件的地址，可指定存放版本数量
```
#应用发布后执行，执行发布后，路径节点再部署路径中，将部署路径的可执行文件替换到容器中，并重启容器
\cp ./target/app.jar /www/app/app/app.jar
docker restart app
```

