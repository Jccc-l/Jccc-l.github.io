---
title: 基于NexT主题的Hexo博客美化
date: 2024-05-08 23:22:01
description: 记录如何基于NexT主题对Hexo博客进行美化
mathjax: false
katex: true
categories:
  - Blog
tags:
- Blog
- Hexo
- NexT
---


## 主题安装

通过`git`安装主题

```sh
$ cd [Blogroot]
$ git clone https://github.com/next-theme/hexo-theme-next themes/next
```

修改博客主配置文件

```yml [Blogroot]/_config.yml
theme: next
```

备份NexT主题的配置文件

```sh
$ cp [Blogroot]/theme/next/_config.yml [Blogroot]/_config.next.yml
$ mv [Blogroot]/theme/next/_config.yml [Blogroot]/theme/next/_config.yml.template
```

安装完成，重新生成博客就能直接用了

当然NexT提供了四种主题，可以在`_config.next.yml`文件中进行设置

```yml [Blogroot]/_config.next.yml
scheme: Gemini
```

## 主题修改美化和功能设置

### 黑暗模式

NexT主题支持黑暗~~黑化~~模式

```yml [Blogroot]/_config.next.yml
darkmode: true
```

### 网页信息设置

#### 主页头像

将头像文件放在`[Blogroot]/theme/next/source/images/`中，然后引用文件URL

或者可以使用网络图片的链接

```yml [Blogroot]/_config.next.yml
custom_logo: /images/avatar.jpeg
```

#### 许可信息

可以在文章末或侧栏中设置许可信息

```yml [Blogroot]/_config.next.yml
# Creative Commons 4.0 International License.
# See: https://creativecommons.org/about/cclicenses/
creative_commons:
  # Available values: by | by-nc | by-nc-nd | by-nc-sa | by-nd | by-sa | cc-zero
  license: by-nc-sa
  # Available values: big | small
  size: small
  sidebar: false
  post: true
  # You can set a language value if you prefer a translated version of CC license, e.g. deed.zh
  # CC licenses are available in 39 languages, you can find the specific and correct abbreviation you need on https://creativecommons.org
  language:
```

## 布局调整

### 调整内容宽度

Gemini主题默认的文章宽度太宽了

打开`[Blogroot]/themes/next/source/css/_variables/Pisces.styl`文件，调整三个变量

```styl [Blogroot]/themes/next/source/css/_variables/Pisces.styl
$content-desktop              = 'calc(100% - %s)' % unit($content-desktop-padding / 2, 'px');
$content-desktop-large        = 1000px;
$content-desktop-largest      = 60%;
```

这个文件还能调整标题、侧栏等的布局

## 菜单设置

修改主题配置文件，增加`标签`、`分类`、`归档`等菜单，并为菜单各个选项添加对应的页面

```yml [Blogroot]/_config.next.yml
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
```

建立页面

```sh
$ hexo n page categories
$ hexo n page about
$ hexo n page tags
```

编辑添加的页面，主要是要设置好`type`，让主题能够找到这个页面

```md [Blogroot]/source/categories/index.md
---
title: categories
date: 2024-05-07 09:04:59
type: "categories"
---
```

```md [Blogroot]/source/tags/index.md
---
title: tags
date: 2024-05-07 08:53:57
type: "tags"
---
```

```md [Blogroot]/source/about/index.md
---
title: about
date: 2024-05-07 08:53:57
type: "about"
---
```

## 侧栏

### 侧栏设置

```yml [Blogroot]/_config.next.yml
# 设置侧边栏默认开启
sidebar:
  display: always
# 设置社交链接
social:
  GitHub: https://github.com/Jccc-l || fab fa-github
  # E-Mail: mailto:1216550215Jc@gmail.com || fa fa-envelope
```

### 社交媒体

```yml [Blogroot]/_config.next.yml
social:
  GitHub: https://github.com/yourname || fab fa-github
  #E-Mail: mailto:yourname@gmail.com || fa fa-envelope
```

### 侧栏头像

可以在配置文件中插入图片URL

```yml [Blogroot]/_config.next.yml
avatar:
  url: https://1234567.com/xxxxx.png
```

