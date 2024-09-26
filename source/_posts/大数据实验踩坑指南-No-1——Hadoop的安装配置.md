---
title: 大数据实验踩坑指南_No.1——Hadoop的安装配置
mathjax: false
katex: false
date: 2024-06-16 16:26:43
description: 分布式计算和存储框架Hadoop的安装与配置过程记录
categories:
- Big Data
tags:
- Linux
- Big Data
- Hadoop
---

# 大数据实验踩坑指南_No.1——Hadoop的安装配置

<!--
```sh
#!/bin/bash

# 检查是否以 root 权限运行
if [ "$EUID" -ne 0 ]; then
  echo "Please run as root"
  exit 1
fi

rm -rf /usr/local/hadoop
rm -rf /usr/local/hive
rm -rf /usr/local/hbase
rm -rf /usr/local/zookeeper

# 获取原始用户的用户名和组名
original_user=$(id -un $SUDO_UID)
original_group=$(id -gn $SUDO_GID)

# 解压文件
tar -zxvf ./hadoop*.tar.gz -C /usr/local/
tar -zxvf ./apache-hive*.tar.gz -C /usr/local/
tar -zxvf ./apache-zookeeper*.tar.gz -C /usr/local/
tar -zxvf ./hbase*.tar.gz -C /usr/local/

# 移动解压后的目录
mv /usr/local/hadoop* /usr/local/hadoop
mv /usr/local/apache-hive* /usr/local/hive
mv /usr/local/apache-zookeeper* /usr/local/zookeeper
mv /usr/local/hbase* /usr/local/hbase

echo "Directories have been extracted and ownership changed to $original_user:$original_group"

# 配置环境变量
echo "export HADOOP_HOME=/usr/local/hadoop">> /home/$original_user/.bashrc
echo "export HIVE_HOME=/usr/local/hive">> /home/$original_user/.bashrc
echo "export ZOOKEEPER_HOME=/usr/local/zookeeper">> /home/$original_user/.bashrc
echo "export HBASE_HOME=/usr/local/hbase">> /home/$original_user/.bashrc
echo "export JAVA_HOME=$(readlink -f $(which javac)|sed 's#/bin/javac##')">> /home/$original_user/.bashrc
echo "export JRE_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre">> /home/$original_user/.bashrc
echo "export CLASSPATH=.:\${JAVA_HOME}/lib:\${JRE_HOME}/lib">> /home/$original_user/.bashrc
echo "export PATH=\$PATH:\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:\$HIVE_HOME/bin:\$ZOOKEEPER_HOME/bin:\$HBASE_HOME/bin">>/home/$original_user/.bashrc


# 安装JDBC驱动
cp ./mysql-connector*.deb /tmp/
apt install /tmp/mysql-connector*.deb -y
cp /usr/share/java/mysql-connector-j-*.jar /usr/local/hive/lib/

# 更改目录拥有者
chown -R $original_user:$original_group /usr/local/hadoop
chown -R $original_user:$original_group /usr/local/hive
chown -R $original_user:$original_group /usr/local/hbase
chown -R $original_user:$original_group /usr/local/zookeeper
chown -R $original_user:$original_group /usr/local/bigdatacase

```

-->

## 前置准备

### 系统安装

先要安装好[Ubuntu22.04](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/22.04/ubuntu-22.04.4-desktop-amd64.iso)

安装过程比较简单，下载镜像写入U盘，通过U盘启动镜像，根据提示安装即可

有一些比较重要的点写在下面

#### 显卡驱动问题

NVIDIA显卡在安装的过程中可能会有显示不正常或者黑屏的问题，这是因为默认使用的是开源驱动`Nouveau`，只需要在引导界面`grub`上按`e`，然后在内核参数那一行加上`nomodeset`，禁用显卡驱动，安装过程就显示正常了

系统安装完成重启，同样加上`nomodeset`参数，然后安装好NVIDIA的显卡驱动重启就能正常了（如果在系统安装的时候选择了安装第三方软件，驱动应该也是装好的，装好后就不需要`nomodeset`参数了）

