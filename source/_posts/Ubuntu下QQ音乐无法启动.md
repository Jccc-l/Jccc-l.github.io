---
title: Ubuntu下QQ音乐无法启动
mathjax: false
katex: false
date: 2024-06-20 21:32:57
description: 解决Ubuntu下Electron应用启动闪退的问题
categories:
- Linux
tags:
- Linux
- Ubuntu
- Electron
---

# Ubuntu下QQ音乐无法启动

QQ音乐安装完成后启动不久后闪退，在终端下启动，报错如下

```
(electron) The default value of app.allowRendererProcessReuse is deprecated, it is currently "false".  It will change to be "true" in Electron 9.  For more information please check https://github.com/electron/electron/issues/18397
QQ login
cookie set success
cookie set success
[6846:0620/214021.934203:FATAL:gpu_data_manager_impl_private.cc(1034)] The display compositor is frequently crashing. Goodbye.
zsh: trace trap (core dumped)  qqmusic
```

## 原因分析

原因不太确定，可能是以下原因：
- Wayland与Electron的沙盒机制有兼容性问题
- GPU问题：在使用Wayland或特定的GPU驱动时，沙盒机制引发与GPU相关的兼容性问题，导致GPU进程崩溃

## 解决方式

### 关闭QQ音乐的沙盒（临时方案）

**通过禁用沙盒可以解决这个问题，但是可能会降低应用程序的安全性**

打开启动项文件

```sh
sudo nano /usr/share/applications/qqmusic.desktop
```

以下是文件内容

```desktop /usr/share/applications/qqmusic.desktop
[Desktop Entry]
Name=qqmusic
Exec=/opt/qqmusic/qqmusic %U
Terminal=false
Type=Application
Icon=qqmusic
StartupWMClass=qqmusic
Comment=Tencent QQMusic
Categories=AudioVideo;
```

在Exec一行加上参数`--no-sandbox`

即修改后文件内容如下

```desktop /usr/share/applications/qqmusic.desktop
[Desktop Entry]
Name=qqmusic
Exec=/opt/qqmusic/qqmusic --no-sandbox %U
Terminal=false
Type=Application
Icon=qqmusic
StartupWMClass=qqmusic
Comment=Tencent QQMusic
Categories=AudioVideo;
```

修改完成后通过桌面打开QQ音乐，可以看到能正常启动

<img src="QQ音乐正常启动.png" style="max-width:50%">
