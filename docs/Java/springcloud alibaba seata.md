---
title: 'springcloud alibaba seata'
layout: default
parent: Java
---
springcloud alibaba seata- 微服务分布式事务框架。官方文档：https://seata.io/zh-cn/docs/ops/deploy-by-docker.html

<!--more-->
# 一、创建数据库
## seata库
```
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;


```

## 客户端所有库必须构建undo_log表

```
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```

# 二、nacos配置命名空间

![infoflow_2022-4-21_14-29-49](https://raw.githubusercontent.com/QinL233/QinL233.github.io/master/images/infoflow_2022-4-21_14-29-49.png)

命名空间：SEATA_GROUP
命名空间ID（自动生成，需配置到seata中）：

# 三、docker部署

## 先启动容器，将配置文件目录copy到宿主机并修改关键文件
```
docker pull seataio/seata-server:1.4.2

## 运行容器
docker run --name seata -p 8091:8091 -e SEATA_IP=192.168.1.1 -d seataio/seata-server:1.4.2
## 将容器中的配置拷贝到/usr/local/seata-1.3.0
docker cp seata:/seata-server /home/seata/server
## 进入容器
docker exec -it seata sh
## 完成后就会在/usr/local/seata-1.3.0出现容器的配置，我们现在可以将原来容器停止并删除
docker stop seata
docker rm seata

```


## 配置文件
创建配置文件并映射到容器中，启动容器时指定配置文件

### file.conf 配置mysql，redis分布式全局环境

```
## transaction log store, only used in seata-server
store {
  ## store mode: file、db、redis
  mode = "db" ## 原来为file
 
  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    maxBranchSessionSize = 16384
    # globe session size , if exceeded throws exceptions
    maxGlobalSessionSize = 512
    # file buffer size , if exceeded allocate new buffer
    fileWriteBufferCacheSize = 16384
    # when recover batch read size
    sessionReloadReadSize = 100
    # async, sync
    flushDiskMode = async
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    ## 因为设置为db，所以需要选择数据库，这里设置数据库及密码，seata是需要创建的，默认的表是通过脚本运行得到的
    url = "jdbc:mysql://172.17.180.145:3306/seata?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai"
    user = "root"
    password = "root"
    minConn = 5
    maxConn = 30
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }

  ## redis store property
  redis {
    host = "172.17.180.145"
    port = "6379"
    password = ""
    database = "7"
    minConn = 1
    maxConn = 10
    queryLimit = 100
  }
}

```

### registry.conf 配置注册中心
```

registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  ## 我们这里使用nacos作为注册中心，所以需要设置type为nacos并设置nacos的属性
  type = "nacos"
  nacos {
    application = "seata-server"
    serverAddr = "172.17.180.145:8848"
    group = "SEATA_GROUP"
    ## 这里的namespace是你nacos服务的namespace的data-id，设置那里，到时就会在那个namespace中
    namespace = "2b8b1bcd-9b57-4271-bea6-05fdbb42be6f"
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "file"

  file {
    name = "file:/root/seata-config/file.conf"
  }
}

```

## 启动容器

```shell

docker run -itd --name seata --restart=always --net=host \
        -e SEATA_IP=172.17.180.145 \
        -e SEATA_PORT=8091 \
        -e STORE_MODE=db \
        -e SERVER_NODE=1 \
        -e SEATA_CONFIG_NAME=file:/root/seata-config/registry \
        -v /home/seata/config:/root/seata-config  \
        -v /home/seata/logs:/root/logs \
        -d seataio/seata-server:1.4.2

```

--privileged=true	使得容器内的root拥有真正外部root的权限
--link [容器名] 使当前容器可以通过容器名访问另一个容器
--restart=always	是的容器在docker启动时自动启动
SEATA_IP是seata服务ip
SEATA_PORT是seata服务端口

# 四、坑
## mysql版本驱动包需要保持一致
容器默认使用的驱动包在 /seata-server/libs 目录下，默认为5.0，使用mysql8.0连接需要删除旧的驱动使用8.0+的驱动包

```
# 将容器中的jdbc复制出来
docker cp seata:/seata-server/libs/jdbc/mysql-connector-java-8.0.19.jar ./

#替换原/seata-server/libs下的jdbc.jar
docker cp mysql-connector-java-8.0.19.jar seata:/seata-server/libs

#重启进入容器并删除旧的驱动包
docker restart seata
docker exec -it seata sh
rm -f libs/mysql-connector-java-5
```

## fileListener execute error:null
