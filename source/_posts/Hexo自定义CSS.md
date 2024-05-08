---
title: Hexo自定义CSS
date: 2024-05-07 09:35:47
categories:
- Blog
tags:
- Blog
- Hexo
- CSS
---

# Hexo自定义CSS

## 创建CSS文件

首先创建`[Blogroot]/source/_data/style.css`文件

<!--more-->

在这个文件里编写自定义的CSS样式

```css [Blogroot]/source/_data/style.css
.blockquote {
    text-align: left; /* 将引用块中的文本左对齐 */
    margin-left: 0; /* 去除引用块的左侧间距 */
}
```

## 主题设置

### clean-blog主题

进入主题`layout`目录，在主题的布局文件中`<head>`标签

clean-blog主题的`<head>`标签在`layout/head.ejs`中，且有注释说明自定义的位置，按照说明操作，如果主题中没有注释，则在`<head>`标签内添加下面的内容

```ejs [Blogroot]/themes/clean-blog/layout/_partial/head.ejs
    <!-- Custom CSS -->
    <%- css('css/style.css') %>
```

### NexT主题

在主题配置文件中取消注释

```yml [Blogroot]/_config.next.yml
custom_file_path:
  #head: source/_data/head.njk
  #header: source/_data/header.njk
  #sidebar: source/_data/sidebar.njk
  #postMeta: source/_data/post-meta.njk
  #postBodyStart: source/_data/post-body-start.njk
  #postBodyEnd: source/_data/post-body-end.njk
  footer: source/_data/footer.njk
  #bodyEnd: source/_data/body-end.njk
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  style: source/_data/styles.styl
```

设定完成后，重新生成博客即可

```sh
$ hexo generate
$ hexo deploy
```
