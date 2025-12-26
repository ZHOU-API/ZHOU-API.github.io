---
title: '2021-08-04-Linux Ntpdate'
---
Linux ntpdate 实现时间同步。

<!--more-->

# 安装

```shell
yum -y install ntpdate
```

# 同步

```shell
ntpdate 0.centos.pool.ntp.org
```



## 更改系统时区

```shell
#查看时区
ls -l /etc/localtime
#更新
sudo rm -f /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

