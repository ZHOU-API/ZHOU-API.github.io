---
title: 'Linux Firewall 常用命令'
layout: post
tags:
  - linux
category: 运维
---
Linux Firewall 常用命令。

<!--more-->

# 开启防火墙
```shell
systemctl start firewalld.service
```



# 防火墙开机启动
```shell
systemctl enable firewalld.service
systemctl disable firewalld.service
```



# 关闭防火墙
```shell
systemctl stop firewalld.service
```



# 查看防火墙状态
```shell
firewall-cmd --state
```



# 查看现有的规则
```shell
iptables -nL
firewall-cmd --zone=public --list-ports
```



# 重载防火墙配置
```shell
firewall-cmd --reload
```



# 添加单个单端口
```shell
firewall-cmd --permanent --zone=public --add-port=81/tcp
```



# 添加多个端口
```shell
firewall-cmd --permanent --zone=public --add-port=8080-8083/tcp
```



# 删除某个端口
```shell
firewall-cmd --permanent --zone=public --remove-port=81/tcp
```



# 针对某个 IP开放端口
```shell
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.142.166" port protocol="tcp" port="6379" accept"
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.233" accept"
```



# 删除某个IP
```shell
firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="192.168.1.51" accept"
```



# 针对一个ip段访问
```shell
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.0/16" accept"
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="9200" accept"
```



# 执行重载
```shell
firewall-cmd --reload
```

