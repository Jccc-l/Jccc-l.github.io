---
title: Fcitx5输入法的安装配置
date: 2024-05-03 11:00:00
description: Fcitx5——小企鹅输入法（Fcitx 读作[ˈfaɪtɪks]）是一个支持扩展的输入法框架
categories:
- Linux
tags:
- Linux
- Input Method
- Fcitx5
mathjax: false
katex: false
---


# Fcitx5输入法的安装配置

## 安装Fcitx5

在Arch Linux中，推荐使用Fcitx5输入法

安装以下必要库

```sh
$ sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-chinese-addons fcitx5-pinyin-zhwiki   # 输入法框架      输入法中文组件      词库
```

设置环境变量。如果使用X11，则全局设置以下环境变量（可在/etc/environment中添加）。如果使用支持文本输入的Wayland合成器，则为每一个Xwayland应用程序设置此环境变量。

```environment /etc/environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
```

对于通用X11应用程序，通过以下环境变量启用 XIM 支持：

```environment /etc/environment
XMODIFIERS=@im=fcitx
```

## 添加中文输入法

启动fcitx5，启用fcitx5-configtool进行配置

Available Input Method栏下方取消`Only Show Current Language`选项，找到简体中文（中国）下方的Pinyin，双击添加

<img src="Add_Pinyin.png" style="max-width:70%">

## 配置字典和云拼音


点上方的`Addons`，选择Pinyin，找到Cloud Pinyin，将`Backend`设置为`Baidu`

<img src="Fcitx5_Pinyin_Configure.png" style="max-width:70%">

<table>
    <tr>
        <td><center><img src="Cloud_Pinyin_1.png" style="max-width:80%">
        <td><center><img src="Cloud_Pinyin_Baidu.png" style="max-width:100%">
    </tr>
</table>

找到`Manage Dictionaries`，查看zhwiki词库是否已添加。

<img src="Dictionaries.png" style="max-width:100%">

## 配置外观

找到Classic User Interface，配置自己喜欢的字体和上面安装的nord主题
<img src="UI_Settings.png" style="max-width:70%">

