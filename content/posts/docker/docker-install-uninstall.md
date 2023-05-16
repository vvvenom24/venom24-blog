---
title: "Docker 的安装与卸载"
date: 2022-11-24T11:29:32+08:00
draft: false
series: [Docker]
tags: [Docker]
summary: "在 Ubuntu 18.04.6 LTS 上安装 docker 的步骤。"
---
> Operating System：Ubuntu 18.04.6 LTS
>
> Kernel：Linux 4.15.0-169-generic
>
> Architecture：x86-64

## 安装 docker

### 如果已存在，删除旧版本

```shell
root@ubuntu:~# apt-get remove docker docker-engine docker.io containerd runc
```

### 获取软件最新源

```shell
root@ubuntu:~# apt-get update
```

### 安装 apt 依赖包

```shell
root@ubuntu:~# apt-get -y install apt-transport-https ca-certificates curl software-properties-common
```

### 安装 GPG 证书

```shell
root@ubuntu:~# curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

### 验证

```shell
root@ubuntu:~# apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

### 设置稳定版仓库

```shell
root@ubuntu:~# add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```
### 安装 Docker Engine-Community

> Docker分为：Docker Engine - Community、Docker Engine - Enterprise和Docker Enterprise三个版本。Community 是希望开始使用 Docker 并尝试基于容器的应用程序的个人开发人员和小型团队的理想选择 ， Docker Engine - Enterprise 专为企业开发容器运行时而设计，同时考虑了安全性和企业级SLA，Docker Enterprise 专为企业开发和IT团队而设计，他们可以大规模构建，交付和运行关键业务应用程序。

- 安装最新本

  ```shell
  root@ubuntu:~# apt-get install docker-ce docker-ce-cli containerd.io
  ```

- 安装特定版本

  ```shell
  apt-cache madison docker-ce
  sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
  ```

### 验证

```shell
root@ubuntu:~# docker -v
Docker version 20.10.21, build baeda1f
```

### 配置用户到组

```shell
sudo usermod -aG docker <YOUR_USER>
```

## 卸载

### 删除安装包

```shell
root@ubuntu:~# apt-get autoremove docker docker-ce docker-engine docker.io containerd runc
```

### 删除相关配置文件

```shell
root@ubuntu:~# dpkg -l | grep docker
root@ubuntu:~# dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
```

### 卸载相关插件

```shell
root@ubuntu:~# apt-get autoremove docker-ce-*
```

### 删除相关配置

```shell
root@ubuntu:~# rm -rf /etc/systemd/system/docker.service.d
root@ubuntu:~# rm -rf /var/lib/docker
```

### 查询相关软件包

```shell
root@ubuntu:~# dpkg -l | grep docker
root@ubuntu:~# apt remove --purge xxx
```

### 验证

```shell
root@ubuntu:~# docker -v
Command 'docker' not found, but can be installed with:
apt install docker.io
please ask your administrator.
```

## 国外服务器直接安装

### 卸载当前的版本 

```shell
$ sudo apt-get remove docker docker-engine docker.io
 
$ sudo apt-get purge docker-ce docker-ce-cli containerd.io
 
$ sudo apt autoremove
 
$ sudo rm -rf /var/lib/docker
 
$ sudo rm -rf /var/lib/containerd 
```

### 安装（官网）

```shell
$ sudo apt-get update
 
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
 
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 安装引擎（默认最新版）

```shell
$  sudo apt-get update
$  sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### 查找版本对应安装

```shell
$ apt-cache madison docker-ce
```

<img src="https://cdn.jsdelivr.net/gh/vvvenom24/images/docker-install-1.png" alt="img"  /> 

Tips：
`Ubuntu 16.04 = Ubuntu-xenial`
`Ubuntu 18.04 = Ubuntu-bionic` 
`Ubuntu 20.04 = Ubuntu-focal`

选择对应版本`  5:20.10.6~3-0~ubuntu-focal `

```shell
$ sudo apt-get install docker-ce=5:20.10.6~3-0~ubuntu-focal docker-ce-cli=5:20.10.6~3-0~ubuntu-focal containerd.io
```
