---
title: Ubuntu安装配置Waydroid
mathjax: false
katex: false
date: 2024-06-17 12:17:52
description: Ubuntu下Waydroid的安装与配置
categories:
- Linux
tags:
- Ubuntu
- Android
- Linux
- Waydroid
- Container
---

# Ubuntu安装配置Waydroid

## 安装

根据[Waydroid的文档](https://docs.waydro.id/usage/install-on-desktops)安装好

安装前置软件

```sh
sudo apt-get install curl ca-certificates -y
```

添加软件源

```sh
curl https://repo.waydro.id | sudo bash
```

安装Waydroid

```sh
sudo apt-get install -y waydroid
```

启动Waydroid服务

```sh
systemctl start waydroid-container.service
```

启动Waydroid以后显示“Intialize Waydroid”，点击Download下载安桌内核
