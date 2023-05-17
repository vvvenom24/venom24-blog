---
title: "个性化 Hugo 主题「MemE」"
date: 2023-05-17T16:40:12+08:00
draft: false
series: [Blog]
tags: [Blog,Hugo,MemE,Twikoo,Memos]
summary: "个性化「MemE」的字体和评论，并引入 Memos"
---
## 个性化原则

由于博客主题是通过 Git 子模块的方式引入的，为了避免后面主题的更新不会影响到个性化内容，所以不建议改动「themes」下的内容

我们在博客的根目录下，通过**相同文件路径以及文件名**的方式覆盖主题文件即可生效

## 自定义字体

文章主体，使用 [「霞鹜文楷」](https://github.com/chawyehsu/lxgw-wenkai-webfont) 

修改「config.toml」

```toml
# 文章标题、文章副标题、列表标题、列表的年份和月份标题、相关文章标题、文章上下篇标题、表格的表头、定义列表中的术语
fontFamilyTitle = "'LXGW WenKai Screen', sans-serif"
# 分节标题、目录标题
fontFamilyHeadings = "'LXGW WenKai Screen', sans-serif"
# 主体
fontFamilyBody = "'LXGW WenKai Screen', sans-serif"
```

字体链接使用 [@一蓑烟雨](https://easyf12.top/) 分享的 staticfile CDN

```toml
# 网络字体链接
fontsLink = "https://cdn.staticfile.org/lxgw-wenkai-screen-webfont/1.6.0/lxgwwenkaiscreen.css"
```

文章代码部分，使用 [JetBrains Mono](https://www.jetbrains.com/lp/mono/)

代码的字体直接下载放到项目「static/fonts」下

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517170958.png)

然后项目下新建「assets/scss/custom」文件夹

新增文件「_custom.scc」

```scss
// 字体设定
@import "font";
```

新增文件「_font.scss」

```scss
@font-face {
    font-family: 'JetBrains Mono';
    font-display: swap;
    src: url('#{$baseRelURL}/fonts/JetBrainsMono-Regular.woff2') format('woff2');
    font-weight: 400;
    font-style: normal;
}
@font-face {
    font-family: 'JetBrains Mono';
    font-display: swap;
    src: url('#{$baseRelURL}/fonts/JetBrainsMono-Bold.woff2') format('woff2');
    font-weight: 700;
    font-style: normal;
}
@font-face {
    font-family: 'JetBrains Mono';
    font-display: swap;
    src: url('#{$baseRelURL}/fonts/JetBrainsMono-Italic.woff2') format('woff2');
    font-weight: 400;
    font-style: italic;
}

:root {
    --text-wdth: 90;
    --text-opsz: 40;
    --text-YTLC: 460;
}

body {
    font-variation-settings:
        'wdth' var(--text-wdth),
        'opsz' var(--text-opsz),
        'YTLC' var(--text-YTLC);
}

.post-title {
    font-variation-settings:
        'wght' 550,
        'opsz' 60,
        'YOPQ' 90;
}

.list-item-time {
    font-feature-settings: 'tnum';
}

//自适应屏幕宽度

@media (max-width: 500px) {
    body {
        font-size: $fontSize * 0.875;
    }
    .drop-cap {
        font-size: $fontSize * 0.875 * 3;
        margin-right: $lineHeight * $fontSize * 0.875 - $fontSize * 0.875;
        margin-top: $fontSize * 0.875 / $lineHeight;
        line-height: $lineHeight * $fontSize * 0.875;
    }
}
```

修改配置文件「config.toml」

```toml
# 代码、上标、文章元信息、文章更新徽章、文章的 Git 版本信息、极简页脚、不蒜子站点浏览计数
fontFamilyCode = "'JetBrains Mono', 'LXGW WenKai Screen', monospace"
```

## 引入 Twikoo 评论系统

[Twikoo](https://twikoo.js.org/) 一个简洁、安全、免费的静态网站评论系统

### 通过 Zeabur 部署 Twikoo

使用 GitHub 账号登录 [Zeabur](https://dash.zeabur.com/)[^1]

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517174756.png)

创建一个新的 Project，名字随意

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517175910.png)

点击「Add new service」，然后选择「Deploy Other Services」

![image.png](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517180405.png)

部署一个 Mongo 服务

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517180544.png)

到 GitHub 中将 [Twikoo-zeabur](https://github.com/imaegoo/twikoo-zeabur) fork 到自己的账号下

然后再到 Zeabur 中点击「Add new service」，然后选择「Deploy Your Source Code」将「Twikoo-zeabur」部署到 Zeabur

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517181158.png)

部署完成后，绑定域名

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517180939.png)

