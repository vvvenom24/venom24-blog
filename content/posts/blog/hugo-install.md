---
title: "使用 Hugo 创建个人博客"
date: 2023-05-16T19:29:13+08:00
lastmod: 2023-05-16T19:29:13+08:00
draft: false
series: [Blog]
tags: [Blog,Hugo]
summary: "Ubuntu 与 Windows 环境下安装配置 Hugo 并新建一个个人博客"
---
## Ubuntu 下安装配置 Hugo

> 按照 [hugo 官方文档](https://gohugo.io/installation/linux/)，本以为就是一行命令的事，但......

到 GitHub 下载 [最新版本 v0.111.3](https://github.com/gohugoio/hugo/releases/tag/v0.111.3) 的安装包

![deb](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517113717.png)

**注意**：这里有两个版本的 Hugo，标准版和扩展版[^1]。如果使用的主题是基于拓展版的，那这里就需要下载拓展版的安装包。

- 查找是否安装了其他版本
    ```shell
    dpkg --list |grep hugo
    ```

- 如果有，则卸载其他版本
    ```shell
    apt-get --purge remove hugo
    ```

- 下载
    ```shell
    wget https://github.com/gohugoio/hugo/releases/download/v0.111.3/hugo_extended_0.111.3_linux-amd64.deb
    ```

- 安装
    ```shell
    dpkg -i hugo*.deb
    ```

- 添加环境变量
    ```shell
    export HUGO=/usr/local/bin/hugo
    ```

- 如果上面环境变量的地址不对，可以查找
    ```shell
    find / -iname "hugo"
    ```

## Windows 下安装与配置

基本步骤差不多，不同的是 Windows 下是可视化界面操作而已

还是到 GitHub 下载 [最新版本 v0.111.3](https://github.com/gohugoio/hugo/releases/tag/v0.111.3) 的安装包

下载完后解压

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517123040.png)

配置环境变量

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517123341.png)

**注意**：使用「cmd」时，环境变量会立马生效，但是其他方式，比如「PowerShell」，则需要重启

## 初始化站点

- 初始化一个「hello-hugo」的博客

    ```shell
    hugo new site hello-hugo
    ```

    初始化完成后，会生成一个「hello-hugo」的文件夹

- 进入博客文件夹内

    ```shell
    cd hello-hugo
    ```

- 初始化 Git 仓库

    ```shell
    git init
    ```

## 安装博客主题

以 Git 子模块的方式，安装一个自己喜欢的主题

这里以「MemE」为例

```shell
git submodule add --depth 1 https://github.com/reuixiy/hugo-theme-meme.git themes/meme
```

修改博客配置文件[^2]，配置博客相关属性及功能

这里直接复制「MemE」主题中的配置模板，覆盖初始化的配置

```shell
rm config.toml && cp themes/meme/config-examples/en/config.toml config.toml
```

修改「config.toml」，自定义个性化配置[^3]即可

## 启动 Hugo

简单本地启动

```shell
hugo server
```

加载草稿

```shell
hugo server -D
```

指定 IP 启动[^4]

```shell
hugo server --bind "0.0.0.0" -D
```

指定启动环境[^5]

```shell
hugo --environment production server
```

## 默认模板

`archetypes` 下是每篇文章的一个 Front Matter 字段的模板[^6]

```markdown
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
series: []
tags: []
summary: ""
---
```

## 写一篇文章

```shell
hugo new post/mypost/hello-hugo.md
```

会在「content」目录下新建「post」目录和「mypost」目录以及一个「hello-hugo.md」文档

## 部署

虽然我有一台云服务器，但是由于国内的域名和服务器都需要备案，万一哪天我说了不该说的，被请去🍵，所以就不打算部到国内服务器上了

这里选择使用 ~~国内访问速度还可以~~[^7] 的 Cloudflare Pages

- 进入 [Cloudflare 控制台](https://dash.cloudflare.com/) 打开「Pages」菜单，创建项目，连接到Git

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517160950.png)

- 使用 GitHub 账号授权后，选择对应的库，「开始设置」

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517161509.png)

- 指定 Hugo 版本

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517161652.png)

- 「开始部署」

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517162527.png)

- 「自定义域」中配置自己的域名

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517162721.png)

这样就部署成功了，后面没吃推送到 GitHub 后，Cloudflare 都会自动构建部署

## 特别鸣谢

- [我是如何建立自己的个人博客的？](https://dejavu.moe/posts/how-i-built-my-personal-blog/)



[^1]: 拓展版可以编码 WebP 图像，以及使用嵌入式 LibSass 转译器将 Sass 转译为 CSS

[^2]: 一般 Hugo 的配置文件是「config.toml」，但也有使用「config.yml」，初始化博客使用 `hugo new site <Blog> -f yml` 可指定配置文件为「config.yml」

[^3]: 具体可参考 [本站配置文件](https://github.com/vvvenom24/venom24-blog/blob/master/config.toml)

[^4]: Hugo 默认使用「localhost」启动，意味着只能在启动 Hugo 的这台机器上访问，如果你是部署在云服务器上的话，你本地将无法访问，所以需要指定 IP 访问

[^5]: Hugo 默认的启动环境是「development」开发环境，开发环境下许多配置不会生效，例如：评论，如果你希望看到相关效果的话，需要指定生产环境启动

[^6]: [「Front-matter」是 markdown 文档最上方以「---」分隔的区域，用于指定当前文章的参数](https://zsyyblog.com/7b2db7c2.html)

[^7]: [Cloudflare发现来自中国来源的流量下降，但发现这不是由于Cloudflare服务的任何问题](https://www.cloudflarestatus.com/incidents/7gd0nkh3tqd7)