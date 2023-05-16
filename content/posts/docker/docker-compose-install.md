---
title: "安装 Docker Compose"
date: 2022-11-24T19:01:11+08:00
draft: false
series: [Docker]
tags: [Docker]
summary: "这篇文章介绍了在 Ubuntu 18.04.6 LTS 操作系统上安装 Docker Compose 的步骤，包括从 GitHub 下载和从国内高速下载，以及授权和验证。最终确认安装成功的版本是 v2.6.1。"
---
> Operating System：Ubuntu 18.04.6 LTS
>
> Kernel：Linux 4.15.0-169-generic
>
> Architecture：x86-64

## 安装

 [Docker Compose ](https://docs.docker.com/compose/install/)存放在Github

```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

国内推荐使用 [~~高速安装Docker Compose~~](https://get.daocloud.io/)

```shell
root@ubuntu:~# curl -L https://get.daocloud.io/docker/compose/releases/download/v2.6.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

## 授权

```shell
root@ubuntu:~# chmod +x /usr/local/bin/docker-compose
```

## 验证

```shell
root@ubuntu:~# docker-compose -v
Docker Compose version v2.6.1
```