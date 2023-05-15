---
title: "Ubuntu 安装Node.js"
date: 2022-11-23T19:22:07+08:00
draft: false
series: [Ubuntu]
tags: [Ubuntu,Node.js]
summary: "在Ubuntu 18.04上安装Node.js 18.x会出现GLIBC_2.28版本错误，因此需要卸载Node.js并删除相关文件，然后安装Node.js 16.x和yarn。安装完成后，可以验证版本并切换到淘宝镜像源。"
---
> Operating System：Ubuntu 18.04.6 LTS
>
> Kernel：Linux 4.15.0-169-generic
>
> Architecture：x86-64

- Ubuntu 18.04 安装Node.js 18.x 后报错：

```shell
root@ubuntu:~# node -v
node: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.28' not found (required by node)
```

因为Ubuntu18.04 默认的glibc 版本为2.27

```shell
root@ubuntu:~# ldd --version
Copyright (C) 2018 Free Software Foundation, Inc.
```

- 这里卸载Node.js 重新安装

```shell
root@ubuntu:~# sudo apt remove nodejs
root@ubuntu:~# sudo apt remove npm
root@ubuntu:~# sudo apt autoremove
```

删除`/usr/local/lib`、`/usr/local/include` 和`/usr/local/bin` 下所有`node` 和`node_modules` 文件

- 安装node.js 16.x

```shell
root@ubuntu:~# curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
root@ubuntu:~# sudo apt-get install -y nodejs
```

- 验证

```shell
root@ubuntu:~# node -v
v16.18.1
root@ubuntu:~# npm -v
8.19.2
root@ubuntu:~# npx -v
8.19.2
```

- 安装yarn

```shell
root@ubuntu:~# curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
ok
root@ubuntu:~# echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
deb https://dl.yarnpkg.com/debian/ stable main
root@ubuntu:~# apt update
root@ubuntu:~# apt install yarn
```

- 验证

```shell
root@ubuntu:~# yarn --version
1.22.19
```

- 查看当前yarn 镜像源

```shell
root@ubuntu:~# yarn config get registry
https://registry.yarnpkg.com
```

- 切换淘宝镜像源

```shell
root@ubuntu:~# yarn config set registry 'https://registry.npm.taobao.org'
yarn config v1.22.19
success Set "registry" to "https://registry.npm.taobao.org".
Done in 0.02s.
```