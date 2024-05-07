---
title: 基于NexT主题的Hexo博客美化
tags:
---

## 主题安装

通过`git`安装主题

```sh
$ cd [Blogroot]
$ git clone https://github.com/next-theme/hexo-theme-next themes/next
```
<!--more-->

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

头像

```yml [Blogroot]/_config.next.yml
avatar:
  # Replace the default image and set the url here.
  url: https://static.wixstatic.com/media/7ac599_2e792bfa812140c898f6c3a78e4ab78f~mv2.png/v1/fill/w_951,h_1046,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/7ac599_2e792bfa812140c898f6c3a78e4ab78f~mv2.png #/images/avatar.gif
  # If true, the avatar will be displayed in circle.
  rounded: true
  # If true, the avatar will be rotated with the cursor.
  rotated: true
```

社交媒体

```yml [Blogroot]/_config.next.yml
social:
  GitHub: https://github.com/Jccc-l || fab fa-github
  E-Mail: mailto:1216550215Jc@gmail.com || fa fa-envelope
```

## 主题修改美化

NexT主题支持黑暗~~黑化~~模式

```yml [Blogroot]/_config.next.yml
darkmode: true
```

设置头像

```yml [Blogroot]/_config.next.yml
avatar:
  url: https://static.wixstatic.com/media/7ac599_2e792bfa812140c898f6c3a78e4ab78f~mv2.png/v1/fill/w_951,h_1046,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/7ac599_2e792bfa812140c898f6c3a78e4ab78f~mv2.png
```

设置社交媒体

```yml [Blogroot]/_config.next.yml
social:
  GitHub: https://github.com/yourname || fab fa-github
  #E-Mail: mailto:yourname@gmail.com || fa fa-envelope
```

## 布局调整

### 调整内容宽度

Gemini主题默认的文章宽度太宽了

打开`[Blogroot]/themes/next/source/css/_variables/Pisces.styl`文件，调整三个变量

```styl [Blogroot]/themes/next/source/css/_variables/Pisces.styl
$content-desktop              = 'calc(100% - %s)' % unit($content-desktop-padding / 2, 'px');
$content-desktop-large        = 1160px;
$content-desktop-largest      = 45%;
```

这个文件还能调整标题、侧栏等的布局

## 页面设置

修改主题配置文件，增加`标签`、`分类`、`归档`等页面

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

编辑添加的页面

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

## 侧栏内容设置

