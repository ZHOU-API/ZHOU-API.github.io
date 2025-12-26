---
title: 'springcloud gateway'
layout: post
tags:
  - java
  - springCloud
category: 框架
---
springcloud alibaba gateway - 微服务服务网关框架。

<!--more-->

## 依赖

注意网关配置负载时，需依赖负载插件Ribbon/loadbalancer
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>${spring-cloud.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>

<!--gateway-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--<dependency>-->
    <!--<groupId>org.springframework.cloud</groupId>-->
    <!--<artifactId>spring-cloud-starter-openfeign</artifactId>-->
    <!--<version>3.0.3</version>-->
<!--</dependency>-->
<!--SpringCloud Feign在Hoxton.M2 RELEASED版本之后不再使用Ribbon而是使用spring-cloud-loadbalancer-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-loadbalancer</artifactId>
</dependency>


```

## 配置

```
spring:
  profiles:
    active: dev
  application:
    name: gateway-server
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启注册中心路由功能
      routes:
      - id: approval-server
        uri: lb://approval-server
        predicates:
        - Path=/approval/**

      - id: article-server
        uri: lb://article-server
        predicates:
        - Path=/article/**,/boardAudit/article/**

      - id: base-server1
        uri: lb://base-server
        predicates:
        - Path=/advert/**,/admin/advert/**,/dict/**,/message/**,/notice/**,/admin/notice/**

      - id: board-server
        uri: lb://board-server
        predicates:
        - Path=/admin/board/**,/boardAdmin/boardUser/**

      - id: collect-server
        uri: lb://collect-server
        predicates:
        - Path=/collect/**

      - id: comment-server
        uri: lb://comment-server
        predicates:
        - Path=/comment/**

      - id: exam-server
        uri: lb://exam-server
        predicates:
        - Path=/exam/**,/boardAdmin/exam/**,/boardAdmin/examRecord/**,/boardAdmin/paper/**

      - id: integral-server
        uri: lb://integral-server
        predicates:
        - Path=/boardAdmin/integral/**,/admin/integral/**

      - id: learn-server
        uri: http://172.17.181.4:8018/
        predicates:
        - Path=/learnMap/**,/boardAdmin/learnMap/**,/boardAdmin/stage/**

      - id: medal-server
        uri: lb://medal-server
        predicates:
        - Path=/medal/**,/admin/medal/**

      - id: portal-server
        uri: lb://portal-server
        predicates:
        - Path=/file/**

      - id: search-server
        uri: lb://search-server
        predicates:
        - Path=/search/**

      - id: spider-server
        uri: lb://spider-server
        predicates:
        - Path=/spider/**

      - id: subject-server
        uri: lb://subject-server
        predicates:
        - Path=/precinct/**,/subject/**

      - id: user-server
        uri: lb://user-server
        predicates:
        - Path=/login/**,/user/**,/admin/user/**

      - id: vote-server
        uri: lb://vote-server
        predicates:
        - Path=/vote/**

      - id: intellect-images-server
        uri: lb://images-server
        predicates:
        - Path=/intellect/**


```
