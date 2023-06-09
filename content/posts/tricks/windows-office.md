---
title: "安装正版 Office"
date: 2023-06-09T16:40:50+08:00
lastmod: 2023-06-09T16:40:50+08:00
draft: false
series: [Tricks]
tags: [Windows,Office]
summary: "免费安装使用正版 Office"
---

> 微软官方其实是给出个人免费使用 Office 的方法 —— 使用官方批量授权的 Office 套件（批量授权的才有免费的密钥），并且微软也提供了教程。这样做其实是打击盗版，因为大家私下使用盗版一般查不到。企业用户根本不能使用这个批量授权，它是会抓的。简单讲就是，惩罚企业行为，允许个人可以名正言顺的非法使用，培养用户群体，这样就能一直占有市场，保持市场活力。不推荐大家使用第三方 KMS[^1] 工具激活，因为目前网上第三方的 KMS 激活工具很多都带有后门，不建议使用。

## 下载并安装部署工具

下载 [Office Deployment Tool](https://www.microsoft.com/en-us/download/details.aspx?id=49117)

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230609165626.png)

安装并指定安装目录

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/2023-06-09_16-59-29.png)

部署工具安装完成

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/2023-06-09_17-05-57.png)

## 配置 Office

访问 [Office 自定义工具](https://config.office.com/deploymentsettings)

选择正确的版本

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230609170934.png)

导出配置

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230609171041.png)

将配置文件拷贝到前面部署工具的目录下

## 安装 Office

到部署工具的目录下，打开 CMD 命令窗口

执行：

```cmd
setup /download <前面导出的配置文件>
```

上面下载完成后，执行：

```cmd
setup /configure <前面导出的配置文件>
```

## 激活

到 Office 的默认安装目录

- 64 位

    `C:\Program Files\Microsoft Office\Office16`

- 32 位

    `C:\Program Files (x86)\Microsoft Office\Office16`

执行命令：

```cmd
cscript ospp.vbs /sethst:kms.03k.org

cscript ospp.vbs /act
```

## 特别鸣谢

[Word、Excel、Pointpot 最强免费安装教程！](https://zhuanlan.zhihu.com/p/601894291)



[^1]: KMS 是一种专为中型和大型企业设计的 Microsoft 产品的激活方法。 在标准 SOHO 环境中，您在安装期间输入产品密钥，然后通过 Internet 激活产品。这是通过向 microsoft.com 上的服务器发送请求来完成的，然后该服务器授予或拒绝激活。通过输入称为通用批量许可证密钥（GVLK）的特殊密钥（又名「KMS 客户端密钥」），产品不再要求 Microsoft 服务器进行激活，而是通常驻留在公司内部网中的用户定义服务器（称为 KMS 服务器）。 Microsoft 仅将其 KMS 服务器提供给签署了所谓「选择合同」的公司。