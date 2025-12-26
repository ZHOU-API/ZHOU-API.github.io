---
title: 'Docker Install'
layout: post
tags:
  - docker
category: 运维
---
docker安装过程。

<!--more-->

# 一、编译环境

```shell
yum -y install gcc gcc-c++

```



# 二、卸载旧版本

```shell
yum remove docker \ docker-client \ docker-client-latest \ docker-common \ docker-latest \ docker-latest-logrotate \ docker-logrotate \ docker-selinux \ docker-engine-selinux \ docker-engine
```



# 三、安装依赖包

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```



# 四、设置stable镜像仓库

```shell
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

或者

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```



# 五、更新yum软件包索引

```shell
yum makecache fast
```



# 六、安装

查看版本

```shell
yum list docker-ce.x86_64  --showduplicates | sort -r
```

安装指定版本

```shell
yum -y install docker-ce-18.03.1.ce-1.el7.centos docker-ce-cli-18.03.1.ce-1.el7.centos containerd.io
```



# 七、启动并设置开机自启

```shell
systemctl start docker
systemctl enable docker
```

# 八、配置新的网卡避免路由冲突
关闭docker
vim /etc/docker/daemon.json

```
{
  "bip": "192.168.0.1/16",
  "registry-mirrors": [
    "https://registry.aliyuncs.com"
  ]
}
```



重启验证
```
systemctl daemon-reload

systemctl restart docker

ifconfig

route -n
```


# 九、配置新的存储未知避免空间不足

```
#停止服务
systemctl stop docker
service docker stop

#创建新路径
mkdir -p /home/docker/lib/

#迁移数据
rsync  -avz  /var/lib/docker/ /home/docker/lib/

```

创建配置文件
vim /etc/docker/daemon.json

```
{
  "bip": "192.168.0.1/16",
  "data-root": "/home/docker/lib",
  "registry-mirrors": [
    "https://registry.aliyuncs.com"
  ]
}
```


重启并验证
```
systemctl  daemon-reload

systemctl  restart  docker

docker info
```

# 十、离线使用手动配置docker0网桥
具体是因为Docker的网络控制器通过iptables防火墙对容器网络进行管理，而防火墙默认是禁止所有非本地流量的。对于Docker而言，在第一次启动时会自动在iptables中添加相关规则，但离线安装的Docker如果是第一次启动，就会出现添加规则失败，从而导致网络控制器无法启动的问题。具体表现就是当用户启动Docker时，Docker将会根据本地的IP地址，自动创建一个名为docker0的网桥，如果该操作失败，Docker则无法启动。
```
sudo ip link add name docker0 type bridge
sudo ip addr add dev docker0 172.17.0.1/16
sudo ip link set dev docker0 up
```

# 十一、离线传输镜像
```
docker save mysql:5.7 > /home/soft/mysql5.7.tar
docker load < /home/soft/mysql5.7.tar
```