#### 分区

安装过程中涉及到分区的操作，Linux中的分区与Windows不同，注意谨慎操作，以免造成数据丢失

我把300G的空闲分区分成两部分
- 300MB的EFI分区（这是用于存放引导文件的分区，用于系统的启动）
- 剩余部分分给了根分区（用于存储系统的文件）

当然也可以选择其他的分区方式

### 必要程序的安装

在这个案例中，所需要安装的apt包有以下这些，可以通过`sudo apt-get install [包名]`命令进行安装

- `ssh`：网络传输协议
- `openjdk-8-jdk`：Java开发环境
- `mysql-server`：MySQL开源数据库
- `build-essential`：基础编译工具集合
- `cmake`：开源的跨平台自动化建构系统

还有一些可选的工具包
- `vim`：上手难度大但是强大的编辑器
- `xclip`：为vim提供一个访问系统剪贴板的接口（X11桌面协议）
- `wl-clipboard`：为vim提供一个访问系统剪贴板的接口（Wayland桌面协议，需要将环境变量`WAYLAND_DISPLAY设置为`wayland-0`）
- `git`：版本管理工具
- `curl`：网络传输工具，在安装R等软件的时候会用到

### SSH配置

大数据软件需要SSH登录，用于各个软件、各个节点之间的相互访问

安装好`ssh`软件后，通过以下命令启动ssh服务

```sh
sudo service ssh start
```

然后通过以下命令登录本机

```sh
ssh localhost
```

此时会有如下提示，输入yes。然后按照提示输入用户密码`hadoop`，即可登录到本机

> 输入密码时不会像一些软件一样显示\*\*\*的样式，直接输入密码回车

```
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:6puIK4dNQMuwyyKW+AIJscyEMvCLPSfbAWGM36Bslc4.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'localhost' (ED25519) to the list of known hosts.
hadoop@localhost's password: 
```

但是这样每次登录都需要输入密码，接下来配置无密码登录。首先输入`exit`退出ssh，回到原先的终端窗口，通过`ssh-keygen`生成密钥

```sh
ssh-keygen
```
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/hadoop/.ssh/id_rsa
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:dAclEXM4BgyYqGnpsRqRdUvVGmdbJBZU+Lhn7wi164o hadoop@jccc-MS-7D48
The key's randomart image is:
```

> 可以通过`-t`选项指定密钥类型，我这里不指定了，一路回车即可

通过`ssh-copy-id`命令将公钥安装到远程主机上，按照提示输入密码`hadoop`

```sh
ssh-copy-id localhost
```

```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/hadoop/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hadoop@localhost's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'localhost'"
and check to make sure that only the key(s) you wanted were added.
```

接下来尝试登录本机

```sh
ssh localhost
```

可以看到不需要密码就能登陆了

### JDK

通过命令安装jdk

```sh
sudo apt-get install openjdk-8-jdk
```

查看`/usr/lib/jvm`目录的内容

```sh
ls /usr/lib/jvm
```

可以看到`/usr/lib/jvm`内有两个目录`java-1.8.0-openjdk-amd64`和`java-8-openjdk-amd64`，其中`java-1.8.0-openjdk-amd64`是`java-8-openjdk-amd64`链接，相当于一个快捷方式（说法不太准确，但是先这样理解）

然后用你熟悉的编辑器（比如vim、nano，其中nano用法比较简单，也可以使用gedit，gedit是ubuntu中的一个GUI的编辑器，类似Windows的记事本）来打开`/home/hadoop/.bashrc`文件

```sh
gedit /home/hadoop/.bashrc
```

在这个文件中添加以下内容（林子雨老师的教程中写的是在文件的开头添加，实际上配置的先后没有影响，除非前面的配置出现问题，导致配置文件加载失败，错误行以后的内容都不会进行加载）

```sh /home/hadoop/.bashrc
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH		# 通过apt安装的jdk好像不需要这一行，因为/usr/bin是被包含在PATH里的，而里面的java、javac等程序都是JAVA_HOME内程序的链接
```

