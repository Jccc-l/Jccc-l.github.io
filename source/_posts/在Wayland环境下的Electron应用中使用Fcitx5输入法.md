---
title: 在Wayland环境下的Electron应用中使用Fcitx5输入法
mathjax: false
katex: false
date: 2024-06-23 13:47:23
description: 解决Wayland下Fcitx5在Electron应用中不可用的问题
categories:
- Linux
tags:
- Fcitx5
- Wayland
- Linux
- Electron
- Wayland
---

# 在Wayland环境下的Electron应用中使用Fcitx5输入法

为Electron添加以下配置[^1][^2]

[^1]: [Wayland#Electron - ArchWiki](https://wiki.archlinux.org/title/Wayland#Electron)
[^2]: [Linux Wayland进展+1——Electron应用可以使用fcitx5输入法了](https://www.bilibili.com/video/BV1aL41117kc/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=e2a73086fbc135c64d40acbe84e65505)

```conf $HOME/.config/electron-flags.conf
--enable-features=WaylandWindowDecorations
--ozone-platform-hint=wayland
--enable-wayland-ime
```

