---
title: Linux下腾讯系软件使用
mathjax: false
katex: false
date: 2023-05-26 20:10:31
description: 在Linux下使用腾讯系软件
categories:
- Linux
tags:
- Linux
- QQ
- WeChat
- Tencent
---

# Linux下腾讯系软件使用

## Linux QQ

在2022年年末，腾讯推出了全新的Linux QQ 3.0并[上线官网](https://im.qq.com/linuxqq/download.html)[^1]，经过几天使用，体验还不错

[^1]: [Tencent QQ - ArchWiki](https://wiki.archlinux.org/title/Tencent_QQ)

### Arch Linux下的安装

linuxqq已经被打包上传至[AUR](https://aur.archlinux.org/packages/linuxqq)，安装可通过AUR helper进行

paur安装：

```sh
paru -S linuxqq
```

yay安装：

```sh
yay -S linuxqq
```

不使用AUR helper，手动构建安装

```sh
git clone https://aur.archlinux.org/linuxqq.git # 克隆linuxqq仓库
cd linuxqq                                      # 进入linuxqq目录
makepkg -si                                     # 构建安装linuxqq 或者使用参数全称makepkg --syncdeps --install
```

> 说明：
>
> `s for syncdeps`
>
> s选项表示构建软件包并安装所需要的依赖包
> 
> 当满足依赖包关系且软件包被构建成功时，将会在工作目录下生成一个包文件`pkgname-pkgver.pkg.tar.zst`
>
> `i for install`
>
> 使用`--install`进行安装

### 使用过程问题

可能会发生闪退的现象，需要删除`~/.config/QQ/crash_files/`目录中的所有文件然后删除该目录的读写权限

```sh
chmod -rw ~/.config/QQ/crash_files
```

随后(也许)不会在出现闪退现象

## 微信

微信在Linux中有很多解决方案，比如Wine兼容层、虚拟机等，首选的是微信Linux原生版的重构[^2]

[^2]: [微信 - ArchWiki](https://wiki.archlinuxcn.org/wiki/%E5%BE%AE%E4%BF%A1)

### 原生重构版微信

微信的原生版已被打包到AUR包[wechat-uos-qt](https://aur.archlinux.org/packages/wechat-uos-qt)中

### 统信UOS魔改版

安装[wechat-uos](https://aur.archlinux.org/packages/wechat-uos/)AUR包即可，但是这个魔改版功能有所阉割

### Wine

在Arch Linux中有几种兼容层安装微信的方案
- depin-wine-wechat，为Arch Linux运行微信的Wine容器，版本为最新官方版
- Deepin官网配置的Wine应用，[com.qq.weixin.deepin](https://aur.archlinux.org/packages/com.qq.weixin.deepin/)，版本比较旧
- archlinuxcn仓库的wine-for-wechat
- [Docker容器](https://github.com/huan/docker-wechat)
