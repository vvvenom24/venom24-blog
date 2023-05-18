---
title: "Docker Compose 安装携程 Apollo"
date: 2022-11-25T19:12:59+08:00
lastmod: 2022-11-25T19:12:59+08:00
draft: false
series: [Docker]
tags: [Docker,Apollo]
summary: "这是一份在 Ubuntu 18.04.6 LTS 操作系统上安装携程分布式配置中心 Apollo 的指南。"
---
> Operating System：Ubuntu 18.04.6 LTS
>
> Kernel：Linux 4.15.0-169-generic
>
> Architecture：x86-64

## 获取 Apollo Config Service 镜像

```shell
root@ubuntu:~# docker pull apolloconfig/apollo-configservice:latest
```

## 获取 Apollo Admin Service 镜像

```shell
root@ubuntu:~# docker pull apolloconfig/apollo-adminservice:latest
```

## 获取 Apollo Protal 镜像

```shell
root@ubuntu:~# docker pull apolloconfig/apollo-portal:latest
```

## 验证

```shell
root@ubuntu:~# docker images |grep apollo
apolloconfig/apollo-portal          latest    d8f979fd9631   5 months ago   160MB
apolloconfig/apollo-adminservice    latest    2934fe191aa5   5 months ago   176MB
apolloconfig/apollo-configservice   latest    faf09bfd7daf   5 months ago   180MB
```

## 创建挂载目录

```shell
root@ubuntu:~# mkdir -p apollo/configservice/conf apollo/configservice/logs apollo/adminservice/conf apollo/adminservice/logs apollo/portal/conf apollo/portal/logs
```

## 初始化数据库

### 初始化 [ApolloConfigDB](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/apolloconfigdb.sql)

```sql
source /your_local_path/scripts/sql/apolloconfigdb.sql
```

### 初始化 [ApolloPortalDB](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/apolloportaldb.sql)

```sql
source /your_local_path/scripts/sql/apolloportaldb.sql
```

## 修改启动文件

```yaml
  configservice:
    image: apolloconfig/apollo-configservice:latest
    container_name: configservice
    ports:
      - 7901:7901
    volumes:
      - /opt/apollo/configservice/logs:/opt/logs:rw
      - /opt/apollo/configservice/conf:/opt/conf:rw
      - /etc/localtime:/etc/localtime:ro
    network_mode: host
    restart: always
    environment:
      - SERVER_PORT=7901
      - SPRING_DATASOURCE_URL=jdbc:mysql://172.11.0.2:3306/ApolloConfigDB?characterEncoding=utf8&useSSL=false
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=root
    depends_on:
      - mysql
    deploy:
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  adminservice:
    image: apolloconfig/apollo-adminservice:latest
    container_name: adminservice
    ports:
      - 7902:7902
    volumes:
      - /opt/apollo/adminservice/logs:/opt/logs:rw
      - /opt/apollo/adminservice/conf:/opt/conf:rw
      - /etc/localtime:/etc/localtime:ro
    network_mode: host
    restart: always
    environment:
      - SERVER_PORT=7902
      - SPRING_DATASOURCE_URL=jdbc:mysql://172.11.0.2:3306/ApolloConfigDB?characterEncoding=utf8&useSSL=false
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=root
    depends_on:
      - configservice
    deploy:
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  portal:
    image: apolloconfig/apollo-portal:latest
    container_name: portal
    ports:
      - 7903:7903
    volumes:
      - /opt/apollo/portal/logs:/opt/logs:rw
      - /opt/apollo/portal/conf:/opt/conf:rw
      - /etc/localtime:/etc/localtime:ro
    network_mode: host
    restart: always
    environment:
      - SERVER_PORT=7903
      - SPRING_DATASOURCE_URL=jdbc:mysql://172.11.0.2:3306/ApolloPortalDB?characterEncoding=utf8&useSSL=false
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=root
      - APOLLO_PORTAL_ENVS=dev
      - DEV_META=http://<公网ip>:7901
    depends_on:
      - configservice
      - adminservice
    deploy:
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

## 注意

### 开放所有端口

`configservice` 服务中包含了 `Eureka` 所以它的端口必须开放给其他服务

`adminservice` 服务提供了配置的查询、更新等等操作，

`portal` 服务是 Apollo 的配置页面所以端口也是需要开放的

### 云环境部署

`ApolloConfigDB.ServerConfig` 中增加配置 `eureka.instance.ip-address：<公网ip>`