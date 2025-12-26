---
title: 'Golang'
layout: default
parent: Go
---

Goland笔记

<!--more-->

# 一、安装Golang的版本控制器

## g地址
```
https://github.com/voidint/g
```

## windows配置
```
增加环境变量
G_HOME：C:\Users\~\.g
G_MIRROR：https://golang.google.cn/dl/
GOROOT：%G_HOME%\go

修改PATH环境变量
1.增加g地址，例如：D:\Program Files\g
2.增加go配置：%GOROOT%\bin
```

## 使用命令
```
g h
```


# 二、安装Golang环境

## 创建项目
```
go env

go env -w GO111MODULE=on

go env -w GOPROXY=https://goproxy.cn,direct

go mod init [项目名称]
```

# 三、Goland开发工具设置

## GOROOT设置
1、使用go env 查看GOROOT位置

2、编辑C:\Program Files\Go\src\runtime\internal\sys\zversion.go文件末行增加
```
const TheVersion = `go1.17.5`
```

## 代理设置
```
https://goproxy.cn,direct 
```

![20220810113053](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/20220810113053.png)


## 清理爆红
![20220810113518](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/20220810113518.png)

## 编译
```

SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go env -w CGO_ENABLED=0 GOOS=linux GOARCH=amd64 

go env -w CGO_ENABLED=0 GOOS=windows
go build main.go

#整理mod依赖
go build -mod=mod
```


## go 部署 k8s


### 1、编写Dockerfile用于构建images
```
FROM golang:latest AS build

WORKDIR /test
COPY . /test
RUN go env -w GOPROXY=https://goproxy.cn,direct
RUN CGO_ENABLED=0 go build -v -o main .

FROM alpine AS api
RUN mkdir /app
COPY --from=build /test/main /app
ADD conf/app.ini /test/main/app/conf/

WORKDIR /app
ENTRYPOINT ["./main", "-v" ,"1.0.1"]

```
### 2、编写.yaml用于k8s构建容器
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-deployment
  labels:
    app: go
spec:
  selector:
    matchLabels:
      app: go
  replicas: 3
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: go
    spec:
      serviceAccountName: default
      imagePullSecrets:
        - name: alidockerregistryssecret
      containers:
        - image: registry.cn-hangzhou.aliyuncs.com/qzliao/app:h5-cc-1.0.1
          name: go
          imagePullPolicy: Always
          command: ["./main","-v","v1.0.1"]
          ports:
            - containerPort: 8404
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: go-service
  labels:
    app: go
spec:
  selector:
    app: go
  ports:
    - name: go-port
      protocol: TCP
      port: 8404
      targetPort: 8404
      nodePort: 31404
  type: NodePort

```

#### 注意：
拉取镜像的secret为alidockerregistryssecret，是阿里云私有镜像仓库的安全配置，配置方式https://www.cnblogs.com/wolfstark/p/16217928.html



#### 配置阿里云私有镜像仓库

```
kubectl create secret docker-registry alidockerregistryssecret --docker-server=registry.cn-hangzhou.aliyuncs.com --docker-username=[] --docker-password=[]

kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "alidockerregistryssecret"}]}'
```



### 3、编写shell用于执行构建镜像、推送镜像
