---
title: 大数据实验踩坑指南_No.3——ZooKeeper的安装配置.md
mathjax: false
katex: false
date: 2024-06-16 16:27:38
description: 集中式服务Zookeeper的安装与配置过程记录
categories:
- Big Data
tags:
- Big Data
- ZooKeeper
- HBase
---

# 大数据实验踩坑指南_No.3——ZooKeeper的安装配置

## ZooKeeper基础安装

到[ZooKeeper官网](https://zookeeper.apache.org)下载[apache-zookeeper-3.9.2-bin.tar.gz](https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.9.2/apache-zookeeper-3.9.2-bin.tar.gz)

将其解压到`/usr/local`目录

```sh
$ sudo tar -zxvf /home/hadoop/Downloads/BG/apache-zookeeper-3.9.2-bin.tar.gz -C /usr/local/
```

进入该目录并将ZooKeeper的主目录改名

```sh
$ cd /usr/local/
$ sudo mv ./apache-zookeeper-3.9.2-bin ./zookeeper
```

修改目录拥有者

```sh
$ sudo chown -R hadoop:hadoop ./zookeeper
```

配置环境变量

```sh /home/hadoop/.bashrc
export ZOOKEEPER_HOME=/usr/local/zookeeper
export PATH=$PATH:ZOOKEEPER_HOME/bin
```

加载配置

```sh
$ source /home/hadoop/.bashrc
```

## ZooKeeper配置

进入ZooKeeper的配置目录，并复制一份配置的模板

```sh
$ cd $ZOOKEEPER_HOME/conf
$ cp $ZOOKEEPER_HOME/conf/zoo_sample.cfg $ZOOKEEPER_HOME/conf/zoo.cfg
```

修改`zoo.cfg`文件内容

```cfg $ZOOKEEPER_HOME/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/local/zookeeper/data			# 修改这个数据目录，以免数据被清除
clientPort=2181
```

输入以下命令启动ZooKeeper服务

```sh
$ zkServer.sh start
```

输入`jps`，可以看到ZooKeeper的进程

```
51900 QuorumPeerMain
```

连接ZooKeeper

``` sh
$ zkCli.sh -server 127.0.0.1:2181
```

连接成功会看到以下信息：

```
Connecting to localhost:2181
...
Welcome to ZooKeeper!
...
JLine support is enabled
...
```

输入`quit`退出ZooKeeper

退出后，输入以下命令停止ZooKeeper服务

```sh
$ zkServer.sh stop
```
