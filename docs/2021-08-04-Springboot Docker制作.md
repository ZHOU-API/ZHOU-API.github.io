---
title: 'Springboot Docker制作'
layout: post
tags:
  - java
  - docker
category: 运维
---
Springboot Docker制作java镜像文件。

<!--more-->

# 思路

创建纯java8环境，通过映射磁盘更新app.jar和start.sh，实现便捷docker容器中启动jar

# 准备工作

1.app/app.jar 运行的jar包

2.app/start.sh 容器启动时运行的脚本

3.Dockerfile 构建镜像

4.jdk文件夹

![20210804163921](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/20210804163921.png)

### Dockerfile

* 离线jdk
```dockerfile
FROM centos:centos7.5.1804

ADD jdk1.8.0_291/ /home/jdk1.8.0_291/

ENV JAVA_HOME /home/jdk1.8.0_291/
ENV JRE_HOME=${JAVA_HOME}/jre
ENV CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
ENV PATH=${JAVA_HOME}/bin:$PATH

VOLUME ["/home/app/"]

WORKDIR /home/app/

RUN export LANG=zh_CN.UTF-8

RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

CMD ["sh","/home/app/start.sh"]
```
* openjdk
```
# 使用 OpenJDK 8 镜像作为基础镜像
FROM openjdk:8-jre-alpine

# 设置中国时区(需联网)
RUN apk add --no-cache tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# 设置容器内的工作目录
WORKDIR /app

# 将当前目录下的 app.jar 和 start.sh 复制到容器中的 /app 目录
COPY app.jar start.sh /app/

# 添加权限给启动脚本
RUN chmod +x start.sh

# 定义容器启动时执行的命令
CMD ["sh", "start.sh"]

```


### start.sh

```shell
#!/bin/bash
JAVA_OPTS="-Xmx3550m -Xms3550m -Xmn1256m -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -XX:SurvivorRatio=6 -XX:ParallelGCThreads=2 -XX:+UseConcMarkSweepGC  -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintGCCause -XX:+UseGCLogFileRotation  -Xloggc:./log/gc-%t.log -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dsun.jnu.encoding=UTF-8 -Dfile.encoding=UTF-8"

DIRECTORY="log"

if [ ! -d "$DIRECTORY" ]; then
    mkdir "$DIRECTORY"
    echo "Created DIRECTORY: $DIRECTORY in the current location. $PWD"
else
    echo "Directory $DIRECTORY already exists in the current location. $PWD"
fi

CURRENT_DATE=$(date '+%Y-%m-%d')

LOG_FILE="$DIRECTORY/app_$CURRENT_DATE.log"

java -jar ${JAVA_OPTS} app.jar --spring.profiles.active=prod > "$LOG_FILE" 2>&1


```

# 构建镜像

```shell
docker build -t [imageName:version] -f Dockerfile .
```

# 创建容器
注：宿主机的文件夹下，结构必须是：
app
- log
- start.sh
- app.jar

```shell
docker create -it --name [appName] --net=host -v [/mydir]:/home/app  [imageName:version]

docker create -it --name [appName] --net=host -v [/mydir]:/app  [imageName:version]
```

# 启动容器

```shell
docker start [appName]
```

# jar瘦身方案

## maven打包外部依赖
pom.xml
```

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <executable>true</executable>
                <layout>ZIP</layout>
                <includes>
                    <include>
                        <groupId>${groupId}</groupId>
                        <artifactId>${artifactId}</artifactId>
                    </include>
                </includes>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>${project.build.directory}/lib</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>


```

## 启动时指定依赖目录
```
java -jar -Dloader.path="./lib/" xxx.jar
```
