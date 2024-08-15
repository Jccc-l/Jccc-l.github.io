---
title: Appimage运行失败
mathjax: false
katex: false
date: 2024-06-27 14:19:19
description: Appimage运行报错解决
categories:
- Linux
tags:
- Appimage
- Linux
---

# Appimage运行失败

```
This doesn't look like a squashfs image.

Cannot mount AppImage, please check your FUSE setup.
You might still be able to extract the contents of this AppImage
if you run it with the --appimage-extract option.
See https://github.com/AppImage/AppImageKit/wiki/FUSE
for more information
open dir error: No such file or directory
```

运行以下命令即可

```sh
sudo chmod u+s `which fusermount`
```

