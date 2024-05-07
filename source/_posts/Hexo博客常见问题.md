---
title: Hexo博客常见问题
date: 2024-05-06 15:39:50
categories:
- Blog
tags:
- Hexo
- Blog
katex: true
mathjax: false
---

# Hexo博客常见问题

## 图片存储方案

在上一章中，我们配置了在创建博客同时创建一个同名文件夹存储图片等文件，但是当我们在博客的Markdown文件中引用图片文件，图片并不能显示出来

<!--more-->

<img src="Image_Reference_Failed.png" style="max-width:100%;">

### 官方方案

[官方](https://hexo.io/docs/asset-folders.html)给出了一些解决方案，这里我用一个最简单的方法

在`_config.yml`配置文件中配置以下内容

```yml [Blogroot]/_config.yml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

如果创建了一篇博客"Hello"，则会在`source/_posts`文件夹中同时创建一个同名的`Hello`文件夹，将`PicGo.png`图片文件放置于`Hello`文件夹中，在"Hello.md"使用Markdown格式引用图片文件即可


```md source/_posts/first_blog.md
---
title: first_blog
date: 2024-05-03 18:20:08
tags:
---

# first blog

![](PicGo.png)
```

可以看到图片成功显示出来

<img src="Image_Displayed.png" style="max-width:100%;">

> 在博客主页的图片似乎还是不能显示，只有打开具体某篇博客的页面才能显示出来。。。

### 图床

另一个方案可以使用图床，我在另一篇博客中有教怎么用Github搭建图床

## 模板

在执行`hexo new`命令时，Hexo会在对应的目录根据模板文件生成一个新的文件，可以对模板进行自定义

```md scaffolds/post.md
---
title: {{ title }}
date: {{ date }}
categories: 
- 
tags: 
- 
```

## Hexo的Katex支持

安装插件[hexo-math](https://github.com/hexojs/hexo-math)，通过标签插件支持[KaTeX](https://katex.org/)和[MathJax](https://www.mathjax.org/)

在文章信息部分启用katex和mathjax选项

```md
---
title: Hexo博客常见问题
date: 2024-05-06 15:39:50
categories:
- Blog
tags:
- Hexo
- Blog
katex: true
mathjax: false
---
```

然后就可以数学公式就可以用CSS格式进行渲染了，像这样

```css
{% katex %}
c = \pm\sqrt{a^2 + b^2}
{% endkatex %}
```

{% katex %}
c = \pm\sqrt{a^2 + b^2}
{% endkatex %}