或者将头像文件放至`[Blogroot]/themes/next/source/images/`中，然后设置头像

```yml [Blogroot]/_config.next.yml
avatar:
  # 头像文件位置
  url: /images/avatar.jpeg
  # 圆形头像
  rounded: false
  # 鼠标放在头像处时旋转
  rotated: false
```

### 链接

### 文章目录

```yml [Blogroot]/_config.next.yml
toc:
  enable: true
  # 关闭序号
  number: false
  # 长标题换行
  wrap: false
  # 只在展开浏览中部分的目录
  expand_all: false
  # 目录的最大深度
  max_depth: 6
```

还可以在侧栏添加知识共享许可信息，在[网页信息](#许可信息)部分有提到

## 页脚

### 添加网站运行时间

修改NexT主题配置，自定义footer

```yml [Blogroot]/_config.next.yml
custom_file_path:
  footer: source/_data/footer.njk
  style: source/_data/styles.styl
```

编辑`footer.njk`文件

```njk [Blogroot]/source/_data/footer.njk
<div>
<span id="timeDate">载入天数...</span><span id="times">载入时分秒...</span>
<script>
    var now = new Date();
    function createtime() {
        var grt= new Date("05/04/2023 00:00:00");
        now.setTime(now.getTime()+250);
        days = (now - grt ) / 1000 / 60 / 60 / 24; dnum = Math.floor(days);
        hours = (now - grt ) / 1000 / 60 / 60 - (24 * dnum); hnum = Math.floor(hours);
        if(String(hnum).length ==1 ){hnum = "0" + hnum;} minutes = (now - grt ) / 1000 /60 - (24 * 60 * dnum) - (60 * hnum);
        mnum = Math.floor(minutes); if(String(mnum).length ==1 ){mnum = "0" + mnum;}
        seconds = (now - grt ) / 1000 - (24 * 60 * 60 * dnum) - (60 * 60 * hnum) - (60 * mnum);
        snum = Math.round(seconds); if(String(snum).length ==1 ){snum = "0" + snum;}
        document.getElementById("timeDate").innerHTML = "本站已安全运行 "+dnum+" 天 ";
        document.getElementById("times").innerHTML = hnum + " 小时 " + mnum + " 分 " + snum + " 秒";
    }
setInterval("createtime()",250);
</script>
</div>
```

时间改为网站的起始时间

### 添加版权信息

```yml [Blogroot]/_config.next.yml
footer:
  copyright: <span id="copyright"> 路过的即是风景. All rights reserved. Non-commercial use allowed under <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">CC BY-NC-SA 4.0</a>.</span>
```

## 浏览进度条

配置以下内容，可以在页面顶部或底部添加浏览进度条

```yml [Blogroot]/_config.next.yml
reading_progress:
  # 启用进度条
  enable: true
  # 进度条位置: top | bottom
  position: top
  # 进度条颜色
  color: "#37c6c0"
  # 进度条高度
  height: 3px
```

## 配置背景图

自定义styl文件，设置背景

```styl [Blogroot]/source/_data/styles.styl
/* Body背景设置 */
body {
    background: url(/images/background.jpg) no-repeat fixed;
    background-size: cover;
    background-position: center;
}
```

设置文章背板等的颜色，并设置半透明效果

```styl [Blogroot]/source/_data/styles.styl
.main-inner > .sub-menu,
.main-inner > .post-block,
.main-inner > .tabs-comment,
.main-inner > .comments,
.main-inner > .pagination {
    background: rgba(245, 245, 245, 0.8); /* 背景色透明度设置 */
}
```

将代码块、图片等部分透明度调为1，即不透明

```styl [Blogroot]/source/_data/styles.styl
img {
    opacity: 1; /* 设置图片不透明 */
}
.table-container {
    opacity: 1; /* 设置 table-container 不透明 */
}
```

设置侧边栏的透明度
```styl [Blogroot]/source/_data/styles.styl
.sidebar {
    opacity: 0.9;
}
```


## Reference

[【个人网站搭建】hexo框架Next主题下添加网站运行时间](https://blog.csdn.net/wangqingchuan92/article/details/126346205)
