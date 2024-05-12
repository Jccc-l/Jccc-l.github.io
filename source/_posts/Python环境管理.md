---
title: Python环境管理
date: 2024-05-06 15:40:54
description: Python多版本管理难？版本冲突？看看这里！
categories:
- Python
tags:
- Python
mathjax: false
katex: false
---

# Python环境管理

由于Arch Linux是一个滚动更新的发行版，官方源只提供最新的稳定版本（现在是Python3.12），如果想安装以前的版本或者预览版本，可以通过AUR进行安装

AUR安装Python3.10可以使用以下命令，其它版本同理（或者如果你喜欢，可以使用AUR助手）

```sh
git clone https://aur.archlinux.org/python310
cd python310
makepkg -si
```

对多版本的Python进行管理，一般使用虚拟环境

创建虚拟环境可以通过`python3.10 -m venv venv`命令进行。

创建好虚拟环境以后，通过`source venv/bin/activate`命令将虚拟环境激活，随后即可使用`pip`命令在虚拟环境中管理第三方库文件。如果想退出虚拟环境，输入`deactivate`命令。

