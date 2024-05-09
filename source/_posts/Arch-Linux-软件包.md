---
title: Arch Linux 软件包
date: 2024-05-06 21:45:51
categories:
- Linux
tags:
- Linux
- Arch Linux
- pacman
---

# Arch Linux 软件包

Arch Linux中常用的软件包

<!-- more-->

| 包名                  | 解释                                                                                         |
|-----------------------|----------------------------------------------------------------------------------------------|
| base                  | 定义系统安装的最小包集合                                                                     |
| linux-zen             | 内核                                                                                         |
| linux-zen-headers     | 内核对应的系统头文件                                                                         |
| linux-firmware        | 系统固件                                                                                     |
| intel-ucode/amd-ucode | CPU微码更新文件，根据CPU进行选择                                                             |
| btrfs-progs           | btrfs文件系统的重要工具                                                                      |
| ntfs-3g               | Linux访问ntfs分区                                                                            |
| zsh                   | Z-Shell                                                                                      |
| grub                  | 启动加载器                                                                                   |
| efibootmgr            | EFI启动项管理器                                                                              |
| os-prober             | 检测硬盘其他系统的工具                                                                       |
| neovim                | 编辑器                                                                                       |
| networkmanager        | 网络管理器                                                                                   |
| man-pages-zh\_cn      | 中文帮助手册                                                                                 |
| grub                  | 启动管理器                                                                                   |
| efibootmgr            | EFI启动项管理器                                                                              |
| os-prober             | 检测硬盘其他系统                                                                             |
| sudo                  | 提权工具                                                                                     |
| xclip                 | neovim的剪贴板工具                                                                           |
| nodejs                | JS运行环境                                                                                   |
| npm                   | JS包管理器                                                                                   |
| yarn                  | JS包管理器                                                                                   |
| python-pynvim         | neovim的python支持                                                                           |
| ripgrep               | 文件内容搜索工具                                                                             |
| openssh               |                                                                                              |
| gnome                 | GNOME DESKTOP                                                                                |
| gnome-shell           | GNOME shell                                                                                  |
| gnome-tweaks          | gnome插件                                                                                    |
| tmux                  | 终端复用器                                                                                   |
| fcitx5-im             | 输入法框架                                                                                   |
| fcitx5-chinese-addons | 中文输入法                                                                                   |
| fcitx5-nord           | 输入法主题                                                                                   |
| fcitx5-pinyin-zhwiki  | 输入法zhwiki词库                                                                             |
| base-devel            |                                                                                              |
| rust                  |                                                                                              |
| git                   | 版本控制工具                                                                                 |
| acpi                  | 查看电源剩余电量                                                                             |
| chromium              | 浏览器                                                                                       |
| mariadb               | 数据库                                                                                       |
| arch-install-scripts  | Arch Linux安装工具                                                                           |
| xf86-video-intel      | Intel核显驱动                                                                                |
| nvidia-dkms           | Nvidia显卡驱动（非linux内核）                                                                |
| bbswitch-dkms         | 显卡驱动电源管理（非linux内核）                                                              |
| xf86-video-intel      | X.Org的Intel核显驱动                                                                         |
| filelight             | 检测硬盘使用信息的工具                                                                       |
| htop                  | 系统资源监测工具                                                                             |
| bc                    |                                                                                              |
| rsync                 | 文件同步工具                                                                                 |
| fzf                   | 模糊搜索工具                                                                                 |
| jdk-openjdk           | Java开发工具                                                                                 |
| ntp                   | 时间同步工具                                                                                 |
| lsd                   | 更美观的ls命令                                                                               |
| exa                   | 更美观的ls命令                                                                               |
| grim                  | Wayland下的截图工具可以参考[Screen Capture](https://wiki.archlinux.org/title/Screen_capture) |
| slurp                 | 在Wayland复合器中选择一个区域，与grim配合，部分屏幕截图                                      |
| wl-clipboard          | Wayland 的命令行复制/粘贴工具，可用于将屏幕截图增加到到剪贴版                                |
| github-cli            | The GitHub CLI                                                                               |
| obsidian              | 笔记应用程序                                                                                 |
| wmname                | 修改窗口管理器名称，用于修复部分应用在小众窗口管理器的运行问题                               |
| pactl                 | pulseaudio控制软件                                                                           |
| remmina               | 远程控制软件                                                                                 |
| calibre               | 书库软件                                                                                     |
| okular                | 浏览pdf文档等                                                                                |
| pinta                 | 图片编辑工具、画图工具                                                                       |
| feh                   | 图片浏览、背景设置                                                                           |

AUR包

| 包名               | 解释                                                       |
|--------------------|------------------------------------------------------------|
| optimus-manager    | Intel/AMD+Nvidia双显卡管理                                 |
| optimus-manager-qt | optimus-manager的图形界面以及配置文件自动生成(GNOME不可用) |
| gdm-prime          | GDM\:加上Prime补丁以支持Optimus                            |
| python310          | Python3.10版本                                             |
| google-chrome-dev  | Google Chrome浏览器Dev通道测试版                           |
| hmcl-stable-bin    | 我的世界HMCL启动器（文字有问题）                           |
