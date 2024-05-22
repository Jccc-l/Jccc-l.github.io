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


### 解决方法

#### 修改系统的计时方式

- 将Windows的计时方式改为`UTC`
    - 在`PowerShell`中运行以下命令：
    ```ps1
    reg add “HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation” /v RealTimeIsUniversal /d 1 /t REG_QWORD /f
    ```
- 将Linux的计时方式改为`Local Time`（这个方法好像没有生效）
    - 以`root`用户运行以下命令（如果想改回`UTC`，则将`--localtime`改为`--utc`）
    ```sh
    hwclock -s --localtime
    ```

#### 系统启动联网后自动同步时间

##### Linux系统

在Linux中，联网后一段时间，`systemd-timesyncd`服务会自动同步时间，不需要额外的操作设置

##### Windows系统


`Win+r`运行`services.msc`，打开服务管理页面

找到`Windows Time`服务，将启动类型修改为**自动（延迟启动）**
<img src="Windows时间服务.png" style="max-width:50%">

打开控制面板，找到**时钟和区域**，点击**日期和时间**

切换到`Internet时间`标签，点击**更改设置**，将服务器修改为可用的NTP服务器，比如这里我选择了阿里云的服务器`ntp2.aliyun.com`，并勾选上**与Internet时间服务器同步**
<img src="修改NTP服务器.png" style="max-width:50%">

以后启动电脑并联网后，时间将会自动同步
