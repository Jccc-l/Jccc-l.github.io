---
title: 将Hexo备份到Github仓库
date: 2024-05-07 10:55:22
categories:
- Blog
tags:
- Hexo
- Blog
- Github
---

# 将Hexo博客备份到Github仓库

在另一篇文章中，我示范了如何利用[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)插件将博客推送到GitHub。使用这个插件，我们能够将已编译的博客文件推送到名为`<username>.github.io`的Github仓库。  
在这个仓库中，只有public目录下的文件会被上传，而其他的源文件，比如source、scaffolds，以及node模块等，则不会被包含在上传的内容中。  
<img src="未备份1.png" style="max_width:100%">
<img src="未备份2.png" style="max_width:100%">
因此我们需要通过git创建分支，将我们的源文件进行备份。

## 创建分支

首先克隆我们的`<username>.github.io`仓库

```sh
$ git clone https://github.com/<username>/<username>.github.io
$ cd <username>.github.io
```

创建一个新分支`hexo`并切换到这个分支

```sh
$ git checkout -b hexo
```

然后将.git整个文件夹移动到你的博客根目录

```sh
$ mv .git ~/Blog/
```

## 配置gitignore

一般需要忽略一些无关文件，这些文件不需要推送到仓库中，配置`.gitignore`文件

```.gitignore .gitignore
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

提交`.gitignore`文件

```sh
$ git add .gitignore
$ git commit -m "Add .gitignore"
```

## 开始备份

> 如果对主题进行了修改，要删除主题目录下的`.git`文件夹，比如`Blog/theme/next/.git`

依次执行添加暂存区、提交本地库、推送至Github

```sh
git add .       # 添加暂存区
git commit -m "Hexo source post"
git push origin hexo
```

## 恢复博客

那么如果本地博客没了，要怎么恢复呢？

只需要将Github仓库里的`hexo`分支（即备份分支）克隆下来

Reference
---
[【Hexo】Hexo博客备份到Github](https://ayshansu.github.io/%E3%80%90Hexo%E3%80%91Hexo%E5%8D%9A%E5%AE%A2%E5%A4%87%E4%BB%BD%E5%88%B0Github/)
