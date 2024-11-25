---
title: Mariadb数据库安装配置
date: 2024-05-03 13:50:09
description: Mariadb数据库的安装配置记录
categories:
- Database
tags:
- 数据库
- MySQL
- Mariadb
- Linux
- Arch Linux
mathjax: false
katex: false
---

# Mariadb安装配置

## 基本安装

创建`/var/lib/mysql`目录并关闭目录的写时复制(CoW)

```sh
sudo mkdir /var/lib/mysql
chattr +C /var/lib/mysql
```

运行以下命令安装`mariadb`包并初始化

```sh
sudo pacman -S mariadb
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

设置Mariadb服务开机自动启动并立刻启动

```sh
sudo systemctl enable --now mariadb.service
```

命令创建了两个具有所有特权的帐户。

- 一个是`root@localhost`，它没有密码，但您需要是系统的`root`用户才能连接。例如，使用`sudo mysql`。
- 另一个是`mysql@localhost`，它也没有密码，但您需要是系统的`mysql`用户才能连接。连接后，你可以设置密码，如果你需要以有密码和无需`sudo`的任何这些用户身份连接。

## 添加用户

创建用户普通用户`user`，密码为`password`，并赋予`mydb`数据库的所有权限
> `localhost`表示运行本机登录
>
> `%`表示所有主机均可登录

```sql
CREATE USER 'user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'user'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
```

## 开启自动补全

mariadb的自动补全默认关闭，需要手动开启。

修改客户端设置，在`client-maariadb`的下方添加`auto-rehash`

```cnf /etc/my.cnf.d/client.cnf
[client-maariadb]
auto-rehash
```