直接访问绑定的域名

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517181419.png)

这样就说明部署成功了

### MemE 中添加 Twikoo

到博客根目录下

新建「layouts/partials/components/comments.html」文件

在原地址文件的基础上增加

```html
{{ if .Site.Params.enableTwikoo }}
    <div id="tcomment"></div>
{{ end }}
```

完整如下

```html
{{ if and (.Params.comments | default .Site.Params.enableComments) (eq hugo.Environment "production") }}
    {{ if or (in .Site.Params.mainSections .Section) .Params.comments }}

        {{ if not .Site.Params.autoLoadComments }}
            <div class="load-comments">
                <div id="load-comments">{{ i18n "loadComments" }}</div>
            </div>
        {{  end }}

        {{ if .Site.Params.enableTwikoo }}
            <div id="tcomment"></div>
        {{ end }}

        {{ if .Site.Params.enableDisqus }}
            <div id="disqus_thread"></div>
        {{ end }}

        {{ if .Site.Params.enableValine }}
            <div id="vcomments"></div>
        {{ end }}

        {{ if .Site.Params.enableUtterances }}
            <div id="utterances"></div>
        {{ end }}

        {{ if .Site.Params.enableGitalk }}
            <div id="gitalk-container"></div>
        {{ end }}
    {{ end }}
{{ end }}
```

新建「layouts/partials/pages/third-party/script.html」文件

在原地址文件的基础上增加

```html
{{ if .Site.Params.enableTwikoo }}
    {{ partial "third-party/twikoo.html" . }}
{{ end }}
```

完整如下

```html
{{ if .Params.katex | default .Site.Params.enableKaTeX }}
    {{ partial "third-party/katex.html" . }}
{{ end }}

{{ if .Params.mathjax | default .Site.Params.enableMathJax }}
    {{ partial "third-party/mathjax.html" . }}
{{ end }}

{{ if .Params.mermaid | default .Site.Params.enableMermaid }}
    {{ partial "third-party/mermaid.html" . }}
{{ end }}

{{ if and (.Params.comments | default .Site.Params.enableComments) (eq hugo.Environment "production") }}
    {{ if or (in .Site.Params.mainSections .Section) .Params.comments }}

        {{ if .Site.Params.enableTwikoo }}
            {{ partial "third-party/twikoo.html" . }}
        {{ end }}

        {{ if .Site.Params.enableDisqus }}
            {{ partial "third-party/disqus.html" . }}
        {{ end }}

        {{ if .Site.Params.enableValine }}
            {{ partial "third-party/valine.html" . }}
        {{ end }}

        {{ if .Site.Params.enableUtterances }}
            {{ partial "third-party/utterances.html" . }}
        {{ end }}

        {{ if .Site.Params.enableGitalk }}
            {{ partial "third-party/gitalk.html" . }}
        {{ end }}

    {{ end }}
{{ end }}

{{ if .Site.Params.enableMediumZoom }}
    {{ partial "third-party/medium-zoom.html" . }}
{{ end }}

{{ if .Site.Params.enableInstantPage }}
    {{ partial "third-party/instant-page.html" . }}
{{ end }}

{{ partial "third-party/busuanzi.html" . }}

{{ partial "custom/script.html" . }}
```

新建「layouts/partials/pages/third-party/twikoo.html」文件

这是个新增文件，引入 Twikoo

```html
<script>
    function loadComments() {
        if (typeof Twikoo === 'undefined') {
            const getScript = (options) => {
                const script = document.createElement('script');
                script.defer = true;
                script.crossOrigin = 'anonymous';
                Object.keys(options).forEach((key) => {
                    script[key] = options[key];
                });
                document.body.appendChild(script);
            };
            getScript({
                src: 'https://cdn.jsdelivr.net/npm/twikoo@1.6.16/dist/twikoo.all.min.js',
                onload: () => {
                    newTwikoo();
                }
            });
        } else {
            newTwikoo();
        }
    }
    function newTwikoo() {
        twikoo.init({
            el: '#tcomment',
            envId: '{{ .Site.Params.twikooEnvId }}'
        });
    }
</script>
```

