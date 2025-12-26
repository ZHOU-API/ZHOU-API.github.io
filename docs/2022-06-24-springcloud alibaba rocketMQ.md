---
title: 'springcloud alibaba rocketMQ'
layout: post
tags:
  - java
  - springCloud
category: 框架
---
springcloud alibaba rocketMQ - 消息队列：

1:NameServer是一个非常简单的Topic路由注册中心，其角色类似Dubbo中的zookeeper，支持Broker的 动态注册与发现。

2:Broker主要负责消息的存储、投递和查询以及服务高可用保证。

3:Producer消息发布的角色，支持分布式集群方式部署。Producer通过MQ的负载均衡模块选择相应的 Broker集群队列进行消息投递，投递的过程支持快速失败并且低延迟。

4:Consumer消息消费的角色，支持分布式集群方式部署。支持以push推，pull拉两种模式对消息进行消 费。同时也支持集群方式和广播方式的消费，它提供实时消息订阅机制，可以满足大多数用户的需求。


<!--more-->

## 一、Docker 部署
### 1、创建namesrv容器
下载镜像，创建宿主机文件夹，构建容器并映射到宿主机
```

docker pull rocketmqinc/rocketmq:4.4.0

mkdir /home/rocketmq

docker run -d -p 9876:9876 \
--name rmqnamesrv \
--restart=always \
--privileged=true \
-v /home/rocketmq/data/namesrv/logs:/home/rocketmq/logs \
-v /home/rocketmq/data/namesrv/store:/home/rocketmq/store \
-e "MAX_POSSIBLE_HEAP=100000000" \
rocketmqinc/rocketmq:4.4.0 \
sh mqnamesrv

```
| 参数| 说明 | 
| ----- | ---- | 
| -d | 以守护进程的方式启动 | 
|  -p 9876:9876 | 容器内的端口9876挂载到宿主机9876 | 
| --name rmqnamesrv| 把容器的名字设置为rmqnamesrv | 
| --privileged=true| 容器具备外部资源权限| 
| -v /home/rocketmq/data/namesrv/logs:/root/logs| 挂载宿主机| 
| -v /home/rocketmq/data/namesrv/store:/root/store| 挂载宿主机| 
| -e "MAX_POSSIBLE_HEAP=100000000"| 设置容器的最大堆内存为100000000| 
| rocketmqinc/rocketmq:4.4.0 | 使用的镜像名称| 
| sh mqnamesrv | 启动namesrv服务| 


### 2、创建broker容器
创建宿主机目录、创建初始化配置文件
```
mkdir /home/rocketmq/conf 

vim /home/rocketmq/conf/broker.conf

```

broker.conf
```
# 所属集群名称，如果节点较多可以配置多个
brokerClusterName = DefaultCluster 
#broker名称，master和slave使用相同的名称，表明他们的主从关系 
brokerName = broker-a 
#0表示Master，大于0表示不同的
slave brokerId = 0 
#表示几点做消息删除动作，默认是凌晨4点 
deleteWhen = 04 
#在磁盘上保留消息的时长，单位是小时 
fileReservedTime = 48 
#有三个值：SYNC_MASTER，ASYNC_MASTER，SLAVE；同步和异步表示Master和Slave之间同步数据的机 制；
brokerRole = ASYNC_MASTER 
#刷盘策略，取值为：ASYNC_FLUSH，SYNC_FLUSH表示同步刷盘和异步刷盘；SYNC_FLUSH消息写入磁盘后 才返回成功状态，ASYNC_FLUSH不需要；
flushDiskType = ASYNC_FLUSH 
# 设置broker节点所在服务器的ip地址 
brokerIP1 = 172.17.181.4
#剩余磁盘比例 
diskMaxUsedSpaceRatio=99

```
启动命令
```
docker run -d \
-p 10911:10911 -p 10909:10909 \
--name rmqbroker \
--restart=always \
--privileged=true \
--link rmqnamesrv:namesrv \
-v /home/rocketmq/data/broker:/root/logs \
-v /home/rocketmq/data/broker/store:/root/store \
-v /home/rocketmq/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf \
-e "NAMESRV_ADDR=namesrv:9876" \
-e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq:4.4.0 \
sh mqbroker -c /opt/rocketmq-4.4.0/conf/broker.conf


docker run -d \
-p 10911:10911 -p 10909:10909 \
--name rmqbroker \
--restart=always \
--privileged=true \
--link rmqnamesrv:namesrv \
--mount source=/home/rocketmq/data/broker/logs,target=/home/rocketmq/logs \
--mount source=/home/store,target=/home/rocketmq/store \
-v /home/rocketmq/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf \
-e "NAMESRV_ADDR=namesrv:9876" \
-e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq:4.4.0 \
sh mqbroker -c /opt/rocketmq-4.4.0/conf/broker.conf

```

| 参数| 说明 | 
| ----- | ---- | 
|  -p 10911:10911 -p 10909:10909 | 端口映射 | 
| --name rmqbroker | 把容器的名字设置为rmqbroker | 
| --privileged=true| 容器具备外部资源权限| 
| --link rmqnamesrv:namesrv | 和rmqnamesrv容器通信 | 
| -v ... | 挂载宿主机| 
| -e "NAMESRV_ADDR=namesrv:9876"| 指定namesrv的地址为本机namesrv的ip地址:9876 |
| -e “MAX_POSSIBLE_HEAP=200000000” rocketmqinc/rocketmq sh mqbroker| 指定broker服务的最大堆内存|  
| rocketmqinc/rocketmq:4.4.0 | 使用的镜像名称| 
| sh mqbroker -c /opt/rocketmq-4.4.0/conf/broker.conf| 启动broker服务并指定配置文件运行（虚拟机内部目录）| 


### 3、创建控制台
注意需要开放防火墙端口（9876/10911）、或者关闭防火墙才能访问namesrv和broker服务、范围9999端口web服务测试控制台
```
docker pull styletang/rocketmq-console-ng

docker run -d \
--restart=always \
--name rmqadmin \
-e "JAVA_OPTS=-Drocketmq.namesrv.addr=172.17.181.4:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
-p 9999:8080 \
-t styletang/rocketmq-console-ng

```

## 二、springboot 配置使用

### 依赖

生产者和消费者都需添加依赖
```xml

<!-- rocketMQ -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
</dependency>

```

### 生产者
#### yml配置
添加mq服务
```
spring:
  cloud:
    # mq 配置
    stream:
      rocketmq:
        binder:
          name-server: 172.17.181.4:9876

```
#### code使用
```java
private StreamBridge streamBridge;

//对齐topic_name发送msg消息
streamBridge.send(topic_name,msg);

```

### 消费者
#### yml配置
添加mq服务：message即为topic_name
```
spring:
   cloud:
    # mq 配置
    stream:
      rocketmq:
        binder:
          name-server: 172.17.181.4:9876
      #使用函数式绑定topic名称
      function:
        definition: message
      #指定消费者的topic和group
      bindings:
        message-in-0:
          destination: message
          group: message-group
          content-type: application/json


```
#### code使用
```java
import com.baidu.sz.kap.service.MessageService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.messaging.Message;
import org.springframework.stereotype.Service;

import java.util.function.Consumer;


@RequiredArgsConstructor(onConstructor = @__(@Autowired))
@Slf4j
@Service
public class MessageConsumer {

    private final MessageService messageService;

    /**
     * 方法名为topic名称
     *
     * @return
     */
    @Bean
    public Consumer<Message<String>> message() {
        return message -> System.out.println(message.getPayload());
    }
}

```
