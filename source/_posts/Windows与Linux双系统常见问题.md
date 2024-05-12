---
title: Windows与Linux双系统常见问题
date: 2023-05-31 20:00:58
description: 记录一些双系统的问题和解决
categories:
- Linux
tags:
- Windows
- Linux
mathjax: false
katex: false
---

# Windows与Linux双系统常见问题解决

## 双系统八小时时间差

原因：
硬件时钟是存储在主板CMOS的时间，关机后该时钟仍然运行，主板电池为其供电。
系统时间：软件系统的时钟。系统启动后读取硬件时间，然后独立运行。
- `Local Time`: 本地时间，只有Windows使用，Windows把硬件时间当作本地时间，或同步时间后把本地时间写入主板，即当地时间与硬件时间一致
- `UTC`: 世界标准时间，Unix-like系统多数会使用，UTC时间经过加减时区后得到本地时间，操作系统中显示的时间与硬件时钟相差相应的时区，如北京时间是UTC+8，系统中的时间是硬件时间加上8小时

当安装双系统后，两者同步时间后都会对主板时钟进行写入，而操作系统一般不会在开机时自动同步时间

解决方法有：
- 将Windows的计时方式改为`UTC`
    - 在`PowerShell`中运行以下命令：
    ```ps1
    reg add “HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation” /v RealTimeIsUniversal /d 1 /t REG_QWORD /f
    ```
- 将Linux的计时方式改为`Local Time`
    - 以`root`用户运行以下命令（如果想改回`UTC`，则将`--localtime`改为`--utc`）
    ```sh
    hwclock -s --localtime
    ```
- 开机使用ntp手动校正时间
