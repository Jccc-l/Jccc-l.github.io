---
title: Docker学习
date: 2024-05-09 17:42:56
description: Docker学习笔记
categories:
- Docker
tags:
- Docker
- Container
mathjax: false
katex: false
---

# Docker学习

## 安装

使用包管理器安装

```sh
sudo pacman -Syyu docker
```

安装完后记得把电脑重启一下，然后启动`docker`服务

```sh
sudo systemctl restart docker
```

## 命令解读

```sh
$ docker -run -d \
    --name mysql \
    -p 3306:3306 \
    -e TZ=Asia/Shanghai \
    -e MYSQL_ROOT_PASSWORD=123 \
    mysql
```

- `docker run -d`: 创建并运行一个容器，`-d`为后台运行
- `--name mysql`: 给容器起名，必须唯一
- `-p 3306:3306`: 端口映射
    - 容器有独立的网络空间，宿主机无法直接连接到容器，可以将容器的端口映射到宿主机的端口，外部就可以通过宿主机的这个端口转发到容器的端口，从而实现连接
    - 宿主机端口:容器端口
- `-e`: 设置一些容器的环境变量
- `mysql`: 指定运行的镜像的名字，根据这个名字下载对应的镜像
    - 镜像名称一般为两部分：`[repository]:[tag]`
        - repository: 镜像名
        - tag: 镜像的版本
    - 未指定tag时，默认为`latest`，代表最新版本镜像

> 所有docker命令都可以通过`--help`参数查看说明，比如
> 
> `$ docker run --help`

## 常见命令

- 查看本地镜像
    ```sh
    $ docker images
    ```
- 删除本地镜像
    ```sh
    $ docker rmi
    ```
- 构建镜像
    ```sh
    $ docker build
    ```
- 保存镜像文件
    ```sh
    $ docker save
    ```
- 加载镜像文件
    ```sh
    $ docker load
    ```
- 从仓库拉取镜像
    ```sh
    $ docker pull
    ```
- 创建并运行镜像：每次运行都会创建一个新的容器并运行
    ```sh
    $ docker run
    ```
- 停止容器：将容器停止，但是容器还在，下次不用重复创建
    ```sh
    $ docker stop
    ```
- 启动容器：将存在的容器启动
    ```sh
    $ docker start
    ```
- 查看容器进程
    ```sh
    $ docker ps
    ```
    - 查看所有容器，包括已经停止的
        ```sh
        $ docker ps -a
        ```
- 进入容器内部：使用bash进入mysql容器（或者直接进入mysql的root用户）
    ```sh
    $ docker exec -it mysql bash
    $ docker exec -it mysql mysql -u root -p
    ```

## 数据卷

**数据卷（vulume）**是一个虚拟目录，是容器内目录与宿主机目录之间映射的桥梁

数据卷在宿主机对对应的目录在`/var/lib/docker/volumes`

数据卷对宿主机和容器的文件是双向绑定的，对其中一个修改，对应的另一个也会修改

在创建容器的同时完成数据卷的挂载：

```sh
$ docker run -d ... -v 数据卷名称:容器内目录
```

- 查看数据卷
    ```sh
    $ docker volume ls
    ```
- 查看某个数据卷的详细信息
    ```sh
    $ docker volume inspect [volume name]
    ```

`docker inspect Blog`输出信息：

```
        "Mounts": [
            {
                "Type": "volume",
                "Name": "html",
                "Source": "/var/lib/docker/volumes/html/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
```

挂载信息：
- 种类
- 挂载卷名字
- 宿主机目录
- 容器目录