## Hadoop基础安装

到Hadoop官网[^1]下载好[hadoop-3.3.6.tar.gz](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz)

[^1]: [Apache Hadoop](https://hadoop.apache.org/)

将Hadoop解压到`/usr/local/`目录下

```sh
sudo tar -zxvf /home/hadoop/Downloads/hadoop-3.3.6.tar.gz -C /usr/local/	# 如果系统语言为中文，则文件路径应当是/home/hadoop/下载/hadoop-3.3.6.tar.gz，或者你下载到其他目录，就改为对应的路径
cd /usr/local													# 进入安装目录
sudo mv ./hadoop-3.3.6 hadoop									# 将文件明改为hadoop
sudo chown -R hadoop:hadoop ./hadoop							# 将目录以及目录内的所有子目录、文件的拥有者改为hadoop用户组的hadoop用户
```

编辑Hadoop的环境变量，找到文件对应的行进行修改，需要将`#`删掉，每一项的说明都可以在文件里查看

```sh $HADOOP_HOME/etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_PID_DIR=/usr/local/hadoop/pids
```

输入以下命令来检查Hadoop是否可用，成功则会显示版本信息

```sh
cd $HADOOP_HOME			    # 进入Hadoop的目录
./bin/hadoop version        # 查看Hadoop版本
```

为了可以在任何目录下执行hadoop的命令，我们在.bashrc中配置环境变量

```sh /home/hadoop/.bashrc
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

> `$`表示用于引用变量的值，`:`用于分隔路径列表

配置完成后，通过以下命令加载配置

```sh
source /home/hadoop/.bashrc
```

执行以下命令，如果显示版本信息，则配置成功

```sh
hadoop version
```

## 伪分布式配置

Hadoop可以在单节点以一个伪分布式的模式机型运行。不同的Hadoop守护进程运行在不同的java进程中。

编辑以下文件：<a id="tmpdir"></a>

```xml $HADOOP_HOME/etc/hadoop/core-site.xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://localhost:9000</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/usr/local/hadoop/tmp</value>
	</property>
</configuration>
```

```xml $HADOOP_HOME/etc/hadoop/hdfs-site.xml
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
</configuration>
```

> 在林老师的文档中，还配置了`dfs.namenode.name.dir`和`dfs.datanode.data.dir`
> 
> 而这两个变量的默认值为
> - `dfs.datanode.data.dir`: `file://${hadoop.tmp.dir}/dfs/data`
> - `dfs.namemode.name.dir`: `file://${hadoop.tmp.dir}/dfs/name`
> 
> 所以只需要设置好tmp目录即可，默认的tmp目录为`/tmp`，这个目录内的文件会在电脑重启以后被清除，因此将其配置在$HADOOP_HOME内

格式化hdfs文件系统<a id="namenodeformat"></a>

```sh
hdfs namenode -format
```

在倒数第十二行左右可以看到格式化成功的信息

```
2024-06-15 20:55:38,067 INFO common.Storage: Storage directory /usr/local/hadoop/tmp/dfs/name has been successfully formatted.
2024-06-15 20:55:38,161 INFO namenode.FSImageFormatProtobuf: Saving image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
2024-06-15 20:55:38,279 INFO namenode.FSImageFormatProtobuf: Image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 401 bytes saved in 0 seconds .
2024-06-15 20:55:38,292 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
2024-06-15 20:55:38,322 INFO namenode.FSNamesystem: Stopping services started for active state
2024-06-15 20:55:38,322 INFO namenode.FSNamesystem: Stopping services started for standby state
2024-06-15 20:55:38,324 INFO namenode.FSImage: FSImageSaver clean checkpoint: txid=0 when meet shutdown.
2024-06-15 20:55:38,325 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at jccc-MS-7D48/127.0.1.1
************************************************************/
```

启动`NameNode`守护进程和`DataNode`守护进程

```sh
start-dfs.sh
```

启动完成后，通过`jps`命令可以查看到相应的进程

```
43529 NameNode
44041 Jps
43692 DataNode
43901 SecondaryNameNode
```

> 这些进程的作用如下
> - `NameNode`：HDFS的主节点，负责管理文件系统的命名空间和元数据信息。
> - `DataNode`：HDFS的从节点，负责处理文件系统客户端的读写请求，在文件系统中实际存储数据。
> - `SecondaryNameNode`：主要作用是帮助NameNode合并编辑日志（edits log）和文件系统镜像（fsimage），以此来减少NameNode启动时间。
> - `Jps`：`jps`命令本身的进程，用于列出当前运行的Java进程。

## 运行Hadoop伪分布式实例

尝试运行Hadoop伪分布实例[^2]

[^2]: [Hadoop3.3.5安装教程_单机/伪分布式配置_Hadoop3.3.5/Ubuntu22.04(20.04/18.04/16.04)](https://dblab.xmu.edu.cn/blog/4193/)

## 问题集锦

##### 缺少Namenode

在集群启动后，输入`jps`查看进程发现缺少了Namenode[^3]

[^3]: [Hadoop集群启动后jps缺少namenode](https://www.cnblogs.com/wansiqi/p/14848496.html#:~:text=Hadoop%E9%9B%86%E7%BE%A4%E5%90%AF%E5%8A%A8%E5%90%8Ejps%E7%BC%BA%E5%B0%91namenode%20%E5%8E%9F%E5%9B%A0%E5%88%86%E6%9E%90%EF%BC%9A1%E3%80%81%E5%87%BA%E7%8E%B0%E8%BF%99%E7%A7%8D%E6%83%85%E5%86%B5%E7%9A%84%E5%8E%9F%E5%9B%A0%E5%A4%A7%E5%A4%9A%E6%98%AF%E5%9B%A0%E4%B8%BA%E6%88%91%E4%BB%AC%E5%90%AF%E5%8A%A8Hadoop%E9%9B%86%E7%BE%A4%E5%90%8E%E6%B2%A1%E6%9C%89%E5%85%B3%E9%97%AD%E5%B0%B1%E7%9B%B4%E6%8E%A5%E5%85%B3%E9%97%AD%E4%BA%86%E8%99%9A%E6%8B%9F%E6%9C%BA%202%E3%80%81%E9%A2%91%E7%B9%81%E4%BD%BF%E7%94%A8hadoop%20namenode,-format%E5%AF%B9namenode%E8%BF%9B%E8%A1%8C%E6%A0%BC%E5%BC%8F%E5%8C%96%20%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%EF%BC%9A1%EF%BC%89%E5%81%9C%E6%AD%A2Hadoop%E9%9B%86%E7%BE%A4%202%EF%BC%89%E6%9F%A5%E7%9C%8B%E6%A0%B8%E5%BF%83%E6%96%87%E4%BB%B6core-site.xml%203%EF%BC%89%E5%88%A0%E9%99%A4namenode%E5%BB%BA%E7%AB%8B%E7%9A%84%E4%B8%B4%E6%97%B6%E6%96%87%E4%BB%B6%E5%A4%B9%204%EF%BC%89%E9%87%8D%E6%96%B0%E5%AF%B9namenode%E6%A0%BC%E5%BC%8F%E5%8C%96%E5%90%8E%E5%86%8D%E6%AC%A1%E5%90%AF%E5%8A%A8%E9%9B%86%E7%BE%A4%E5%B0%B1%E5%A5%BD%E4%BA%86)

原因：
1. 启动Hadoop以后没有正常关闭就直接关机
2. 频繁使用`hadoop namenode -format`对`Namenode`进行格式化
3. NameNode格式化失败

解决方式：
1. 停止Hadoop集群
    - `stop-dfs.sh`
2. 查看[核心文件](#tmpdir)`core-site.xml`里的临时文件夹配置
    - `<value>/usr/local/hadoop/tmp</value>`
3. 删除tmp文件夹
    - `rm -rf /usr/local/hadoop/tmp`
4. [重新对Namenode进行格式化](#namenodeformat)
