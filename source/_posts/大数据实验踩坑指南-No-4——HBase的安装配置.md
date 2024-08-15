---
title: 大数据实验踩坑指南_No.4——HBase的安装配置.md
mathjax: false
katex: true
date: 2024-06-16 16:28:01
description: 分布式数据库HBase的安装与配置过程记录
categories:
- Big Data
tags:
- Big Data
- HBase
- ZooKeeper
- Hadoop
- Database
---

# 大数据实验踩坑指南_No.4——HBase的安装配置

## 前置准备

按照之前的内容配置好Hadoop和Zookeeper[^1]

[^1]: - {% post_link 大数据实验踩坑指南-No-1——Hadoop的安装配置 %}
      - {% post_link 大数据实验踩坑指南-No-3——ZooKeeper的安装配置 %}

## HBase基础安装

到[HBase官网](https://hbase.apache.org)下载[hbase-2.5.8-hadoop3-bin.tar.gz](https://dlcdn.apache.org/hbase/2.5.8/hbase-2.5.8-hadoop3-bin.tar.gz)

解压文件到`/usr/local`目录

```sh
sudo tar -zxvf /home/hadoop/Downloads/hbase-2.5.8-hadoop3-bin.tar.gz -C /usr/local/
cd /usr/local													# 进入安装目录
sudo mv hbase-2.5.8-hadoop3 hbase								# 将文件明改为hadoop
sudo chown -R hadoop:hadoop ./hbase							# 将目录以及目录内的所有子目录、文件的拥有者改为hadoop用户组的hadoop用户
```

配置hbase的环境变量

```sh /home/hadoop/.bashrc
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin
```

```sh $HBASE_HOME/conf/hbase-env.sh
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HBASE_MANAGES_ZK=false								# 因为我们使用自己安装的ZooKeeper，就禁用自带的ZooKeeper
```

## HBase配置

参考`$HADOOP_HOME/etc/hadoop/core-site.xml`修改`hbase-site.xml`
```xml $HADOOP_HOME/etc/hadoop/core-site.xml
<configuration>
	<property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

```xml $HBASE_HOME/conf/hbase-site.xml
<configuration>
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<property>
		<name>hbase.tmp.dir</name>
		<value>/usr/local/hbase/tmp</value>
	</property>
	<property>
		<name>hbase.unsafe.stream.capability.enforce</name>
		<value>false</value>
	</property>
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://localhost:9000/hbase</value>
	</property>
</configuration>
```

> 注意`hbase.rootdir`的端口号要与`$HADOOP_HOME/etc/hadoop/core-site.xml`文件中`fs.defaultFS`的端口号要一致，这里是`9000`，HBase给出的端口号是`8020`，是不正确的

## 启动HBase

启动HBase之前先启动HDFS和ZooKeeper

```sh
start-dfs.sh
zkServer.sh start
```

然后启动HBase

```sh
start-hbase.sh
```

输入`jps`可以看到`HRegionServer`、`HMaster`、进程已经启动

```
56532 HRegionServer
56328 HMaster
43529 NameNode
43692 DataNode
51900 QuorumPeerMain
43901 SecondaryNameNode
57310 Jps
```

进入HBase Shell：

```sh
hbase shell
```
输出如下：

```
HBase Shell
Use "help" to get list of supported commands.
Use "exit" to quit this interactive shell.
For Reference, please visit: http://hbase.apache.org/2.0/book.html#shell
Version 2.5.8-hadoop3, r37444de6531b1bdabf2e445c83d0268ab1a6f919, Thu Feb 29 15:55:21 PST 2024
Took 0.0017 seconds
hbase:001:0>
```

输入`exit`退出HBase Shell

停止HBase的运行：

```sh
stop-hbase.sh
```

如果无法停止，先通过`hbase-daemon.sh`把`master`和`religionserver`停止

```sh
hbase-daemon.sh stop religionserver
hbase-daemon.sh stop master
stop-hbase.sh
```

输入`jps`可以看到`HMaster`和`HRegionServer`已经不见了

> **重要**：  
> HBase和Hadoop的启动顺序一定是【启动HDFS$\rightarrow$启动HBase】  
> 关闭顺序一定是【关闭HBase$\rightarrow$关闭HDFS】  
> ZooKeeper一定要在HBase之前启动

## HBase试用和排错

启动Hbase Shell
```sh
hbase shell
```

尝试创建并列出表

```
HBase Shell
Use "help" to get list of supported commands.
Use "exit" to quit this interactive shell.
For Reference, please visit: http://hbase.apache.org/2.0/book.html#shell
Version 2.5.8-hadoop3, r37444de6531b1bdabf2e445c83d0268ab1a6f919, Thu Feb 29 15:55:21 PST 2024
Took 0.0017 seconds                                                                                                                     
hbase:001:0> create "test","cf"
Created table test
Took 1.0771 seconds                                                                                                                     
=> Hbase::Table - test
hbase:002:0> list
TABLE                                                                                                                                   
test                                                                                                                                    
1 row(s)
Took 0.0303 seconds                                                                                                                     
=> ["test"]

```

可以看到创建成功了

其他HBase的操作请查看[林老师的文档](https://dblab.xmu.edu.cn/blog/4252/)

**如果没有创建成功并报错**：[^2]

[^2]: [Hbase正常启动，执行命令报错 Server is not running yet](https://www.cnblogs.com/zhangrui153169/p/15736700.html)

```
ERROR: org.apache.hadoop.hbase.ipc.ServerNotRunningYetException: Server is not running yet
```

有两种原因

**原因一**：可能是因为hadoop还处于安全模式，输入以下命令查看

```sh
hdfs dfsadmin -safemode get
```

如果输出如下，等待一段时间，Hadoop会自动退出安全模式，`ON`会变为`OFF`

```
Safe mode is ON
```

**原因二**：jar包冲突

在Hadoop和HBase中的jar包冲突了

首先停止HBase

```sh
stop-hbase.sh
```

如果无法停止，先通过`hbase-daemon.sh`把`master`和`religionserver`停止

```sh
hbase-daemon.sh stop religionserver
hbase-daemon.sh stop master
stop-hbase.sh
```

修改HBase的以下配置，将注释符号`#`去掉，然后重启HBase

```sh $HBASE_HOME/conf/hbase-env.sh
export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP="true"
```

