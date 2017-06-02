---
author: Blanboom
layout: post
slug: hack-hexo-theme-landscape
title: "Hexo 主题 Landscape 改造小记"
date: 2015-02-07 18:31:00 +0800
comments: true
categories: 软件与工具
tags:
- Hexo
- CSS
- HTML
- 主题
---

昨晚更新 Hexo 后，发现新版 Hexo 自带的主题 [Landscape](https://github.com/hexojs/hexo-theme-landscape) 不错，就决定使用这一款主题。同时，对 Landscape 中一些不太符合自己使用习惯的地方进行了修改。

#### 目录：

1. [自动切换 banner 图片](#toc_0)
2. [更改引文样式](#toc_1)
3. [更改标题样式](#toc_2)
4. [优化字体方案](#toc_3)
5. [修改分享按钮](#toc_4)
6. [替换 Google Javascript 库和字体库](#toc_5)

<!-- more -->

<h1 id="toc_0">1. 自动切换 banner 图片</h1>

更换主题后，打开网页，首先看到的是顶部大大的 banner 图片。不过，默认的 banner，只能固定显示一张图片。如果这张图片能替换成自己作品的照片，并且能自动随机切换，效果应该会更好。

上网搜索了一下，刚好有人有同样的想法。使用 Javascript 即可实现：

``` javascript
<script>
    <% if (theme.switch_banner){ %>
    var number_of_banners = 6;
    var randnum = Math.floor(Math.random() * number_of_banners + 1);
    document.getElementById("banner").style.backgroundImage = "url(/css/images/banner" + randnum + ".jpg)";
    <% } else { %>
    document.getElementById("banner").style.backgroundImage = "url(/css/images/banner.jpg)";
    <% } %>
</script>
```

详细的操作步骤请参考：[自动随机切换Hexo博客的banner图片](http://kuangqi.me/tricks/hexo-banner-auto-switcher/)

<h1 id="toc_1">2. 更改引文样式</h1>

Hexo 默认的引文样式为大字号居中显示，但是对我来说，`blockquote` 被更多地用来展示内容的层级关系。所以，我对 `blockquote` 的样式进行了简单的修改：

``` css
font-family: font-sans
border-left: 5px solid #DDD
margin: 15px 0 0 2px
padding-left: 20px
```

效果如图所示：

![引文样式](/images/2015/02/hexo_theme_blockquote.png)

<h1 id="toc_2">3. 更改标题样式</h1>

在一级标题下加上浅色背景，二级标题下加浅色下划线，能使文章层次更加清晰，便于阅读。可在 `article.styl` 的 `.article-entry` 合适位置添加如下内容：

``` css
 h1
   padding: 7px 6px 7px 0px;
   background: #dddddd;
 h2
   border-bottom: 1px #d8d8d8 solid;
```

显示效果如下：

![标题样式](/images/2015/02/hexo_theme_title.png)

#### 参考资料：

* [关于可能吧排版的一些分享](http://www.windson.in/?p=573)
* [再谈博客文章排版](http://wangyueblog.com/2010/08/27/blog-post-layout/)

<h1 id="toc_3">4. 优化字体方案</h1>

为了是网页在各个操作系统中都能显示出高质量的中英文字体，对默认的字体方案稍作修改：

``` css
font-sans = "Helvetica Neue", "Helvetica", "Hiragino Sans GB", "Microsoft YaHei", "Source Han Sans CN", "WenQuanYi Micro Hei", Arial, sans-serif
```

通过上述代码，可以在 OS X 操作系统下默认显示 Hiragino Sans GB （冬青黑体简体中文），Windows 操作系统下默认显示微软雅黑，Linux 操作系统下默认显示思源黑体或文泉驿微米黑。但在 iOS 等系统里，默认只能以 黑体-简 显示中文，显示效果不太理想（尤其是显示粗体文本时）。

#### 参考资料:

* [如何保证网页的字体在各平台都尽量显示为最高质量的黑体？](http://www.zhihu.com/question/19911793)
* [谈谈网页设计中的字体设定](http://ptbsare.org/2014/09/24/谈谈网页设计中的字体设定/)

<h1 id="toc_4">5. 修改分享按钮</h1>

Landscape 主题提供了分享功能，可以将文章分享到 Facebook，Twitter 等网站。但在国内，这些网站用户不多，所以我将其替换成了新浪微博等国内社交网站。

在 `source/js/script.js` 中，可以找到 `'<div class="article-share-links">',`，下面的四个链接就是 Facebook 等社交网站的分享链接。将其替换成如下代码，即可实现分享到国内社交网站：

``` html
'<a href="http://service.weibo.com/share/share.php?&title=' + encodedUrl + '" class="article-share-sina" target="_blank" title="微博"></a>',
'<a href="http://share.renren.com/share/buttonshare.do?link=' + encodedUrl + '" class="article-share-renren" target="_blank" title="人人"></a>',
'<a href="http://sns.qzone.qq.com/cgi-bin/qzshare/cgi_qzshare_onekey?url=' + encodedUrl + '" class="article-share-qq" target="_blank" title="QQ 空间"></a>',
'<a href="http://v.t.qq.com/share/share.php?url=' + encodedUrl + '" class="article-share-tencent" target="_blank" title="腾讯微博"></a>',
```

同时，还需要替换四个网站的图标。本主题使用 Font Awesome 来显示图标，但内置的 Fone Awesome 版本较旧，无法显示 QQ、腾讯微博等图标，所以，需要[下载最新版 Font Awesome](http://fortawesome.github.io/Font-Awesome/)，替换掉 `source/fonts` 中相关文件，并在 `source/css/_variables.styl` 中的 `font-icon-version` 修改为最新的 Font Awesome 版本号。

然后，在 `source/css/_partial/article.styl` 中，找到四段以 `.article-share-***` 开头的代码，替换为如下内容：

``` css
.article-share-sina
  @extend $article-share-link
  &:before
    content: "\f18a"
  &:hover
    background: color-sina
    text-shadow: 0 1px darken(color-sina, 20%)

.article-share-qq
  @extend $article-share-link
  &:before
    content: "\f1d6"
  &:hover
    background: color-qq
    text-shadow: 0 1px darken(color-qq, 20%)

.article-share-renren
  @extend $article-share-link
  &:before
    content: "\f18b"
  &:hover
    background: color-renren
    text-shadow: 0 1px darken(color-renren, 20%)

.article-share-tencent
  @extend $article-share-link
  &:before
    content: "\f1d5"
  &:hover
    background: color-tencent
    text-shadow: 0 1px darken(color-tencent, 20%)
```

最后，找到 `source/css/_variables.styl` 中 `Colors` 部分，最后四行分别为四个社交网站图标的背景色，可根据这些网站的主题色修改。这是我的修改结果：

```css
color-sina = #ea0020
color-qq = #518adb
color-renren = #406ccb
color-tencent = #33b5eb
```

修改后的效果如下：

![分享按钮](/images/2015/02/hexo_theme_share.png)

#### 参考资料：

* [Font Awesome Cheatsheet](http://fortawesome.github.io/Font-Awesome/cheatsheet/)

<h1 id="toc_5">6. 替换 Google Javascript 库和字体库</h1>

本主题使用 Google 提供的 JS 库和字体，但 Google API 在国内基本无法访问，导致网站部分功能失效。而新浪、百度、微软、360 均提供类似的服务，可以在在国内正常使用。

找到 `layout/_partial/after-footer.ejs` 和 `layout/_partial/head.ejs` 两个文件，将其中的 `googleapis` 替换为 `useso` 即可。

#### 参考资料:

* [国内有哪些靠谱的 Javascript 库 CDN 可用？](http://www.zhihu.com/question/20227463)