最后，到「comfig.toml」中添加对应配置

```toml
## Twikoo
enableTwikoo = true
twikooEnvId = ""
```

**注意**：开发环境下启动 Hugo 时，不会展示评论，需要在生产环境下启动：`hugo --environment production server`

## 引入 Memos

同样的，通过 Zeabur 部署 [Memos](https://usememos.com/)[^2]

在 Zeabur 中绑定完域名后，第一次进入需要注册一个账号，这个账号便是管理员

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230517183716.png)

### 主页滚动近期 Memos

MemE 的主页有「文章摘要」和「诗意人生」两种风格，这两种分别对应「layouts/partials/pages/home-posts.html」和「layouts/partials/pages/home-poetry.html」两个文件

我这里使用的是「文章摘要」的风格

复制「layouts/partials/pages/home-posts.html」到博客根目录

在「main」标签下的第一行增加

```html
<div id="memos" class="main-inner">
    {{ partial "pages/memos.html" . }}
</div>
```

同级目录下，新建文件「memos.html」

```html
<!--引入相对时间 Lately 插件-->
<!-- <script src="https://tokinx.github.io/lately/lately.min.js"></script> -->

<!--JS 处理 Memos API-->
<script>
    (() => {
        window.Lately = new function () {
            this.lang = {
                second: "秒",
                minute: "分钟",
                hour: "小时",
                day: "天",
                month: "个月",
                year: "年",
                ago: "前",
                error: "NaN"
            };
            const format = (date) => {
                date = new Date(date);
                const floor = (num, _lang) => Math.floor(num) + _lang,
                    obj = new function () {
                        this.second = (Date.now() - date.getTime()) / 1000;
                        this.minute = this.second / 60;
                        this.hour = this.minute / 60;
                        this.day = this.hour / 24;
                        this.month = this.day / 30;
                        this.year = this.month / 12
                    },
                    key = Object.keys(obj).reverse().find(_ => obj[_] >= 1);
                return (key ? floor(obj[key], this.lang[key]) : this.lang.error) + this.lang.ago;
            },
            _val = (date) => {
                // date = new Date(date && (typeof date === 'number' ? date : date.replace(/-/g, '/').replace('T', ' ')));
                // return isNaN(date.getTime()) ? false : date.getTime();
                if (date == 'undefined') {
                    return false;
                }
                if ((date + '').indexOf('T') > 0) {
                    return new Date(date).getTime();
                }
                if ((date + '').length < 13) {
                    return false;
                }
                return date;
            };
            return {
                init: ({ target = "time", lang } = {}) => {
                    if (lang) this.lang = lang;
                    for (let el of document.querySelectorAll(target)) {
                        const date = _val(el.dateTime) || _val(el.title) || _val(el.innerHTML) || 0;
                        if (!date) return;
                        el.title = new Date(+date).toLocaleString("zh-CN");
                        const i = el.innerHTML.indexOf(';');
                        if (i > 0) {
                            el.innerHTML = el.innerHTML.substring(0, i+1) + format(+date);
                        } else {
                            el.innerHTML = format(+date);
                        }
                    }
                },
                format
            }
        }
    })();

    let jsonUrl = "" + Date.parse(new Date());

    fetch(jsonUrl).then((res) => res.json()).then((resdata) => {
        var result = "",
            resultAll = "",
            data = resdata.data;
        for (var i = 0; i < data.length; i++) {
            var talkTime = data[i].createdTs * 1000;
            // var talkTime = timeConver(data[i].createdTs);
            var talkContent = data[i].content;
            var newtalkContent = talkContent.replace(/```([\s\S]*?)```[\s]*/g, " <code>$1</code> ") //全局匹配代码块
                .replace(/`([\s\S ]*?)`[\s]*/g, " <code>$1</code> ") //全局匹配内联代码块
                .replace(/\!\[[\s\S]*?\]\([\s\S]*?\)/g, "🌅") //全局匹配图片
                .replace(/\[[\s\S]*?\]\([\s\S]*?\)/g, "🔗") //全局匹配连接
                .replace(/\bhttps?:\/\/(?!\S+(?:jpe?g|png|bmp|gif|webp|jfif|gif))\S+/g, "🔗"); //全局匹配纯文本连接
            result += `<li class="item"><span class="memosTime" title=${talkTime}></span>： <a href="https://venom-memos.zeabur.app" target="_blank">${newtalkContent}</a></li>`;
        }
        var talkDom = document.querySelector("#memos");
        var talkBefore = `<img title="Memos" style="width: 1.7em; float: left; position: relative; top: 1px;" src="icons/tg.gif"></img><div class="talk-wrap"><ul class="talk-list">`;
        var talkAfter = `</ul></div>`;
        resultAll = talkBefore + result + talkAfter;
        talkDom.innerHTML = resultAll;

        // 相对时间
        window.Lately && Lately.init({ target: 'time, .memosTime' });
    });

    // 滚动效果
    setInterval(function () {
        var talkWrap = document.querySelector(".talk-list");
        var talkItem = talkWrap.querySelectorAll(".item");
        for (i = 0; i < talkItem.length; i++) {
            setTimeout(function () {
                talkWrap.appendChild(talkItem[0]);
            }, 2000);
        }
    }, 2000);

    // 时间戳转换
    function timeConver(t) {
        t = t * 1000;
        //年-月-日-时-分-秒
        var now = new Date(t);
        var year = now.getFullYear();
        var month = now.getMonth() + 1;
        var date = now.getDate();
        var hour = now.getHours();
        var minute = now.getMinutes();
        var second = now.getSeconds();
        // 补零
        if (month < 10) {
            month = "0" + month;
        }
        if (date < 10) {
            date = "0" + date;
        }
        if (hour < 10) {
            hour = "0" + hour;
        }
        if (minute < 10) {
            minute = "0" + minute;
        }
        if (second < 10) {
            second = "0" + second;
        }
        return (
            //按需求拼接   
            // year + "-" + month + "-" + date
            // year + "-" + month + "-" + date + " " + hour + ":" + minute + ":" + second
            year + "-" + month + "-" + date + " " + hour + ":" + minute
        );
    }
</script>
```

