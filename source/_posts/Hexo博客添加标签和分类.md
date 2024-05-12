---
title: Hexo博客添加标签和分类
date: 2024-05-07 14:18:52
description: 标签和分类是博客的一个重要功能，方便我们进行快速索引。 
categories:
- Blog
tags:
- Hexo
- Blog
- 标签
mathjax: false
katex: false
---

# Hexo博客添加标签和分类

## 配置博客标签

首先要创建一个标签页面，在博客根目录输入

```shell
$ hexo new page tags
```

在创建的文章中添加页面类型，如下：

```md [Blogroot]/source/tags/index.md
---
title: tags
date: 2024-05-07 08:53:57
type: "tags"
---
```

在博客中添加标签，比如这篇博客的标签：

```md [Blogroot]/source/_posts/搭建我的Hexo博客.md
---
title: 搭建我的Hexo博客
date: 2024-01-17 13:35:13
tags:
- Hexo
- Blog
---
```

部分主题需要取消注释开启tags页面，比如NexT主题：

```yml [Blogroot]/themes/next/_config.yml
menu:
  #home: / || fa fa-home
  #about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

## 博客分类

与博客标签类似，将`tags`改为`categories`即可

`categories/index.md`文件内容如下

```md [Blogroot]/source/categories/index.md
---
title: categories
date: 2024-05-07 09:04:59
type: "categories"
---
```

博客信息部分如下：

```md [Blogroot]/source/_posts/搭建我的Hexo博客.md
---
title: 搭建我的Hexo博客
date: 2024-01-17 13:35:13
tags:
- Hexo
- Blog
categories:
- Blog
---
```

## Reference

[Hexo使用攻略-添加分类及标签](https://www.jianshu.com/p/e17711e44e00)
