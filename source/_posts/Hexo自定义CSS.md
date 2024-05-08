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

创建`[Blogroot]/style.css`文件，在里面编写就好了

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

创建文件`[Blogroot]/source/_data/styles.styl`等文件，编辑文件

```styl [Blogroot]/source/_data/styles.styl
/* Body背景设置 */
body {
    background: url(/images/background.jpg) no-repeat fixed;
    background-size: cover;
    background-position: center;
}

/* 文章背板颜色设置 */
.main-inner > .sub-menu,
.main-inner > .post-block,
.main-inner > .tabs-comment,
.main-inner > .comments,
.main-inner > .pagination {
    background: rgba(245, 245, 245, 0.8); /* 背景色透明度设置 */
}

/* 文字颜色设置 */
body {
    color: #000; /* 主体字体颜色为纯黑 */
}

.posts-expand .post-title-link {
    color: #222; /* 首页文章标题颜色 */
}

.posts-expand .post-meta-container {
    color: #62a7b6; /* 文章日期颜色 */
}

/* 侧边框透明度设置 */
.sidebar {
    opacity: 0.9;
}

/* 菜单栏颜色设置 */
.header-inner {
    background: rgba(255, 255, 255, 0.7); /* 菜单栏背景色透明度设置 */
}

/* 搜索框透明度设置 */
.popup {
    opacity: 0.5;
}

/* 主体背景设置 */
.main-inner {
    background-color: rgba(255, 255, 255, 0); /* 主体背景透明 */
    padding: 0 40px; /* 调整组件位置 */
}

/* 底部字体颜色设置 */
.footer-inner {
    color: #555555;
}

/* 图片和table-container不透明设置 */
img {
    opacity: 1; /* 设置图片不透明 */
}

.table-container {
    opacity: 1; /* 设置 table-container 不透明 */
}

```

设定完成后，重新生成博客即可

```sh
$ hexo generate
$ hexo deploy
```