`jsonUrl` 替换为自己的 Memos API 地址

「_custom.scss」中增加

```scss
// Memos
@import 'memos';
```

同级目录下增加「_memos.scss」

```scss
#memos {
    display: block;
    margin: 0 auto;
    width: 36em;
}

.index-talk {
    display: flex;
    flex: 1 auto;
    width: 100%;
    text-align: left;
    position: relative;
}

.talk-list {
    padding-left: 0px;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
}

.talk-list li {
    list-style: none;
    margin-bottom: 10px;
    width: 100%;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
    display: inline;
    vertical-align: middle;
}

.talk-list li:not(:first-child) {
    display: none !important;
}

.talk-list li a:hover {
    text-decoration: none;
    color: #1890ff;
}
```

## 总结

这里的这些内容花了我一个星期左右的时间才弄好，实在是汗颜😓。为了这点效果，也是找遍了各个大佬的博客，东拼西凑地算是弄出来这些效果吧。虽然还是有些不完美，但也就这样了，不想再多折腾了，毕竟我只是个后端开发（虽然刚毕业也搞前端）。

上面的代码不乏有冗余，错误之处，但也不想顾此失彼多费精力了。

## 特别鸣谢

- [启用「霞鹜文楷」在线字体](https://immmmm.com/lxgw-wenkai-webfont/)

- [Waline 评论系统的介绍与基础配置](https://guanqr.com/tech/website/introduction-and-basic-setting-of-waline/)

- [一种在 MemE 主题中实现轮播图功能的思路](https://guanqr.com/tech/website/a-way-to-realize-carousel-in-meme/)

- [Memos 简介](https://eallion.com/memos-deployment/)

- [Memos API 公告样式滚动效果](https://eallion.com/memos-ticker/)



[^1]: Zeabur 属于大学生创业产品，团队成员主要来自浙江大学，目前仍处于推广阶段，服务免费使用，后续则将采用免费额度 + 按量计费的模式

[^2]: Memos 一个开源且免费的自托管知识库