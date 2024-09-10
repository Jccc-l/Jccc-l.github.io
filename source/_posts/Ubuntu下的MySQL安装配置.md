---
title: Ubuntu下的MySQL安装配置
mathjax: false
katex: false
date: 2024-06-16 16:10:35
description: 简单记录一下安装MySQL
categories:
- Database
tags:
- Linux
- MySQL
- Database
---

## Ubuntu下MySQL的安装与配置

本案例实验需要把数据存入关系型数据库MySQL，同时，也需要安装MySQL为Hive提供元数据存储服务，因此，需要安装MySQL数据库。

通过`apt`安装MySQL

```sh
sudo apt-get install mysql-server
```

启动MySQL服务

```sh
sudo service mysql start
```

配置MySQL开机自动启动

```sh
sudo systemctl enable mysql
```

```
Synchronizing state of mysql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable mysql
Created symlink /etc/systemd/system/multi-user.target.wants/mysql.service → /lib/systemd/system/mysql.service.
```

登录MySQl Shell

```sh
sudo mysql -u root
```

> 在Ubuntu中，MySQL的root用户登录验证方式为`auth_socket`，允许Linux的同名用户`root`免密登录，而不能使用密码登录

输入`exit`退出MySQL
