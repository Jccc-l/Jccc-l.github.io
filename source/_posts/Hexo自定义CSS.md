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

首先在博客目录创建`source/css/style.css`

<!--more-->

在这个文件里编写自定义的CSS样式

```css blog/source/css/style.css
.blockquote {
    text-align: left; /* 将引用块中的文本左对齐 */
    margin-left: 0; /* 去除引用块的左侧间距 */
}
```

进入主题`layout`目录，在主题的布局文件中`<head>`标签

clean-blog主题的`<head>`标签在`layout/head.ejs`中，且有注释说明自定义的位置，按照说明操作，如果主题中没有注释，则在`<head>`标签内添加下面的内容

```ejs blog/themes/clean-blog/layout/_partial/head.ejs
    <!-- Custom CSS -->
    <%- css('css/style.css') %>
```

设定完成后，重新生成博客即可

```sh
$ hexo generate
$ hexo deploy
```
