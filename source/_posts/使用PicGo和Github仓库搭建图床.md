---
title: 使用PicGo和Github仓库搭建图床
date: 2024-05-06 16:01:56
description: 记录一下搭建图床（非原创）
categories:
- Blog
tags:
- Blog
- 图床
- Github
- PicGo
mathjax: false
katex: false
---

# 搭建图床

图床是储存图片的服务器，允许把外部网站链接到图片，方便我们编写博客和网站

## 安装PicGO

从[PicGo的仓库](https://github.com/Molunerfinn/PicGo/releases/)中下载PicGo的AppImage

赋予执行权限

```sh
chmod +x PicGo-2.3.1.AppImage
```

## 创建并配置Github仓库

创建一个公开的Github仓库用于存储我们的图片

在终端输入PicGo-2.3.1.AppImage的路径，启动PicGo，就会看到它的图标

<img src="https://pic.molunerfinn.com/picgo/docs/logo-150.png" style="max-width:100%">

右键图标打开主窗口

图床设置选择Github，设置好仓库名

将分支名从master改为main

生成token

<table>
    <tr>
        <td ><center><img src="Generate_token_1.png" style="max-width:100%">
        <td ><center><img src="Generate_token_2.png" style="max-width:100%">
    </tr>
</table>
<table>
    <tr>
        <td ><center><img src="Generate_token_3.png" style="max-width:100%">
        <td ><center><img src="Generate_token_4.png" style="max-width:100%">
    </tr>
</table>

选择期限和权限

<img src="Generate_token_5.png" style="max-width:100%">

复制这个token，粘贴到PicGo的`设定Token`中

<img src="Copy_Token.png" style="max-width:100%">

PicGo的设置如图所示，将Github设置为默认图床

<img src="PicGo_Settings.png" style="max-width:100%">

## 配置CDN加速

使用[JSDELIVR](https://www.jsdelivr.com/github)进行CND配置

只需要在`设定自定义域名`这一项中添加网址，后面跟着你的仓库链接。

```
https://cdn.jsdelivr.net/gh/<username>/<repo>
```

我的仓库是`Jccc-l/Image-Bed`，就写成

```
https://cdn.jsdelivr.net/gh/Jccc-l/Image-Bed
```

## Reference

- [通过GitHub+PicGo+CDN搭建自己的图床](https://www.bilibili.com/video/BV1Ua4y1D7Li/?spm_id_from=333.337.search-card.all.click)
