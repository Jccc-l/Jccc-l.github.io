---
title: 在Linux中使用锐捷登录校园网
mathjax: false
katex: false
date: 2024-06-27 14:32:39
description: 在Linux中使用锐界客户端进行校园网连接
categories:
- Linux
tags:
- Ubuntu
- Linux
- Network
---

# 在Linux中使用锐捷登录校园网

> **注**：不知为何现在脚本会报错*不允许的接入方式*

本文以Ubuntu20.04为例

## 下载配置锐捷客户端

如果通过网线连接，在访问网页时，浏览器将会自动导航至`客户端管理中心`，可直接下载Linux系统客户端。若未自动转到该网页，可通过[链接](http://10.10.232.51:8012/Setup/index.htm)下载**RG Supplicant For Linux**。

下载完成后，在`下载`文件夹中找到`RG_Supplicant_For_Linux_V1.31.zip`文件，将其解压，进入`rjsupplicant`文件夹

在`rjsupplicant`文件夹中打开终端，输出以下命令，为`rjsupplicant.sh`文件添加执行权限
```shell
sudo chmod +x rjsupplicant.sh
```

## 登录校园网

首次登录需要输入学号、密码，输入以下命令以登录校园网
```shell
sudo ./rjsupplicant.sh -a 1 -d 1 -u xxx -p *** -S 1
```

上述命令各参数的作用可通过`./rjsupplicant --help`查看，`xxx`改为你的学号，`***`改为你的密码，其中`-S 1`是为了保存帐号密码，保存以后再次登录不需要手动输入学号和密码，即：
```shell
sudo ./rjsupplicant.sh -a 1 -d 1
```

## 开机自动登录

如果需要开机运行这条命令，需要将这条命令写入一个shell脚本，例如将其保存到`connect.sh`文件中，其中`./jsupplicant.sh`需要改成文件的绝对路径，比如我的文件目录为`/home/jccc/Downloads/RG_Supplicant_For_Linux_V1.31/rjsupplicant.sh`，我的`connect.sh`文件应写为：
```shell
sudo /home/jccc/Downloads/RG_Supplicant_For_Linux_V1.31/rjsupplicant.sh -a 1 -d 1
```

将文件保存后，在文件当前目录进入终端，通过以下命令为该文件添加超级用户执行权限
```shell
sudo chmod +x connect.sh
```

然后在**全部应用**中找到**启动应用程序首选项**(Startup Applicantions Preferences)，将`connect.sh`添加到开机启动程序中，即可实现开机自动登录

<img src="startup_apps.png" style="max-width:80%">
