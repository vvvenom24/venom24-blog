---
title: "Docker Compose 安装 MySQL8"
date: 2022-12-15T19:05:29+08:00
lastmod: 2022-12-15T19:05:29+08:00
draft: false
series: [MySQL]
tags: [Docker,MySQL]
summary: "本文介绍了在 Ubuntu 18.04.6 LTS 操作系统上拉取 MySQL 镜像、创建相关挂载目录和配置文件、编辑启动的 yml 文件、启动和验证 MySQL 容器，并进入容器内登录 MySQL。"
---
> Operating System：Ubuntu 18.04.6 LTS
>
> Kernel：Linux 4.15.0-169-generic
>
> Architecture：x86-64

## 拉取 MySQL 镜像

```bash
root@ubuntu:~# docker pull mysql:8.0.31
```

## 验证

```bash
root@ubuntu:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
mysql        8.0.31    3842e9cdffd2   8 days ago   538MB
```

## 创建 MySQL 相关挂载目录

```bash
root@ubuntu:~# mkdir -p mysql/data mysql/conf mysql/log mysql/mysql-files
```

## 创建 MySQL 配置文件

```shell
root@ubuntu:~# vim mysql/conf/my.cnf
```

```
[mysqld]
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files

character-set-server=utf8
default_authentication_plugin=mysql_native_password
expire_logs_days=7
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
max_connections=1000

pid-file=/var/run/mysqld/mysqld.pid
[client]
socket=/var/run/mysqld/mysqld.sock

[mysql]
default-character-set=utf8

#!includedir /etc/mysql/conf.d/
```

## 编辑 MySQL 启动的 yml 文件

```yaml
mysql:
    image: mysql:8.0.31
    container_name: mysql
    restart: always
    deploy:
      resources:
        limits:
          memory: 1024m
    user: root
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - 3306:3306
    volumes:
      - /opt/mysql/data:/var/lib/mysql:rw
      - /opt/mysql/conf/my.cnf:/etc/mysql/my.cnf:rw
      - /opt/mysql/log:/var/log/mysql:rw
      - /opt/mysql/mysql-files:/var/lib/mysql-files:rw
```

## 启动

```shell
root@ubuntu:~# docker-compose up -d
```

## 验证

```shell
root@ubuntu:~# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS         PORTS                                                  NAMES
49e9b0115a54   mysql:8.0.31   "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds   33060/tcp, 0.0.0.0:3006->3306/tcp, :::3006->3306/tcp   mysql
```

进入容器内

```shell
root@ubuntu:~# docker exec -it mysql bash
```

登录 mysql

```shell
bash-4.4# mysql -uroot -p
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.31 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```